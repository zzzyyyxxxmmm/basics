# Process to delopy a job

Anytime a job is updated, Nomad creates an evaluation to determine what actions need to take place. In this case, because this is a new job, Nomad has determined that an allocation should be created and has scheduled it on our local agent.

# QA

## 什么是Job Summary
# Archetecture

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/nomad_ach.png" width="700" height="500">
</div>

The Nomad agent is a long running process which runs on every machine. 每个region里至少包含一个nomad cluster, 一个server可能会接收多个client的请求, 也就是说, nomad cluster是横跨多个datacenter, 可能属于某个datacenter, 但没有归属关系

In a Nomad multi-region architecture, communication happens via WAN gossip. 

Nomad to Consul connectivity is over HTTP and should be secured with TLS as well as a Consul token to provide encryption of all traffic. 

# API
```go
func (s *HTTPServer) registerHandlers(enableDebug bool) {

}
```
# Job

### /v1/jobs
```go
//command/agent/job_endpoint.go
sJob, writeReq := s.apiJobAndRequestToStructs(args.Job, req, args.WriteRequest)
	regReq := structs.JobRegisterRequest{
		Job:            sJob,
		EnforceIndex:   args.EnforceIndex,
		JobModifyIndex: args.JobModifyIndex,
		PolicyOverride: args.PolicyOverride,
		PreserveCounts: args.PreserveCounts,
		WriteRequest:   *writeReq,
	}

	var out structs.JobRegisterResponse
	if err := s.agent.RPC("Job.Register", &regReq, &out); err != nil {      //通过rpc来调用核心方法的
		return nil, err
	}
```

```go 
//nomad/job_endpoint
// Register is used to upsert a job for scheduling
func (j *Job) Register(args *structs.JobRegisterRequest, reply *structs.JobRegisterResponse) error {
    if done, err := j.srv.forward("Job.Register", args, args, reply); done {    //转发到leader执行, 如果允许stale, 那么可以不是leader
		return err
    }
    
    // Run admission controllers
    //https://kubernetes.io/blog/2019/03/21/a-guide-to-kubernetes-admission-controllers/
	job, warnings, err := j.admissionControllers(args.Job)
	if err != nil {
		return err
    }

    // Check job submission permissions
	if aclObj, err := j.srv.ResolveToken(args.AuthToken); err != nil {
    }

    // Validate Volume Permissions
		for _, tg := range args.Job.TaskGroups {
        }

        // Validate job transitions if its an update
	if err := validateJobUpdate(existingJob, args.Job); err != nil {
		return err
    }
    

    // Submit a multiregion job to other regions (enterprise only).
	// The job will have its region interpolated.
	var existingVersion uint64
	if existingJob != nil {
		existingVersion = existingJob.Version
	}
	isRunner, err := j.multiregionRegister(args, reply, existingVersion)
	if err != nil {
		return err
    }
    

    // Create a new evaluation
	now := time.Now().UnixNano()
	submittedEval := false
	var eval *structs.Evaluation

	// Set the submit time
	args.Job.SubmitTime = now

	// If the job is periodic or parameterized, we don't create an eval.
	if !(args.Job.IsPeriodic() || args.Job.IsParameterized()) {
    }

//****************************************send job here*********************8//
    fsmErr, index, err := j.srv.raftApply(structs.JobRegisterRequestType, args)


//trigger eval
    _, evalIndex, err := j.srv.raftApply(structs.EvalUpdateRequestType, update)
    
}
```

ModifyIndex原来就是raft的index
```go
//nomad/fsm.go
func (n *nomadFSM) Apply(log *raft.Log) interface{} {
case structs.JobRegisterRequestType:
		return n.applyUpsertJob(buf[1:], log.Index)
}
```

```go
func (n *nomadFSM) applyUpsertJob(buf []byte, index uint64) interface{} {
if err := n.state.UpsertJob(index, req.Job); err != nil {
		n.logger.Error("UpsertJob failed", "error", err)
		return err
    }
    
    if req.Eval != nil {
		req.Eval.JobModifyIndex = index
		if err := n.upsertEvals(index, []*structs.Evaluation{req.Eval}); err != nil {
			return err
		}
    }
    
}
```


