---
layout: post
title:  "zmq只router"
date:   2018-06-25 22:21:49
categories: distributed_system
tags: distributed_system
---


[image02](/assets/img/zmq_router_1.png)

1、identity 为 aaa 的 socket 发送了一个消息，这个消息由两部分组成，一个是目的 socket的identity，名字为bbb，另一个是真正的消息，就是"hello"。

2、当aaa 发送的消息被其连接的router接收到之后呢，就不仅仅是刚刚的消息了，ZMQ的底层会偷偷的增加一个消息，那就是 aaa 的identity，所以在 router 看来呢，它接收到的其实是三部分的消息，第一个是消息的来源，第二个是目的地址（bbb 的 identity），第三部分就是真正要传达的信息。

3、当router接收到这么一个消息的时候，会发现，这个消息来源于aaa,并且是发向bbb的，所以router就会发送如下消息：首先发送一个 bbb，表示要发给的目的地址的identity是bbb，然后发送aaa,最后是信息hello。

4、identity 为 bbb的dealer 接收到消息之后，就只有aaa和hello了。router发送的时候不是首先发送了一个bbb吗，去哪里了呢？这次被ZMQ偷偷的拿走了。这就是router的神奇之处，它会看到ZMQ_DEALER和ZMQ_REQ/ZMQ_REP不能看到的东西。

现在再把“请求-回复”模式的规则说一下：就是，ZMQ_ROUTER能够看到消息的来源，以及消息的去向，并且ZMQ_ROUTER会把接收到的第一个消息作为消息来源的identity，把发送的第一个消息作为消息目的地址的identity。


comm.h
{% highlight ruby %}
//comm.h
#ifndef _ZMQCOMM_H_
#define _ZMQCOMM_H_
#include <zmq.h>

#define NAME_LEN    256
#define MSG_LEN        1024

typedef struct {
    char szSrc[NAME_LEN];
    char szDst[NAME_LEN];
    char szMsg[MSG_LEN];
}Zmqmsg;

typedef struct {
    void * sock;
    int iType;
}ZmqSock;

void lockSocket();
void unlockSocket();

int  s_recv(ZmqSock * sock, Zmqmsg * zMsg);
int  s_send(ZmqSock * sock, Zmqmsg * zMsg);

#endif
{% endhighlight %}

comm.c
{% highlight ruby %}
//comm.c
#include <string.h>
#include "comm.h"

void lockSocket()
{
    // lock
}
void unlockSocket()
{
    // unlock
}

int  s_recv(ZmqSock * zmqsock, Zmqmsg * zMsg)
{
    if(NULL == zmqsock || NULL == zMsg)
    {
        return -1;
    }
    int iRet = -1;
    lockSocket();
    int iType = 0;
    int len = sizeof(iType);
    void * sock = zmqsock->sock;
    do{
        iType = zmqsock->iType;
        if(ZMQ_ROUTER == iType)
        {
            printf("router:\n");
            errno = 0;
            if(zmq_recv(sock, zMsg->szSrc, sizeof(zMsg->szSrc), 0) < 0)
            {
                printf("send msg faild : [%s]\n", zmq_strerror(errno));
                break;
            }
            printf("recv : [%s]\n", zMsg->szSrc);
            if(zmq_recv(sock, NULL, 0, 0) < 0)
            {
                printf("send msg faild : [%s]\n", zmq_strerror(errno));
                break;
            }
            printf("recv : []\n");
        }
        else if (ZMQ_DEALER == iType)
        {
            printf("dealer:\n");
            if(zmq_recv(sock, NULL, 0, 0) < 0)
            {
                printf("send msg faild : [%s]\n", zmq_strerror(errno));
                break;
            }
            printf("recv : []\n");
        }
        else if (ZMQ_REQ == iType)
        {
            printf("req:\n");
        }
        else if (ZMQ_REP == iType)
        {
            printf("rep:\n");
        }

        if(zmq_recv(sock, zMsg->szDst, sizeof(zMsg->szDst), 0) < 0)
        {
            printf("send msg faild : [%s]\n", zmq_strerror(errno));
            break;
        }
        printf("recv : [%s]\n", zMsg->szDst);
        if(zmq_recv(sock, NULL, 0, 0) < 0)
        {
            printf("send msg faild : [%s]\n", zmq_strerror(errno));
            break;
        }
        printf("recv : []\n");
        if(zmq_recv(sock, zMsg->szMsg, sizeof(zMsg->szMsg), 0) < 0)
        {
            printf("send msg faild : [%s]\n", zmq_strerror(errno));
            break;
        }
        printf("recv : [%s]\n", zMsg->szMsg);
        iRet = 0;
    }while(0);
    unlockSocket();

    return iRet;
}

