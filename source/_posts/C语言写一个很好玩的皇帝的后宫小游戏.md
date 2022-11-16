---
title: C语言写一个很好玩的皇帝的后宫小游戏
date: 2020-06-29 20:07:34
categories: C
tags:
    - C
cover: /img/CLogo.jpg
---
## 前言
只是单纯喜欢C语言，闲着无事把以前学习的时候的案例编了一下，都是很基础的代码，for,swich,if这些，基础好的看完后完全可以自己写出这种小游戏
## 先演示一下
游戏设定：后宫中有一些妃子。妃子的级别根据好感度的分为"贵人","嫔妃","贵妃","皇贵妃","皇后"，选择一些互动可以增加或减少好感度。
游戏开始：
1、自行输入自己的名号
2、选择“皇帝下旨选妃”或“翻牌宠妃”或“打入冷宫”或“单独召见爱妃去小树林做纯洁的事”

**皇帝下旨选妃（如图）：因为妃子数量最多为5，所以提示“陛下注意身体呀！”，无法增加妃子**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629195420413.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ4OTIyNDU5,size_16,color_FFFFFF,t_70)

**翻牌宠妃（如图）：输入翻牌妃子“何光辉”，“何光辉”好感度上升10，其他妃子下降10**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200701122052769.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ4OTIyNDU5,size_16,color_FFFFFF,t_70)
**单独召见爱妃去小树林做纯洁的事（如图）：
选择和“金子亿”去小树林做纯洁的事情——就散散步，没有获得好感度**
**选择和“孙茂珺”去小树林做纯洁的事——强行推到，由于我的粗鲁，孙茂珺对我的好感度下降20**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629201708657.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ4OTIyNDU5,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629200913645.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ4OTIyNDU5,size_16,color_FFFFFF,t_70)
**打入冷宫（如图）：选择把“杨洪利”打入冷宫，从妃子列表删除了“杨洪利”**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629195218900.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ4OTIyNDU5,size_16,color_FFFFFF,t_70)
## 上源码

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#define MAX 6       //表示系统要求最大妃子数量
int main()
{
    int i,j,temp;             //循环变量和临时变量
    int searchIndex=-1;     //存放查找的元素的下标
    int count=6;            //当前未打入冷宫的妃子个数
    char emperorName[50];   //皇帝的名号
    int choice;             //皇帝的选择1-4之间
    char tempName[20];      //用来存放临时字符串的字符数组
    char names[MAX][20]={"孙茂珺","祁兴哲","金子亿","何光辉","杨洪利"};
    char levelName[5][10]={"贵人","嫔妃","贵妃","皇贵妃","皇后"};
    int levels[MAX]={1,2,0,0,0,-1};
    int loves[MAX]={100,100,100,100,100,-1};
    printf("各嫔妃的当前状态\n");
    printf("%-12s级别\t好感度\n","姓名");
    for(i=0;i<count-1;i++)
    {
        printf("%-12s%s\t%d\n",names[i],levelName[levels[i]],loves[i]);
    }
    printf("请输入当前登录的皇帝名号：");
    scanf("%s",emperorName);
    printf("*********************************************\n");
    printf("皇帝 %s 驾临，有事上奏，无事退朝！\n",emperorName);
    printf("*********************************************\n");
    system("pause");
    printf("1、皇帝下旨选妃。\n");
    printf("2、翻牌宠妃。\n");
    printf("3、打入冷宫！\n");
    printf("4、单独召见爱妃去小树林做纯洁的事。\n");
    printf("陛下请选择：");
    scanf("%d",&choice);
    printf("*********************************************\n");
    switch(choice)
    {
        case 1:
            if(count<MAX)
            {
                printf("请输入娘娘的名讳：");
                scanf("%s",names[count]);
                levels[count]=0;
                loves[count]=100;
                count++;
            }
            else
            {
                printf("陛下注意身体呀！\n");
            }
            break;
        case 2:
            printf("陛下，请输入今天翻牌的妃子名讳：");
            scanf("%s",tempName);
            for(i=0;i<count;i++)
            {
                if(strcmp(tempName,names[i])==0) //strcmp判断字符串是否相等
                {
                    loves[i]+=10;
                    levels[i]=levels[i]>=4?4:levels[i]+1; //相当于if,else
                }
                else
                {
                    loves[i]-=10;
                }
            }
            break;
        case 3:
            printf("陛下，请输入要打入冷宫的妃子名讳：");
            scanf("%s",tempName);
            for(i=0;i<count;i++)
            {
                if(strcmp(tempName,names[i])==0)
                {
                    searchIndex=i;
                    break;
                }
            }
            if(-1==searchIndex)
            {
                printf("虚惊一场，无人被打入冷宫，该吃吃该喝喝！\n");
            }
            else
            {
                for(i=searchIndex;i<count-1;i++)
                {
                    strcpy(names[i],names[i+1]); //字符串需用stecpy赋值
                    loves[i]=loves[i+1];
                    levels[i]=levels[i+1];
                }
                count--;
            }
            break;
        case 4:
            printf("请输入单独召见去小树林做纯洁的事的爱妃名讳：");
            scanf("%s",tempName);
            printf("*********************************************\n");
            printf("1、强行推到\n");
            printf("2、温柔哄骗\n");
            printf("3、聊点政事\n");
            printf("4、就散散步\n");
            printf("陛下想做哪件纯洁的事：");
            scanf("%d",&temp);
            printf("*********************************************\n");
            switch(temp)
            {
                case 1:
                    printf("%s：“陛下不要！”\n",tempName);
                    system("pause");
                    for(i=0;i<count;i++)
                    {
                        if(strcmp(tempName,names[i])==0)
                        {
                            loves[i]-=20;
                            printf("\n由于您的粗鲁，您与%s的好感度下降了20。\n",tempName);
                            printf("*********************************************\n");
                        }
                    }
                    break;
                case 2:
                    printf("%s：“陛下，我们来爱爱吧！”\n",tempName);
                    system("pause");
                    for(i=0;i<count;i++)
                    {
                        if(strcmp(tempName,names[i])==0)
                        {
                            loves[i]+=20;
                            printf("\n一番春光过后，您与%s的好感度增加了20。\n",tempName);
                            printf("*********************************************\n");
                        }
                    }
                    break;
                case 3:
                    printf("%s：“你真无趣！我走了！”\n",tempName);
                    system("pause");
                    for(i=0;i<count;i++)
                    {
                        if(strcmp(tempName,names[i])==0) //strcmp判断字符串是否相等
                        {
                            loves[i]-=5;
                            printf("\n显然女人不爱政事，您与%s的好感度下降了5。\n",tempName);
                            printf("*********************************************\n");
                        }
                    }
                    break;
                case 4:
                    printf("%s：“就只散散步吗？”\n",tempName);
                    system("pause");
                    if(strcmp(tempName,names[i])==0) //strcmp判断字符串是否相等
                    {
                        printf("\n您与%s的好感度没有变化。\n",tempName);
                        printf("*********************************************\n");
                    }
                    break;
            default:
                printf("\n陛下，请升级来解锁更多纯洁的事！");
                printf("*********************************************\n");
            }
    }
    system("pause");
    for(i=0;i<count-1;i++)
    {
        for(j=0;j<count-1;j++)
        {
            if(levels[j]<levels[j+1])
            {
                temp=levels[j];
                levels[j]=levels[j+1];
                levels[j+1]=temp;
                temp=loves[j];
                loves[j]=loves[j+1];
                loves[j+1]=temp;
                strcpy(tempName,names[j]);
                strcpy(names[j],names[j+1]);
                strcpy(names[j+1],tempName);
            }

        }
    }
    printf("各嫔妃的当前状态\n");
    printf("%-12s级别\t好感度\n","姓名");
    for(i=0;i<count-1;i++)
    {
        printf("%-12s%s\t%d\n",names[i],levelName[levels[i]],loves[i]);
    }
    printf("*********************************************\n");
    return 0;
}
```
## 总结
没有总结，就是C语言基础学完以后，可以写一写这种逻辑较多的小游戏，可以巩固记忆，代码不完善，还可以实现更多的功能，有兴趣的继续修改完善把