这里是插入Job信息到db
```go
//nomad state state_store.go
// upsertJobImpl is the implementation for registering a job or updating a job definition
func (s *StateStore) upsertJobImpl(index uint64, job *structs.Job, keepVersion bool, txn *memdb.Txn) error {

    // Check if the job already exists 原来job是存在数据库里的
	existing, err := txn.First("jobs", "id", job.Namespace, job.ID)
	if err != nil {
		return fmt.Errorf("job lookup failed: %v", err)
    }

    // Setup the indexes correctly
	if existing != nil {
		job.CreateIndex = existing.(*structs.Job).CreateIndex
		job.ModifyIndex = index

		existingJob := existing.(*structs.Job)

		// Compute the job status
		var err error
		job.Status, err = s.getJobStatus(txn, job, false)   //先看allocation, 再看evaluation
		if err != nil {
			return fmt.Errorf("setting job status for %q failed: %v", job.ID, err)
		}
	} else {
		job.CreateIndex = index
		job.ModifyIndex = index
		job.JobModifyIndex = index
		job.Version = 0

		if err := s.setJobStatus(index, txn, job, false, ""); err != nil {
			return fmt.Errorf("setting job status for %q failed: %v", job.ID, err)
		}

		// Have to get the job again since it could have been updated
		updated, err := txn.First("jobs", "id", job.Namespace, job.ID)
		if err != nil {
			return fmt.Errorf("job lookup failed: %v", err)
		}
		if updated != nil {
			job = updated.(*structs.Job)
		}
    }
    
    if err := s.updateSummaryWithJob(index, job, txn); err != nil {
		return fmt.Errorf("unable to create job summary: %v", err)
	}

	if err := s.upsertJobVersion(index, job, txn); err != nil {
		return fmt.Errorf("unable to upsert job into job_version table: %v", err)
	}

	if err := s.updateJobScalingPolicies(index, job, txn); err != nil {
		return fmt.Errorf("unable to update job scaling policies: %v", err)
	}

	// Insert the job
	if err := txn.Insert("jobs", job); err != nil {
		return fmt.Errorf("job insert failed: %v", err)
	}
	if err := txn.Insert("index", &IndexEntry{"jobs", index}); err != nil {
		return fmt.Errorf("index update failed: %v", err)
	}
    
}
```




```go
//fsm
case structs.EvalUpdateRequestType:
		return n.applyUpdateEval(buf[1:], log.Index)
```


```go
// handleUpsertingEval is a helper for taking action after upserting an eval.
func (n *nomadFSM) handleUpsertedEval(eval *structs.Evaluation) {
	if eval == nil {
		return
	}

	if eval.ShouldEnqueue() {
		n.evalBroker.Enqueue(eval)
	} else if eval.ShouldBlock() {
		n.blockedEvals.Block(eval)
	} else if eval.Status == structs.EvalStatusComplete &&
		len(eval.FailedTGAllocs) == 0 {
		// If we have a successful evaluation for a node, untrack any
		// blocked evaluation
		n.blockedEvals.Untrack(eval.JobID, eval.Namespace)
	}
}
```

这里是由worker触发的

```go
// run is the long-lived goroutine which is used to run the worker
func (w *Worker) run() {
	for {
		// Dequeue a pending evaluation
		eval, token, waitIndex, shutdown := w.dequeueEvaluation(dequeueTimeout)
		if shutdown {
			return
		}

		// Check for a shutdown
		if w.srv.IsShutdown() {
			w.logger.Error("nacking eval because the server is shutting down", "eval", log.Fmt("%#v", eval))
			w.sendAck(eval.ID, token, false)
			return
		}

		// Wait for the raft log to catchup to the evaluation
		snap, err := w.snapshotMinIndex(waitIndex, raftSyncLimit)
		if err != nil {
			w.logger.Error("error waiting for Raft index", "error", err, "index", waitIndex)
			w.sendAck(eval.ID, token, false)
			continue
		}

		// Invoke the scheduler to determine placements
		if err := w.invokeScheduler(snap, eval, token); err != nil {
			w.logger.Error("error invoking scheduler", "error", err)
			w.sendAck(eval.ID, token, false)
			continue
		}

		// Complete the evaluation
		w.sendAck(eval.ID, token, true)
	}
}

// invokeScheduler is used to invoke the business logic of the scheduler
func (w *Worker) invokeScheduler(snap *state.StateSnapshot, eval *structs.Evaluation, token string) error {
	err = sched.Process(eval)
}

```