int  s_send(ZmqSock * zmqsock, Zmqmsg * zMsg)
{
    if(NULL == zmqsock || NULL == zMsg)
    {
        return -1;
    }
    int iRet = -1;
    lockSocket();
    int iType = zmqsock->iType;
    int len = sizeof(iType);
    void * sock = zmqsock->sock;
    do{
        if(ZMQ_ROUTER == iType)
        {
            printf("router:\n");
            if(zmq_send(sock, zMsg->szDst, strlen(zMsg->szDst), ZMQ_SNDMORE) < 0)
            {
                printf("send msg faild : [%s]\n", zmq_strerror(errno));
                break;
            }
            printf("send : [%s]\n", zMsg->szDst);
            if(zmq_send(sock, "", 0, ZMQ_SNDMORE) < 0)
            {
                printf("send msg faild : [%s]\n", zmq_strerror(errno));
                break;
            }
            printf("send : []\n");
            if(zmq_send(sock, zMsg->szSrc, strlen(zMsg->szSrc), ZMQ_SNDMORE) < 0)
            {
                printf("send msg faild : [%s]\n", zmq_strerror(errno));
                break;
            }
            printf("send : [%s]\n", zMsg->szSrc);
        }
        else if (ZMQ_DEALER == iType)
        {
            printf("dealer:\n");
            if(zmq_send(sock, "", 0, ZMQ_SNDMORE) < 0)
            {
                printf("send msg faild : [%s]\n", zmq_strerror(errno));
                break;
            }
            printf("send : []\n");
            if(zmq_send(sock, zMsg->szDst, strlen(zMsg->szDst), ZMQ_SNDMORE) < 0)
            {
                printf("send msg faild : [%s]\n", zmq_strerror(errno));
                break;
            }
            printf("send : [%s]\n", zMsg->szDst);
        }
        else if (ZMQ_REQ == iType || ZMQ_REP == iType)
        {
            printf("rex:\n");
            if(zmq_send(sock, zMsg->szDst, strlen(zMsg->szDst), ZMQ_SNDMORE) < 0)
            {
                printf("send msg faild : [%s]\n", zmq_strerror(errno));
                break;
            }
            printf("send : [%s]\n", zMsg->szDst);
        }

        if(zmq_send(sock, "", 0, ZMQ_SNDMORE) < 0)
        {
            printf("send msg faild : [%s]\n", zmq_strerror(errno));
            break;
        }
        printf("send : []\n");
        if(zmq_send(sock, zMsg->szMsg, strlen(zMsg->szMsg), 0) < 0)
        {
            printf("send msg faild : [%s]\n", zmq_strerror(errno));
            break;
        }
        printf("send : [%s]\n", zMsg->szMsg);
        iRet = 0;
    }while(0);
    unlockSocket();

    return iRet;
}
{% endhighlight %}

send.c
{% highlight ruby %}
//send.c
//包含zmq的头文件 
#include <zmq.h>
#include <stdio.h>
#include <string.h>
#include "comm.h"

