# Intro
Note for learning Unit Network Programming

# Socket

## Connect function
```c++
#include <sys/socket.h>
int connect(int sockfd, const struct sockaddr *servaddr, socklen_t addrlen);
//Returns: 0 if OK, -1 on error
```

In the case of a TCP socket, the connect function initiates TCP's three-way handshake ( Section 2.6). The function returns only when the connection is established or an error occurs. There are several different error returns possible.
1. If the client TCP receives no response to its SYN segment, **ETIMEDOUT** is returned. 4.4BSD, for example, sends one SYN when connect is called, another 6 seconds later, and another 24 seconds later (p. 828 of TCPv2). If no response is received after a total of 75 seconds, the error is returned.
Some systems provide administrative control over this timeout; see Appendix E of TCPv1.
2. If the server's response to the client's SYN is a reset (RST), this indicates that no process is waiting for connections on the server host at the port specified (i.e., the server process is probably not running). This is a hard error and the error **ECONNREFUSED** is returned to the client as soon as the RST is received.
An RST is a type of TCP segment that is sent by TCP when something is wrong. Three conditions that generate an RST are: when a SYN arrives for a port that has no listening server (what we just described), when TCP wants to abort an existing connection, and when TCP receives a segment for a connection that does not exist. (TCPv1 [pp. 246 250] contains additional information.)
3. If the client's SYN elicits an ICMP "destination unreachable" from some intermediate router, this is considered a soft error. The client kernel saves the message but keeps sending SYNs with the same time between each SYN as in the first scenario. If no response is received after some fixed amount of time (75 seconds for 4.4BSD), the saved ICMP error is returned to the process as either EHOSTUNREACH or ENETUNREACH. It is also possible that the remote system is not reachable by any route in the local system's forwarding table, or that the connect call returns without waiting at all.
Many earlier systems, such as 4.2BSD, incorrectly aborted the connection establishment attempt when the ICMP "destination unreachable" was received. This is wrong because this ICMP error can indicate a transient condition. For example, it could be that the condition is caused by a routing problem that will be corrected.
   Notice that ENETUNREACH is not listed in Figure A.15, even when the error indicates
 Page 134
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
that the destination network is unreachable. Network unreachables are considered obsolete, and applications should just treat ENETUNREACH and EHOSTUNREACH as the same error.