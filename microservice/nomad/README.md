# Process to delopy a job

Anytime a job is updated, Nomad creates an evaluation to determine what actions need to take place. In this case, because this is a new job, Nomad has determined that an allocation should be created and has scheduled it on our local agent.

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
    
}
```