```go
// Process is used to handle a single evaluation
func (s *GenericScheduler) Process(eval *structs.Evaluation) error {


	// Retry up to the maxScheduleAttempts and reset if progress is made.
	progress := func() bool { return progressMade(s.planResult) }
	limit := maxServiceScheduleAttempts
	if s.batch {
		limit = maxBatchScheduleAttempts
	}
	if err := retryMax(limit, s.process, progress); err != nil {
		if statusErr, ok := err.(*SetStatusError); ok {
			// Scheduling was tried but made no forward progress so create a
			// blocked eval to retry once resources become available.
			var mErr multierror.Error
			if err := s.createBlockedEval(true); err != nil {
				mErr.Errors = append(mErr.Errors, err)
			}
			if err := setStatus(s.logger, s.planner, s.eval, nil, s.blocked,
				s.failedTGAllocs, statusErr.EvalStatus, err.Error(),
				s.queuedAllocs, s.deployment.GetID()); err != nil {
				mErr.Errors = append(mErr.Errors, err)
			}
			return mErr.ErrorOrNil()
		}
		return err
    }
    
	// Update the status to complete
	return setStatus(s.logger, s.planner, s.eval, nil, s.blocked,
		s.failedTGAllocs, structs.EvalStatusComplete, "", s.queuedAllocs,
		s.deployment.GetID())
}
```

```go
// process is wrapped in retryMax to iteratively run the handler until we have no
// further work or we've made the maximum number of attempts.
func (s *GenericScheduler) process() (bool, error) {
	// Lookup the Job by ID
	s.job, err = s.state.JobByID(ws, s.eval.Namespace, s.eval.JobID)
	// Create a plan
	s.plan = s.eval.MakePlan(s.job)

	// Construct the placement stack
	s.stack = NewGenericStack(s.batch, s.ctx)
	if !s.job.Stopped() {
		s.stack.SetJob(s.job)
	}

	// Compute the target job allocations
	if err := s.computeJobAllocs(); err != nil {
		s.logger.Error("failed to compute job allocations", "error", err)
		return false, err
	}

	// Submit the plan and store the results.
	result, newState, err := s.planner.SubmitPlan(s.plan)
	s.planResult = result
	if err != nil {
		return false, err
	}

	// Decrement the number of allocations pending per task group based on the
	// number of allocations successfully placed
	adjustQueuedAllocations(s.logger, result, s.queuedAllocs)

	// If we got a state refresh, try again since we have stale data
	if newState != nil {
		s.logger.Debug("refresh forced")
		s.state = newState
		return false, nil
	}

	// Try again if the plan was not fully committed, potential conflict
	fullCommit, expected, actual := result.FullCommit(s.plan)
	if !fullCommit {
		s.logger.Debug("plan didn't fully commit", "attempted", expected, "placed", actual)
		if newState == nil {
			return false, fmt.Errorf("missing state refresh after partial commit")
		}
		return false, nil
	}

	// Success!
	return true, nil
}

```

```go
//worker.go
// SubmitPlan is used to submit a plan for consideration. This allows
// the worker to act as the planner for the scheduler.
func (w *Worker) SubmitPlan(plan *structs.Plan) (*structs.PlanResult, scheduler.State, error) {

	// Setup the request
	req := structs.PlanRequest{
		Plan: plan,
		WriteRequest: structs.WriteRequest{
			Region: w.srv.config.Region,
		},
	}
	var resp structs.PlanResponse

SUBMIT:
	// Make the RPC call
	if err := w.srv.RPC("Plan.Submit", &req, &resp); err != nil {
		w.logger.Error("failed to submit plan for evaluation", "eval_id", plan.EvalID, "error", err)
		if w.shouldResubmit(err) && !w.backoffErr(backoffBaselineSlow, backoffLimitSlow) {
			goto SUBMIT
		}
		return nil, nil, err
	} else {
		w.logger.Debug("submitted plan for evaluation", "eval_id", plan.EvalID)
		w.backoffReset()
	}

	// Look for a result
	result := resp.Result
	if result == nil {
		return nil, nil, fmt.Errorf("missing result")
    }

	// Return the result and potential state update
	return result, state, nil
}
```

