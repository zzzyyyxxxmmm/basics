### 基于Flume的美团日志收集系统(一)架构和设计(https://tech.meituan.com/2013/12/09/meituan-flume-log-system-architecture-and-design.html)

系统基于flume开发, 直接看架构图就明白了, 创新点在于DualChannel的使用, 根据流量大小可以使用不同的channel

Flume uses a transactional approach to guarantee the reliable delivery of the events. 

### [基于Flume的美团日志收集系统(二)改进和优化](https://tech.meituan.com/2013/12/09/meituan-flume-log-system-optimization.html)
调优经验还是比较值得一看的, 基本上都是因地制宜, 结合实际情况进行调优