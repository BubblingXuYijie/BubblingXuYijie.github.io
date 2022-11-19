---
title: Java实现扑克牌的生成、发牌、洗牌、排序
date: 2020-06-24 20:31:49
categories: Java
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/javaLogo.png
---
## 需求
生成一副斗地主的扑克牌
三个角色玩牌
输出洗牌后的随机扑克
三个人每人随机获得17张牌，并留下的3张底牌
输出排序过的三人的牌
## 源码

```java
package com.wzbc;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class DouDiZhu {
    public static void main(String[] args) {
        List<String> pokes = new ArrayList<>();
        List<String> colors = new ArrayList<>();
        List<String> numbers = new ArrayList<>();
        colors.add("♠");
        colors.add("♥");
        colors.add("♣");
        colors.add("♦");
        numbers.add("A");
        for (int i=2;i<11;i++){
            numbers.add(i+"");
        }
        numbers.add("J");
        numbers.add("Q");
        numbers.add("K");
        String pai = null;
        for (String c:colors){
            for(String n:numbers){
                pai = c+n;
                pokes.add(pai);
            }
        }
        pokes.add("大王");
        pokes.add("小王");
        System.out.println("牌："+pokes);
        Collections.shuffle(pokes);
        System.out.println("洗牌之后："+pokes);
        ArrayList<String> me = new ArrayList<>();
        ArrayList<String> jzy = new ArrayList<>();
        ArrayList<String> hgh = new ArrayList<>();
        ArrayList<String> dipai = new ArrayList<>();
        for (int i = 0; i < pokes.size(); i++) {
            pai = pokes.get(i);
            if (i>=51){
                dipai.add(pai);
            }else if (i%3==0){
                me.add(pai);
            }else if(i%3==1) {
                jzy.add(pai);
            }else {
                hgh.add(pai);
            }
        }
        System.out.println("底牌"+dipai);
        System.out.println("排序前我的牌："+me);
        System.out.println("排序前Jzy的牌："+jzy);
        System.out.println("排序前Hgh的牌："+hgh);
        System.out.println();

       // System.out.println("排序后我的牌："+me);
       // System.out.println("排序后Jzy的牌："+jzy);
       // System.out.println("排序后Hgh的牌："+hgh);
//        for (int i = 0; i < me.size()-1; i++) {
//            for (int j = 0; j < me.size()-i-1; j++) {
//                if (me.get(j+1).substring(1).compareTo(me.get(j).substring(1))<=0){
//                    String temp = me.get(j).substring(1);
//                    me.get(j) = me.get(j+1);
//                    me.get(j+1) = temp;

                }
            }
        }
    }
}
```

## 输出结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200704165924782.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ4OTIyNDU5,size_16,color_FFFFFF,t_70)
学习顺序原因。。。排序暂时不会
