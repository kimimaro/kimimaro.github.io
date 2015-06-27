---
layout: post
title: "二分法查找的实现（递归&amp;循环）"
date: 2015-06-27 12:07:27 +0800
comments: true
categories: 
---

		#include "stdafx.h"
		
		//////////////////////////////////////////
		//二分查找                            //
		//Author:Kimimaro                    //
		//Date:2010-3-28                    //
		/////////////////////////////////////////
		
		typedef int KeyType;    //定义关键字类型
		
		//typedef struct        //定义数据元素结构
		//{
		//    KeyType key;    //关键字域
		//}SElemType;
		
		//宏定义关键字比较操作
		
		#define EQ(a,b) ((a) == (b))
		#define LT(a,b) ((a) <  (b))
		#define LQ(a,b) ((a) <= (b))
		
		//定义表长
		
		#define MAX 5
		
		//----------------静态查找表的顺序存储结构-------------------
		
		typedef struct
		{
		    KeyType *elem;
		    int length;
		}SSTable;
		
		//创建指定长度的静态查找表
		
		int CreatSST(SSTable &S,int n)
		{
		    S.length = n;
		    S.elem = (KeyType*)malloc(S.length*sizeof(KeyType));
		
		    int i = 1;
		
		    while(scanf("%d",&(S.elem[i])) != EOF)
		    {
		        i++;
		    }
		   
		    return 0;
		}
		
		//递归二分查找
		
		int binarySearch(SSTable S,KeyType key,int low,int high)
		{
		    if(low > high) return -1;
		
		    int mid = (low + high) / 2;
		
		    if(EQ(key, S.elem[mid])) return mid;
		    else if(LT(key, S.elem[mid])) binarySearch(S,key,low,mid - 1);
		    else binarySearch(S,key,mid + 1,high);
		}
		
		//循环二分查找
		
		//int binarySearch(SSTable S,KeyType key)
		//{
		//    int low = 1,high = S.length,mid;
		//
		//    while(low <= high)
		//    {
		//        mid = (low + high) / 2;
		//
		//        if(EQ(key,mid)) return mid;
		//        else if(LT(key,mid)) high = mid - 1;
		//        else low = mid + 1;
		//    }
		//
		//    return -1;
		//}
		
		int _tmain(int argc, _TCHAR *argv[])
		{
		    SSTable S;
		    CreatSST(S,MAX);
		
		    for(int i = 1;i <= S.length;i++)
		    {
		        printf("%d ",S.elem[i]);
		        if(i == S.length) printf("\n");
		    }
		
		    int key;
		    printf("Which number do you want to search?\n");
		    scanf("%d",&key);
		
		    int location;
		//    location = binarySearch(S,key);        //循环二分查找
		    location = binarySearch(S,key,1,S.length);        //递归二分查找
		    char chs = getchar();
		    printf("The elem is S.elem[%d].\n",location);
		
		    char ch = getchar();
		   
		    return 0;
		}