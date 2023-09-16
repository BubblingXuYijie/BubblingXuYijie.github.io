---
title: Java实现扑克牌的生成、洗牌、发牌、排序
date: 2020-06-24 20:31:49
categories: Java
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/javaLogo.png
---
## 需求

> - 1、生成一副斗地主的扑克牌
> - 2、输出洗牌后的扑克
> - 3、发牌，并留下的3张底牌
> - 4、排序三人的牌

## 源码

> 看代码注释哦

## 方案一（暴力模式，就硬排）

```java
import java.util.*;

public class DouDiZhu {
    public static void main(String[] args) {
        //构建一副扑克
        List<String> pokes = new ArrayList<>();
        List<String> colors = new ArrayList<>(Arrays.asList("♠", "♥", "♣", "♦"));
        List<String> numbers = new ArrayList<>(Arrays.asList("3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A", "2"));
        for (String c : colors){
            for(String n : numbers){
                pokes.add(c + n);
            }
        }
        pokes.add("小王");
        pokes.add("大王");
        //洗牌
        Collections.shuffle(pokes);
        System.out.println("洗牌之后：" + pokes);
        System.out.println();

        //开始发牌
        String pai;
        ArrayList<String> me = new ArrayList<>();
        ArrayList<String> jzy = new ArrayList<>();
        ArrayList<String> hgh = new ArrayList<>();
        ArrayList<String> dipai = new ArrayList<>();
        for (int i = 0; i < pokes.size(); i++) {
            pai = pokes.get(i);
            if (i >= 51){
                dipai.add(pai);
            }else if (i % 3 == 0){
                me.add(pai);
            }else if(i % 3 == 1) {
                jzy.add(pai);
            }else {
                hgh.add(pai);
            }
        }
        System.out.println("底牌" + dipai);
        System.out.println("排序前我的牌：" + me);
        System.out.println("排序前Jzy的牌：" + jzy);
        System.out.println("排序前Hgh的牌：" + hgh);
        System.out.println();

        //开始排序，以“我”的牌为例子
        List<String> paiNumberList = new ArrayList<>();
        List<String> paiCharacterList = new ArrayList<>();
        List<String> kingList = new ArrayList<>();
        //先分别取出数字牌、字母牌、大小王
        for (String tempPai : me){
            //65以内的ascii码不包括汉字和字母
            if (tempPai.charAt(1) < 65){
                paiNumberList.add(tempPai);
            } else if (tempPai.charAt(1) > 64 && tempPai.charAt(1) < 91) {
                paiCharacterList.add(tempPai);
            } else {
                kingList.add(tempPai);
            }
        }
        System.out.println("我的数字牌：" + paiNumberList);
        System.out.println("我的字母牌：" + paiCharacterList);
        System.out.println("我的大小王：" + kingList);
        System.out.println();

        //排序字母牌
        Queue<String> paiCharacterQueue = new ArrayDeque<>(paiCharacterList);
        paiCharacterList.clear();
        List<String> JList = new ArrayList<>();
        List<String> QList = new ArrayList<>();
        List<String> KList = new ArrayList<>();
        List<String> AList = new ArrayList<>();
        while (!paiCharacterQueue.isEmpty()) {
            if (paiCharacterQueue.peek().charAt(1) == 'A') {
                AList.add(paiCharacterQueue.poll());
            } else if (paiCharacterQueue.peek().charAt(1) == 'J') {
                JList.add(paiCharacterQueue.poll());
            } else if (paiCharacterQueue.peek().charAt(1) == 'Q') {
                QList.add(paiCharacterQueue.poll());
            } else {
                KList.add(paiCharacterQueue.poll());
            }
        }
        paiCharacterList.addAll(JList);
        paiCharacterList.addAll(QList);
        paiCharacterList.addAll(KList);
        paiCharacterList.addAll(AList);

        //排序数字牌
        for (int i = 0;i < paiNumberList.size();i++) {
            int min = Integer.parseInt(paiNumberList.get(i).substring(1, 2));
            if (paiNumberList.get(i).length() == 3) {
                min = Integer.parseInt(paiNumberList.get(i).substring(1, 3));
            }
            //处理2的特殊情况
            if (min == 2) {
                paiCharacterList.add("2");
                continue;
            }
            for (int j = i + 1;j < paiNumberList.size();j++) {
                int min2 = Integer.parseInt(paiNumberList.get(j).substring(1, 2));
                if (paiNumberList.get(j).length() == 3) {
                    min2 = Integer.parseInt(paiNumberList.get(j).substring(1, 3));
                }
                if (min > min2 && min2 != 2) {
                    String a = paiNumberList.get(i);
                    String b = paiNumberList.get(j);
                    paiNumberList.set(i, b);
                    paiNumberList.set(j, a);
                    min = min2;
                }
            }
        }
        paiNumberList.removeIf(item -> item.contains("2"));
        paiNumberList.addAll(paiCharacterList);

        //排序大小王
        if (kingList.size() == 2) {
            paiNumberList.add("小王");
            paiNumberList.add("大王");
        } else {
            paiNumberList.addAll(kingList);
        }

        //完成
        System.out.println("排序后我的牌：" + paiNumberList);

    }
}
```

