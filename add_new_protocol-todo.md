- Add a new protocol to Linux Kernel 

```
Add a new protocol to Linux Kernel 

Written By : Vishal Thanki

The Linux Network sub-system supports several protocols. It is flexible enough to allow addition of new protocols.
These protocols are accessible to user application through socket interface by means of protocol family. The subsequent
 sections will cover major steps to add a new protocol family (with Linux kernel 2.6.24 as reference).
The implementation of the new protocol is not in scope of this document.

Top level view of Linux Network (Kernel) Sub-system:
For the scope of this document, we can consider the network sub-system in Linux consisting of three layers as shown above.
1. The top most “SOCKET” layer takes care of all socket related system calls. It identifies the protocol
family and forwards the call to respective protocol implementation.

2.The next layer implements transport and network layer protocols, where we can introduce our new protocol family.

3. The lowest layer is the network controller device driver providing hardware access.

Adding a New Protocol Family:
The Linux kernel network subsystem data structures, “struct proto” (/include/net/sock.h) and the “struct net_proto_family”
(/include/linux/net.h) encapsulates the protocol family implementation.
Following step by step code snippets show a simplified example to register the new protocol family similar to TCP/IP stack
(using IP as the network layer). Please note, all the protocol specific new functions (to be implemented) has prefix “my_”.
1) Initialize an instance of “struct proto” and register to Linux network sub-system with call “proto_register()”.


/* Protocol specific socket structure */
struct my_sock {
struct inet_sock isk;
/* Add the Protocol implementation specific data members per socket here from here on */
};

struct proto my_proto = {
.close = my_close,
.connect = my_connect,
.disconnect = my_disconnect,
.accept = my_accept,
.ioctl = my_ioctl,
.init = my_init_sock,
.shutdown = my_shutdown,
.setsockopt = my_setsockopt,
.getsockopt = my_getsockopt,
.sendmsg = my_sendmsg,
.recvmsg = my_recvmsg,
.unhash = my_unhash,
.get_port = my_get_port,
.enter_memory_pressure = my_enter_memory_pressure,
.sockets_allocated = &sockets_allocated,
.memory_allocated = &memory_allocated,
.memory_pressure = &memory_pressure,
.orphan_count = &orphan_count,
.sysctl_mem = sysctl_tcp_mem,
.sysctl_wmem = sysctl_tcp_wmem,
.sysctl_rmem = sysctl_tcp_rmem,
.max_header = 0,
.obj_size = sizeof(struct my_sock),
.owner = THIS_MODULE,
.name = "NEW_TCP",
};

rc = proto_register(&my_proto, 1);

2) Provide an interface to create the new protocol specific socket creation routine. Register our handler to socket layer
using call “sock_register()”. The “family” member specifies the address family for the new protocol.
struct net_proto_family my_net_proto = {
.family = AF_INET_NEW_TCP,
.create = my_create_socket,
.owner = THIS_MODULE,
};

rc = sock_register(&my_net_proto, 1);

3) The new protocol’s address family is the only interface for user level socket calls to reach the new protocol implementation.
 The new protocol’s address family AF_INET_NEW_TCP should be added in /include/linux/socket.h. Any socket() call with this new address
 family will be directed to my_create_socket() function in kernel, and which establishes the use of new protocol stack for all subsequent socket operations.

4) The protocol can be connection oriented or connection less as chosen by the protocol implementer. In the socket creation routine,
 protocol implementer specifies a “struct proto_ops” (/include/linux/net.h) instance. The socket layer calls function members
of this proto_ops instance before the protocol specific functions are called (as defined in step# 1). A typical implementation
 of the create socket routine for TCP/IP like (connection oriented) new protocol:
static struct proto_ops my_proto_ops = {
.family = PF_INET,
.owner = THIS_MODULE,
.release = inet_release,
.bind = my_bind,
.connect = inet_stream_connect,
.socketpair = sock_no_socketpair,
.accept = inet_accept,
.getname = inet_getname,
.poll = my_poll,
.ioctl = inet_ioctl,
.listen = my_inet_listen,
.shutdown = inet_shutdown,
.setsockopt = sock_common_setsockopt,
.getsockopt = sock_common_getsockopt,
.sendmsg = inet_sendmsg,
.recvmsg = sock_common_recvmsg,
};

static int my_create_socket(struct socket *sock, int protocol)
{
struct sock *sk;
int rc;

sk = sk_alloc(PF_INET_NEW_TCP, GFP_KERNEL, &my_proto, 1);
if (!sk) {
printk("failed to allocate socket.\n");
return -ENOMEM;
}

sock_init_data(sock, sk);
sk->sk_protocol = 0x0;

sock->ops = &my_proto_ops;
sock->state = SS_UNCONNECTED;

/* Do the protocol specific socket object initialization */
return 0;
};

```  

