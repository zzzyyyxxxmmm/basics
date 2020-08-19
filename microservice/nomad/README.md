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