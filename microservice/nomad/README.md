# Process to delopy a job

Anytime a job is updated, Nomad creates an evaluation to determine what actions need to take place. In this case, because this is a new job, Nomad has determined that an allocation should be created and has scheduled it on our local agent.

# Archetecture

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/nomad_ach.png" width="700" height="500">
</div>

The Nomad agent is a long running process which runs on every machine. 每个region里至少包含一个nomad cluster, 一个server可能会接收多个client的请求, 也就是说, nomad cluster是横跨多个datacenter, 可能属于某个datacenter, 但没有归属关系

In a Nomad multi-region architecture, communication happens via WAN gossip. 

Nomad to Consul connectivity is over HTTP and should be secured with TLS as well as a Consul token to provide encryption of all traffic. 