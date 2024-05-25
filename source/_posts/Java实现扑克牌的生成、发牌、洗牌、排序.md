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
        List<String> pokeList = new ArrayList<>();
        List<String> shapes = new ArrayList<>(Arrays.asList("♠", "♥", "♣", "♦"));
        List<String> numbers = new ArrayList<>(Arrays.asList("3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A", "2"));
        for (String s : shapes) {
            for (String n : numbers) {
                pokeList.add(s + n);
            }
        }
        pokeList.add("小王");
        pokeList.add("大王");
        System.out.println("构建一副扑克：" + pokeList);
        //洗牌
        Collections.shuffle(pokeList);
        System.out.println("洗牌之后：" + pokeList);
        System.out.println();

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

        //开始排序
        List<String> myFinalPai = DouDiZhu.sortPai(me);
        List<String> jzyFinalPai = DouDiZhu.sortPai(jzy);
        List<String> hghFinalPai = DouDiZhu.sortPai(hgh);

        //完成
        System.out.println("排序后我的牌：" + myFinalPai);
        System.out.println("排序后Jzy的牌：" + jzyFinalPai);
        System.out.println("排序后Hgh的牌：" + hghFinalPai);
    }

    /**
     * 排序扑克
     *
     * @param list 乱序扑克
     * @return 有序扑克
     */
    private static List<String> sortPai(List<String> list) {
        //paiNumberList存储有序数字牌
        List<String> paiNumberList = new ArrayList<>();
        //paiCharacterList存储有序数字2和字母牌
        List<String> paiCharacterList = new ArrayList<>();
        //kingList存储大小王牌
        List<String> kingList = new ArrayList<>();

        //先分别取出数字牌、字母牌、大小王
        for (String tempPai : list) {
            //这里用字符之间比较大小来区分数字牌、字母牌、大小王
            if (tempPai.charAt(1) < 'A') {
                paiNumberList.add(tempPai);
            } else if (tempPai.charAt(1) >= 'A' && tempPai.charAt(1) <= 'Z') {
                paiCharacterList.add(tempPai);
            } else {
                kingList.add(tempPai);
            }
        }

        //排序字母牌
        Queue<String> paiCharacterQueue = new ArrayDeque<>(paiCharacterList);
        paiCharacterList.clear();
        List<String> jList = new ArrayList<>();
        List<String> qList = new ArrayList<>();
        List<String> kList = new ArrayList<>();
        List<String> aList = new ArrayList<>();
        while (!paiCharacterQueue.isEmpty()) {
            if (paiCharacterQueue.peek().charAt(1) == 'A') {
                aList.add(0, paiCharacterQueue.poll());
            } else if (paiCharacterQueue.peek().charAt(1) == 'J') {
                jList.add(0, paiCharacterQueue.poll());
            } else if (paiCharacterQueue.peek().charAt(1) == 'Q') {
                qList.add(0, paiCharacterQueue.poll());
            } else {
                kList.add(0, paiCharacterQueue.poll());
            }
        }
        //按顺序把J、Q、K、A加入到列表
        paiCharacterList.addAll(aList);
        paiCharacterList.addAll(kList);
        paiCharacterList.addAll(qList);
        paiCharacterList.addAll(jList);

        //排序数字牌
        for (int i = 0; i < paiNumberList.size(); i++) {
            //获取数字
            int max = Integer.parseInt(paiNumberList.get(i).substring(1, 2));
            //两位数情况
            if (paiNumberList.get(i).length() == 3) {
                max = Integer.parseInt(paiNumberList.get(i).substring(1, 3));
            }
            //处理2的特殊情况，直接加入到paiCharacterList头部，因为2正好比A大
            if (max == 2) {
                paiCharacterList.add(0, paiNumberList.get(i));
                continue;
            }
            for (int j = i + 1; j < paiNumberList.size(); j++) {
                int maxNew = Integer.parseInt(paiNumberList.get(j).substring(1, 2));
                if (paiNumberList.get(j).length() == 3) {
                    maxNew = Integer.parseInt(paiNumberList.get(j).substring(1, 3));
                }
                //按大小交换list的数据位置，类似选择排序的思想
                if (max < maxNew) {
                    String a = paiNumberList.get(i);
                    String b = paiNumberList.get(j);
                    paiNumberList.set(i, b);
                    paiNumberList.set(j, a);
                    max = maxNew;
                }
            }
        }
        //去掉为2的牌，因为上面已经处理过特殊情况，2被加入到了paiCharacterList里面
        paiNumberList.removeIf(item -> item.contains("2"));

        //合并存储最终所有有序扑克
        List<String> finalPokeList = new ArrayList<>();
        if (kingList.size() == 2) {
            finalPokeList.add("小王");
            finalPokeList.add("大王");
        } else {
            finalPokeList.addAll(kingList);
        }
        finalPokeList.addAll(paiCharacterList);
        finalPokeList.addAll(paiNumberList);

        return finalPokeList;
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
