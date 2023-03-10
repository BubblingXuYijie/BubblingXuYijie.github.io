---
title: C语言构建单链表
date: 2020-06-29 19:36:10
categories: C
tags:
    - C
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/CLogo.jpg
---
## 源码
```c
#include <stdio.h>
#include <stdlib.h>

typedef struct Node
{
    int data;
    struct Node *next;
}LNode,*LinkList;

LinkList Create_LinkList1()
{
    LinkList L=NULL;
    LNode *s;
    int x;
    scanf("%d",&x);
    while(x!=0)
    {
        s=(LinkList)malloc(sizeof(LNode));
        s->data=x;
        s->next=L;
        L=s;
        scanf("%d",&x);
    }
    return L;
}

void PrintList(LinkList L)
{
	LNode*s=L;
	while(s!=NULL)
	{
		printf("%d ",s->data);
		s=s->next;
	}
}

int main()
{
	LinkList L;
	L=Create_LinkList1();
	PrintList(L);
	return 0;
}
```
## 运行效果图
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/C单链表.png)