int main(int argc, char * argv[])
{
    void * pCtx = NULL;
    void * pSock = NULL;
    
    const char * pAddr = "ipc://drd.ipc"; //使用ipc协议进行通信，需要连接的目标机器IP地址为drd.ipc
    if((pCtx = zmq_ctx_new()) == NULL)//创建context 
    {
        return 0;
    }
    
    if((pSock = zmq_socket(pCtx, ZMQ_DEALER)) == NULL) //创建socket 
    {
        zmq_ctx_destroy(pCtx);
        return 0;
    }
    int iSndTimeout = 5000;// millsecond
    //设置接收超时 
    if(zmq_setsockopt(pSock, ZMQ_RCVTIMEO, &iSndTimeout, sizeof(iSndTimeout)) < 0)
    {
        zmq_close(pSock);
        zmq_ctx_destroy(pCtx);
        return 0;
    }
    char * pName = "send";
    if(zmq_setsockopt(pSock, ZMQ_IDENTITY, pName, strlen(pName)) < 0) //设置identity
    {
        zmq_close(pSock);
        zmq_ctx_destroy(pCtx);
        return 0;
    }
    if(zmq_connect(pSock, pAddr) < 0)
    {
        zmq_close(pSock);
        zmq_ctx_destroy(pCtx);
        return 0;
    }
    printf("connect to [%s]\n", pAddr);
    ZmqSock zmqsock;
    zmqsock.iType = ZMQ_DEALER;
    zmqsock.sock = pSock;
    //循环发送消息 
    while(1)
    {
        static int i = 0;
        Zmqmsg zmsg;
        memset(&zmsg, 0, sizeof(zmsg));
        snprintf(zmsg.szDst, sizeof(zmsg.szDst), "recv");
        snprintf(zmsg.szMsg, sizeof(zmsg.szMsg), "hello world : %3d", i++);
        printf("Enter to send...\n");
        if(s_send(&zmqsock, &zmsg) < 0)
        {
            fprintf(stderr, "send message faild\n");
        }
        printf("------------------------\n");
        getchar();
    }

    zmq_close(pSock);
    zmq_ctx_destroy(pCtx);
    return 0;
}
{% endhighlight %}

recv.c
{% highlight ruby %}
//recv.c
//包含zmq的头文件 
#include <zmq.h>
#include <stdio.h>
#include <string.h>
#include "comm.h"

int main(int argc, char * argv[])
{
    void * pCtx = NULL;
    void * pSock = NULL;
    const char * pAddr = "ipc://drd.ipc";

    //创建context，zmq的socket 需要在context上进行创建 
    if((pCtx = zmq_ctx_new()) == NULL)
    {
        return 0;
    }
    //创建zmq socket ，socket目前有6中属性 ，这里使用dealer方式
    //具体使用方式请参考zmq官方文档（zmq手册） 
    if((pSock = zmq_socket(pCtx, ZMQ_DEALER)) == NULL)
    {
        zmq_ctx_destroy(pCtx);
        return 0;
    }
    int iRcvTimeout = 5000;// millsecond
    //设置zmq的接收超时时间为5秒 
    if(zmq_setsockopt(pSock, ZMQ_RCVTIMEO, &iRcvTimeout, sizeof(iRcvTimeout)) < 0)
    {
        zmq_close(pSock);
        zmq_ctx_destroy(pCtx);
        return 0;
    }
    char * pName = "recv";
    if(zmq_setsockopt(pSock, ZMQ_IDENTITY, pName, strlen(pName)) < 0)
    {
        zmq_close(pSock);
        zmq_ctx_destroy(pCtx);
        return 0;
    }
    //绑定地址 ipc://drd.ipc 
    //也就是使用ipc协议进行通信，地址为当前目录下的drd.ipc
    if(zmq_connect(pSock, pAddr) < 0)
    {
        zmq_close(pSock);
        zmq_ctx_destroy(pCtx);
        return 0;
    }
    printf("bind at : %s\n", pAddr);
    ZmqSock zmqsock;
    zmqsock.iType = ZMQ_DEALER;
    zmqsock.sock = pSock;
    while(1)
    {
        printf("waitting...\n");
        errno = 0;
        //循环等待接收到来的消息，当超过5秒没有接到消息时，
        //recv函数返回错误信息 ，并使用zmq_strerror函数进行错误定位 
        Zmqmsg zmsg;
        memset(&zmsg, 0, sizeof(zmsg));
        if(s_recv(&zmqsock, &zmsg) < 0)
        {
            printf("error = %s\n", zmq_strerror(errno));
            continue;
        }
        printf("------------------------\n");
    }

    zmq_close(pSock);
    zmq_ctx_destroy(pCtx);
    return 0;
}
{% endhighlight %}