```go
// Submit is used to submit a plan to the leader
func (p *Plan) Submit(args *structs.PlanRequest, reply *structs.PlanResponse) error {
	if done, err := p.srv.forward("Plan.Submit", args, args, reply); done {
		return err
	}

	// Submit the plan to the queue
	future, err := p.srv.planQueue.Enqueue(plan)
	if err != nil {
		return err
	}

	// Wait for the results
	result, err := future.Wait()
	if err != nil {
		return err
	}

	// Package the result
	reply.Result = result
	reply.Index = result.AllocIndex
    return nil
    
}


// Enqueue is used to enqueue a plan
func (q *PlanQueue) Enqueue(plan *structs.Plan) (PlanFuture, error) {
	// Wrap the pending plan
	pending := &pendingPlan{
		plan:        plan,
		enqueueTime: time.Now(),
		errCh:       make(chan error, 1),
	}

	// Push onto the heap
	heap.Push(&q.ready, pending)
}
```

```go
//plan_apply.go
// planApply is a long lived goroutine that reads plan allocations from
// the plan queue, determines if they can be applied safely and applies
// them via Raft.
//
// Naively, we could simply dequeue a plan, verify, apply and then respond.
// However, the plan application is bounded by the Raft apply time and
// subject to some latency. This creates a stall condition, where we are
// not evaluating, but simply waiting for a transaction to apply.
//
// To avoid this, we overlap verification with apply. This means once
// we've verified plan N we attempt to apply it. However, while waiting
// for apply, we begin to verify plan N+1 under the assumption that plan
// N has succeeded.
//
// In this sense, we track two parallel versions of the world. One is
// the pessimistic one driven by the Raft log which is replicated. The
// other is optimistic and assumes our transactions will succeed. In the
// happy path, this lets us do productive work during the latency of
// apply.
//
// In the unhappy path (Raft transaction fails), effectively we only
// wasted work during a time we would have been waiting anyways. However,
// in anticipation of this case we cannot respond to the plan until
// the Raft log is updated. This means our schedulers will stall,
// but there are many of those and only a single plan verifier.
//
func (p *planner) planApply() {
	// planIndexCh is used to track an outstanding application and receive
	// its committed index while snap holds an optimistic state which
	// includes that plan application.
	var planIndexCh chan uint64
	var snap *state.StateSnapshot

	// prevPlanResultIndex is the index when the last PlanResult was
	// committed. Since only the last plan is optimistically applied to the
	// snapshot, it's possible the current snapshot's and plan's indexes
	// are less than the index the previous plan result was committed at.
	// prevPlanResultIndex also guards against the previous plan committing
	// during Dequeue, thus causing the snapshot containing the optimistic
	// commit to be discarded and potentially evaluating the current plan
	// against an index older than the previous plan was committed at.
	var prevPlanResultIndex uint64

	// Setup a worker pool with half the cores, with at least 1
	poolSize := runtime.NumCPU() / 2
	if poolSize == 0 {
		poolSize = 1
	}
	pool := NewEvaluatePool(poolSize, workerPoolBufferSize)
	defer pool.Shutdown()

	for {
		
		// Evaluate the plan
		result, err := evaluatePlan(pool, snap, pending.plan, p.logger)
		if err != nil {
			p.logger.Error("failed to evaluate plan", "error", err)
			pending.respond(nil, err)
			continue
		}

		// Fast-path the response if there is nothing to do
		if result.IsNoOp() {
			pending.respond(result, nil)
			continue
		}

		// Dispatch the Raft transaction for the plan
		future, err := p.applyPlan(pending.plan, result, snap)
		if err != nil {
			p.logger.Error("failed to submit plan", "error", err)
			pending.respond(nil, err)
			continue
		}

		// Respond to the plan in async; receive plan's committed index via chan
		planIndexCh = make(chan uint64, 1)
		go p.asyncPlanWait(planIndexCh, future, result, pending)
	}
}
```