```

socket with protocol family: PF_SYSCOM
Inter node within host= Intra-SoC: syscom socket link.


 AaSysComRouter_RouteLocalMsg(message);


AaSysComRouter_RouteExternalOrReroutedMsg(message, filterSysComSocketLoopedMsgs);---------------------------------------yes go here


if (AASYSCOM_SOCKET_LINK_CPID == linkCpId && GLO_TRUE == euContextSend)
{
AaSysComSendDataInEuContextImpl(message);--------------------------------------------------
}


 int ret = IPC_SysComSocketSend(sicMsg, msgSize);-----------------------------------------



 int IPC_SysComSocketSend(const void* data, const u32 dataLenght)
 {
      return proxy->sysComSocketSend(data, dataLenght);--------------------------
 }

Proxy::Proxy(AaIPCMode modeParam): mode(modeParam)
 {

  sysComSocketSend                          = AaIPC_SysComSocketSend;--------------------------



extern "C" int CCS_FUNC(AaIPC_SysComSocketSend)(const void* data, const u32 dataLenght)
{
    if (initForSending())
         return socketHandler->sendData(data, dataLenght);-----------------------------------
    return ENOTSOCK;



int SocketHandler::sendData(const void* data, const u32 dataLenght)
 {

     const int socketFd = socketList[0]->socketFd;
 
     ssize_t sent = send(socketFd, data, dataLenght, 0);--------------------------------------------------send()----------------------

int __sys_sendto(int fd, void __user *buff, size_t len, unsigned int flags,
		 struct sockaddr __user *addr,  int addr_len)---------------------------------------

err = __sock_sendmsg(sock, &msg);-------------------------------------------------------------

sock_sendmsg_nosec(sock, msg);


static inline int sock_sendmsg_nosec(struct socket *sock, struct msghdr *msg)
{
	int ret = INDIRECT_CALL_INET(READ_ONCE(sock->ops)->sendmsg,-----------------or, so to use this if proto_ops(not null) overwrite as below!!!!!!!!!!!!!!!!!!!
                                      inet6_sendmsg,--------------------------------or
				     inet_sendmsg,----------------------------------or
                                     sock, msg,
				     msg_data_left(msg));



/** Protocol operations of a SYSCOM datagram socket. */
 const struct proto_ops syscom_dgram_ops = {
 	.family =       PF_SYSCOM,
 	.owner =        THIS_MODULE,
 	.release =      syscom_dgram_release,
 	.bind =         syscom_dgram_bind,
 	.connect =      syscom_dgram_connect,
 	.socketpair =   syscom_dgram_socketpair,
 	.accept =       sock_no_accept,
  	.getname =      syscom_getname,
 	.poll =         syscom_dgram_poll,
 	.ioctl =        syscom_ioctl,
 	.listen =       sock_no_listen,
 	.shutdown =     sock_no_shutdown,
  	.setsockopt =   syscom_setsockopt,
 	.getsockopt =   syscom_getsockopt,
  	.sendmsg =      syscom_dgram_sendmsg,-------------------------------------------------------------so send will finally indeed use this.
 	.recvmsg =      syscom_dgram_recvmsg,
  	.mmap =         sock_no_mmap,
  	.sendpage =     sock_no_sendpage,
  };
  

 static int syscom_dgram_sendmsg(struct socket *sock,
 		struct msghdr *msg, size_t size)


FYI:
Inside node: IPC send
IPC_send(reinterpret_cast<UIpcMsg **>(&msg), targetEuId);
proxy->send(msg, to);
AaIPC_send;


```
