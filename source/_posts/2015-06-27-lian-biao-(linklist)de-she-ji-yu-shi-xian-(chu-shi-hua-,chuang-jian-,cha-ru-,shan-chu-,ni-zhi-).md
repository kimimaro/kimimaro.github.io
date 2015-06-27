---
layout: post
title: "链表（LinkList）的设计与实现（初始化，创建，插入，删除，逆置）"
date: 2015-06-27 12:08:20 +0800
comments: true
categories: algorithm
---

只是为了自己学习留作记录，需要的朋友可以看看。

修改日志：

version1.1:2011-3-26 1.在尾插法中增加p->next = NULL;2.将类似于p==NULL改为NULL==p(示范性改正，没有全改);

	//////////////////////////////////////
	//单链表的表示与实现                //
	//Author:YuTianhang               //
	//Date:2011.3.23                    //
	
	//Version:1.0                        //
	//////////////////////////////////////
	
	#include "stdafx.h"
	
	typedef int ElemType;
	
	//定义单链表的结点类型
	
	typedef struct LNode
	{
	    ElemType data;
	    struct LNode *next;
	}LNode,*LinkList;
	
	////////////////////////////////////////////////
	
	//单链表的初始化
	
	int InitList(LinkList &L)
	{
	    L = (LinkList)malloc(sizeof(LNode));    //申请结点空间
	
	    if(NULL == L) printf("申请结点空间失败！\n");    //判断是否有足够的内存空间
	
	    L->next = NULL;
	
	    return 1;
	}
	
	///////////////////////////////////////////////
	
	//头插法创建带头结点的单链表
	
	int CreatList_H(LinkList &L)
	{
	    L = (LinkList)malloc(sizeof(LNode));    //申请头结点空间
	    L->next = NULL;    //初始化一个空链表
	
	    ElemType e;
	
	    printf("Input:!\n");
	
	    while(scanf("%d",&e) != EOF)
	    {
	        LNode *p;
	        p = (LNode *)malloc(sizeof(LNode));    //建立一个新结点
	        p->data = e;
	
	        p->next = L->next;    //头插法插入结点p
	        L->next = p;
	    }
	
	    //int i = 5;
	
	    //while(i > 0)
	    //{
	    //    scanf("%d",&e);
	    //   
	    //    LNode *p;
	    //    p = (LNode *)malloc(sizeof(LNode));    //建立一个新结点
	    //    p->data = e;
	
	    //    p->next = L->next;    //头插法插入结点p
	    //    L->next = p;
	
	    //    i--;
	    //}
	
	    return 1;
	}
	
	////////////////////////////////////////////////////////
	
	//尾插法创建带头结点的链表
	
	int CreatList_T(LinkList &L)
	{
	    L = (LinkList)malloc(sizeof(LNode));
	    L->next = NULL;
	
	    ElemType e;
	    LNode *r;
	    r = L;
	    while(scanf("%d",&e) != EOF)
	    {
	        LNode *p;
	        p = (LNode *)malloc(sizeof(LNode));
	        p->data = e;
	
	        p->next = NULL;    //第一次没有付空值，导致输出的时候p的next指针无法确定
	
	        r->next = p;    //尾插法插入结点p
	        r = p;
	    }
	
	    return 1;
	}
	
	///////////////////////////////////////////////////////
	
	//在单链表的第i个位置插入元素e
	
	int ListInsert(LinkList &L,int i,ElemType e)
	{
	    LNode *pre;        //pre为i位置的前驱结点
	    pre = L;
	    while(i > 0)    //查找i位置的前驱结点
	    {
	        pre = pre->next;
	        --i;
	    }
	
	    LNode *p;
	    p = (LNode *)malloc(sizeof(LNode));
	    p->data = e;
	
	    p->next = pre->next;    //将p结点插入到第i个位置
	    pre->next = p;
	
	    return 1;
	}
	
	/////////////////////////////////////////////////////////////
	
	//在单链表中删除第i个结点并用e返回其结点值
	
	int ListDelete(LinkList &L,int i,ElemType &e)
	{
	    LNode *pre;
	    pre = L;
	
	    while(i > 0)
	    {
	        pre = pre->next;
	        --i;
	    }
	
	    LNode *p;
	    p = pre->next;
	    e = p->data;
	
	    pre->next = p->next;    //删除第i个位置的结点p
	    free(p);
	
	    return 1;
	}
	
	/////////////////////////////////////////////////////
	
	//输入单链表的各项
	
	int OutputList(LinkList L)
	{
	    if(L == NULL) printf("Error!\n");
	    LNode *p;
	    p = L->next;
	
	    printf("Kimimaro:\n");
	    do
	    {
	        if(p != L->next) printf(",");
	
	        printf("%d",p->data);
	        p = p->next;
	    }while(p != NULL);
	
	    printf("\n");
	
	    return 1;
	}
	
	//////////////////////////////////////////////////
	
	//单链表的逆置
	
	int InvertList(LinkList &L)
	{
	    LNode *p,*q,*r;
	    p = L->next->next;
	    q = p->next;
	
	    r = L->next;
	    r->next = NULL;
	
	    while(p->next != NULL)
	    {
	        p->next = r;    //头插法插入结点p
	        r = p;
	        p = q;
	        q = p->next;
	    }
	
	    p->next = r;    //最后一个结点的处理
	    L->next = p;
	
	    //while(p != NULL)                            //错误代码，导致最后两个结点循环？
	    //{
	    //    p->next = L->next;    //头插法插入结点p
	    //    L->next = p;
	    //    p = q;
	    //    q = p->next;
	    //}
	
	    //p->next = L->next;    //最后一个结点的处理
	    //L->next = p;
	
	    return 1;
	}
	
	int _tmain(int argc, _TCHAR* argv[])
	{
	    LinkList L;
	
	    CreatList_H(L);
	
	    OutputList(L);
	
	    int i = 3;
	    ElemType e = 22;
	    ListInsert(L,i,e);
	
	    OutputList(L);
	
	    /*ListDelete(L,i,e);
	
	    OutputList(L);*/
	
	    InvertList(L);
	
	    OutputList(L);
	
	    char ch;
	    ch = getchar();
	    return 0;
	}