```go
// evaluatePlan is used to determine what portions of a plan
// can be applied if any. Returns if there should be a plan application
// which may be partial or if there was an error
func evaluatePlan(pool *EvaluatePool, snap *state.StateSnapshot, plan *structs.Plan, logger log.Logger) (*structs.PlanResult, error) {
	defer metrics.MeasureSince([]string{"nomad", "plan", "evaluate"}, time.Now())
	return evaluatePlanPlacements(pool, snap, plan, logger)
}

// evaluatePlanPlacements is used to determine what portions of a plan can be
// applied if any, looking for node over commitment. Returns if there should be
// a plan application which may be partial or if there was an error
func evaluatePlanPlacements(pool *EvaluatePool, snap *state.StateSnapshot, plan *structs.Plan, logger log.Logger) (*structs.PlanResult, error) {
	// Create a result holder for the plan
	result := &structs.PlanResult{
		NodeUpdate:        make(map[string][]*structs.Allocation),
		NodeAllocation:    make(map[string][]*structs.Allocation),
		Deployment:        plan.Deployment.Copy(),
		DeploymentUpdates: plan.DeploymentUpdates,
		NodePreemptions:   make(map[string][]*structs.Allocation),
	}

	// Collect all the nodeIDs
	nodeIDs := make(map[string]struct{})
	nodeIDList := make([]string, 0, len(plan.NodeUpdate)+len(plan.NodeAllocation))
	for nodeID := range plan.NodeUpdate {
		if _, ok := nodeIDs[nodeID]; !ok {
			nodeIDs[nodeID] = struct{}{}
			nodeIDList = append(nodeIDList, nodeID)
		}
	}
	for nodeID := range plan.NodeAllocation {
		if _, ok := nodeIDs[nodeID]; !ok {
			nodeIDs[nodeID] = struct{}{}
			nodeIDList = append(nodeIDList, nodeID)
		}
	}

	// Setup a multierror to handle potentially getting many
	// errors since we are processing in parallel.
	var mErr multierror.Error
	partialCommit := false

	// handleResult is used to process the result of evaluateNodePlan
	handleResult := func(nodeID string, fit bool, reason string, err error) (cancel bool) {
		// Evaluate the plan for this node
		if err != nil {
			mErr.Errors = append(mErr.Errors, err)
			return true
		}
		if !fit {
			// Log the reason why the node's allocations could not be made
			if reason != "" {
				logger.Debug("plan for node rejected", "node_id", nodeID, "reason", reason, "eval_id", plan.EvalID)
			}
			// Set that this is a partial commit
			partialCommit = true

			// If we require all-at-once scheduling, there is no point
			// to continue the evaluation, as we've already failed.
			if plan.AllAtOnce {
				result.NodeUpdate = nil
				result.NodeAllocation = nil
				result.DeploymentUpdates = nil
				result.Deployment = nil
				result.NodePreemptions = nil
				return true
			}

			// Skip this node, since it cannot be used.
			return
		}

		// Add this to the plan result
		if nodeUpdate := plan.NodeUpdate[nodeID]; len(nodeUpdate) > 0 {
			result.NodeUpdate[nodeID] = nodeUpdate
		}
		if nodeAlloc := plan.NodeAllocation[nodeID]; len(nodeAlloc) > 0 {
			result.NodeAllocation[nodeID] = nodeAlloc
		}

		if nodePreemptions := plan.NodePreemptions[nodeID]; nodePreemptions != nil {

			// Do a pass over preempted allocs in the plan to check
			// whether the alloc is already in a terminal state
			var filteredNodePreemptions []*structs.Allocation
			for _, preemptedAlloc := range nodePreemptions {
				alloc, err := snap.AllocByID(nil, preemptedAlloc.ID)
				if err != nil {
					mErr.Errors = append(mErr.Errors, err)
					continue
				}
				if alloc != nil && !alloc.TerminalStatus() {
					filteredNodePreemptions = append(filteredNodePreemptions, preemptedAlloc)
				}
			}

			result.NodePreemptions[nodeID] = filteredNodePreemptions
		}

		return
	}

	// Get the pool channels
	req := pool.RequestCh()
	resp := pool.ResultCh()
	outstanding := 0
	didCancel := false

	// Evaluate each node in the plan, handling results as they are ready to
	// avoid blocking.
OUTER:
	for len(nodeIDList) > 0 {
		nodeID := nodeIDList[0]
		select {
		case req <- evaluateRequest{snap, plan, nodeID}:
			outstanding++
			nodeIDList = nodeIDList[1:]
		case r := <-resp:
			outstanding--

			// Handle a result that allows us to cancel evaluation,
			// which may save time processing additional entries.
			if cancel := handleResult(r.nodeID, r.fit, r.reason, r.err); cancel {
				didCancel = true
				break OUTER
			}
		}
	}

	// Drain the remaining results
	for outstanding > 0 {
		r := <-resp
		if !didCancel {
			if cancel := handleResult(r.nodeID, r.fit, r.reason, r.err); cancel {
				didCancel = true
			}
		}
		outstanding--
	}

	// If the plan resulted in a partial commit, we need to determine
	// a minimum refresh index to force the scheduler to work on a more
	// up-to-date state to avoid the failures.
	if partialCommit {
		index, err := refreshIndex(snap)
		if err != nil {
			mErr.Errors = append(mErr.Errors, err)
		}
		result.RefreshIndex = index

		if result.RefreshIndex == 0 {
			err := fmt.Errorf("partialCommit with RefreshIndex of 0")
			mErr.Errors = append(mErr.Errors, err)
		}

		// If there was a partial commit and we are operating within a
		// deployment correct for any canary that may have been desired to be
		// placed but wasn't actually placed
		correctDeploymentCanaries(result)
	}
	return result, mErr.ErrorOrNil()
}


// run is a long running go routine per worker
func (p *EvaluatePool) run(stopCh chan struct{}) {
	for {
		select {
		case req := <-p.req:
			fit, reason, err := evaluateNodePlan(req.snap, req.plan, req.nodeID)
			p.res <- evaluateResult{req.nodeID, fit, reason, err}

		case <-stopCh:
			return
		}
	}
}

// evaluateNodePlan is used to evaluate the plan for a single node,
// returning if the plan is valid or if an error is encountered
func evaluateNodePlan(snap *state.StateSnapshot, plan *structs.Plan, nodeID string) (bool, string, error) {
	// If this is an evict-only plan, it always 'fits' since we are removing things.
	if len(plan.NodeAllocation[nodeID]) == 0 {
		return true, "", nil
	}

	// Get the node itself
	ws := memdb.NewWatchSet()
	node, err := snap.NodeByID(ws, nodeID)
	if err != nil {
		return false, "", fmt.Errorf("failed to get node '%s': %v", nodeID, err)
	}

	// If the node does not exist or is not ready for scheduling it is not fit
	// XXX: There is a potential race between when we do this check and when
	// the Raft commit happens.
	if node == nil {
		return false, "node does not exist", nil
	} else if node.Status != structs.NodeStatusReady {
		return false, "node is not ready for placements", nil
	} else if node.SchedulingEligibility == structs.NodeSchedulingIneligible {
		return false, "node is not eligible for draining", nil
	} else if node.Drain {
		// Deprecate in favor of scheduling eligibility and remove post-0.8
		return false, "node is draining", nil
	}

	// Get the existing allocations that are non-terminal
	existingAlloc, err := snap.AllocsByNodeTerminal(ws, nodeID, false)
	if err != nil {
		return false, "", fmt.Errorf("failed to get existing allocations for '%s': %v", nodeID, err)
	}

	// Determine the proposed allocation by first removing allocations
	// that are planned evictions and adding the new allocations.
	var remove []*structs.Allocation
	if update := plan.NodeUpdate[nodeID]; len(update) > 0 {
		remove = append(remove, update...)
	}

	// Remove any preempted allocs
	if preempted := plan.NodePreemptions[nodeID]; len(preempted) > 0 {
		remove = append(remove, preempted...)
	}

	if updated := plan.NodeAllocation[nodeID]; len(updated) > 0 {
		remove = append(remove, updated...)
	}
	proposed := structs.RemoveAllocs(existingAlloc, remove)
	proposed = append(proposed, plan.NodeAllocation[nodeID]...)

	// Check if these allocations fit
	fit, reason, _, err := structs.AllocsFit(node, proposed, nil, true)
	return fit, reason, err
}


```