router.c
{% highlight ruby %}
//router.c
//包含zmq的头文件 
#include <zmq.h>
#include <stdio.h>
#include <string.h>
#include "comm.h"

int main(int argc, char * argv[])
{
    void * pCtx = NULL;
    void * pSock = NULL;
    const char * pAddr = "ipc://drd.ipc";
     
    if((pCtx = zmq_ctx_new()) == NULL) //创建context，zmq的socket 需要在context上进行创建
    {
        return 0;
    }
    //创建zmq socket ，socket目前有6中属性 ，这里使用dealer方式
    //具体使用方式请参考zmq官方文档（zmq手册） 
    if((pSock = zmq_socket(pCtx, ZMQ_ROUTER)) == NULL)
    {
        zmq_ctx_destroy(pCtx);
        return 0;
    }
    int iRcvTimeout = -1;// millsecond
    //设置zmq的接收超时时间为5秒 
    if(zmq_setsockopt(pSock, ZMQ_RCVTIMEO, &iRcvTimeout, sizeof(iRcvTimeout)) < 0)
    {
        zmq_close(pSock);
        zmq_ctx_destroy(pCtx);
        return 0;
    }
    char * pName = "router";
    if(zmq_setsockopt(pSock, ZMQ_IDENTITY, pName, strlen(pName)) < 0)
    {
        zmq_close(pSock);
        zmq_ctx_destroy(pCtx);
        return 0;
    }
    //绑定地址 ipc://drd.ipc 
    //也就是使用ipc协议进行通信，地址为当前目录下的drd.ipc
    if(zmq_bind(pSock, pAddr) < 0)
    {
        zmq_close(pSock);
        zmq_ctx_destroy(pCtx);
        return 0;
    }
    printf("bind at : %s\n", pAddr);
    ZmqSock zmqsock;
    zmqsock.iType = ZMQ_ROUTER;
    zmqsock.sock = pSock;
    while(1)
    {
        printf("waitting...\n");
        errno = 0;
        //循环等待接收到来的消息，当超过5秒没有接到消息时，
        //recv函数返回错误信息 ，并使用zmq_strerror函数进行错误定位 
        Zmqmsg zmsg;
        memset(&zmsg, 0, sizeof(zmsg));
        if(s_recv(&zmqsock, &zmsg) < 0)
        {
            printf("error = %s\n", zmq_strerror(errno));
            continue;
        }
        if(s_send(&zmqsock, &zmsg) < 0)
        {
            printf("error = %s\n", zmq_strerror(errno));
            continue;
        }
    }

    zmq_close(pSock);
    zmq_ctx_destroy(pCtx);
    return 0;
}
{% endhighlight %}

Makefile
{% highlight ruby %}
.PHONY : dummy clean

CFLAGS    = -Wall
LDFLAGS    = -lzmq -lpthread

CC        = gcc -g
CXX        = g++ -g
MAKEF    = make -f Makefile
CPA        = cp -a
MAKE    = $(CC)

subdir-list         = $(patsubst %,_subdir_%,$(SUB_DIRS))
subdir-clean-list     = $(patsubst %,_subdir_clean_%,$(SUB_DIRS))

%.o: %.c
	$(MAKE) -o $@ -c $< $(CFLAGS)

%.o: %.cpp
	$(MAKE) -o $@ -c $< $(CFLAGS)

%.os: %.c
	$(MAKE) -fPIC -c $< -o $@ $(CFLAGS) 

%.os: %.cpp
	$(MAKE) -fPIC -c $< -o $@ $(CFLAGS) 

ALL_FILES    = recv send router

all : $(ALL_FILES)

recv : recv.o comm.o
	$(MAKE) -o $@ $(LDFLAGS) $^

send : send.o comm.o
	$(MAKE) -o $@ $(LDFLAGS) $^

router : router.o comm.o
	$(MAKE) -o $@ $(LDFLAGS) $^

clean : 
	rm -rf *.o
	rm -rf $(ALL_FILES)
{% endhighlight %}