## 输出结果
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/洗牌.png)

---

## Map方法（较优雅）

> 前面的发牌方法都一样，就是构建扑克的时候，把花色和数字组合起来，当作map的key加进去，序号当做value，序号也就是排序序号

```java
import java.util.*;

/**
 * @author 徐一杰
 */
public class MapSortPai {
    public static void main(String[] args) {
        //构建一副扑克
        List<String> shapes = List.of("♠", "♥", "♣", "♦");
        List<String> numbers = List.of("2", "A", "K", "Q", "J", "10", "9", "8", "7", "6", "5", "4", "3");
        Map<String, Integer> pokeMap = new HashMap<>();
        pokeMap.put("大王", 0);
        pokeMap.put("小王", 1);
        int sort = 2;
        for(String n : numbers){
            for (String s : shapes) {
                pokeMap.put(s + n, sort);
                sort++;
            }
        }
        System.out.println("扑克哈希表：" + pokeMap);

        List<String> pokeList = new ArrayList<>();
        for (Map.Entry<String, Integer> entry : pokeMap.entrySet()) {
            pokeList.add(entry.getKey());
        }
        //洗牌
        Collections.shuffle(pokeList);
        System.out.println("洗牌之后：" + pokeList);

        //开始发牌
        String pai;
        List<String> me = new ArrayList<>();
        List<String> jzy = new ArrayList<>();
        List<String> hgh = new ArrayList<>();
        List<String> dipai = new ArrayList<>();
        for (int i = 0; i < pokeList.size(); i++) {
            pai = pokeList.get(i);
            if (i >= 51) {
                dipai.add(pai);
            } else if (i % 3 == 0) {
                me.add(pai);
            } else if (i % 3 == 1) {
                jzy.add(pai);
            } else {
                hgh.add(pai);
            }
        }
        System.out.println("开始发牌");
        System.out.println("底牌" + dipai);
        System.out.println("排序前我的牌：" + me);
        System.out.println("排序前Jzy的牌：" + jzy);
        System.out.println("排序前Hgh的牌：" + hgh);
        System.out.println();

        me.sort(Comparator.comparingInt(pokeMap::get));
        jzy.sort(Comparator.comparingInt(pokeMap::get));
        hgh.sort(Comparator.comparingInt(pokeMap::get));
        System.out.println("排序后我的牌：" + me);
        System.out.println("排序后Jzy的牌：" + jzy);
        System.out.println("排序后Hgh的牌：" + hgh);
    }
}
```

---

# 结语

> 不太喜欢研究算法，用的是都能看懂的笨办法，如果有大佬有更棒的排序方案，一定要叫我去观摩！
