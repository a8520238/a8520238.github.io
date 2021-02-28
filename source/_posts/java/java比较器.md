---
title: java比较器
date: 2021-2-28
cover:
top_img:
categories: 
    - java
    - javaSE
tags: 
mathjax: true
katex: true
---
# java比较器
java比较器包括内比较器和外比较器，这里记录这两种比较器和一种便捷的lambda写法
## 内比较器
该类首先要实现Comparable<> 接口，并重写compareTo方法。如在city类中重写如下方法：

    public int compareTo(City o) 
    {        
        if (this.id<o.id){
            return -1;
        }else if (this.id==o.id){
            return 0;
        }else { 
            return 1; 
        }    
    }
## 外比较器
外比较器需要额外写一个比较工具类，该类实现了Comparator<>接口，如下：

    public class CityIdComparator implements Comparator<City> {    
        @Override    
        public int compare(City o1, City o2){
            if(o1.getId()<o2.getId()){
                return -1;
            }else if (o1.getId()==o2.getId()){
                return 0; 
            }else {
                return 1;
            }    
        }
    }
在main方法中调用`Arrays.sort(m, new CityIdComparator())`,即可对m对象元素按工具类中的规则进行比较。
## lambda表达式进行比较器构造
这里仍要利用`Arrays.sort()`方法进行举例，如：
`Arrays.sort(m, (x, y) -> x.value - y.value)`，即可按照lambda表达式所示规则进行比较。

https://blog.csdn.net/u013066244/article/details/78997869
讲的很明白的<\>\=

在源代码中，有调用方法 
if (compare(back, front) < 0) {
    reverse()
}
else () {
}

即compare函数默认是后面的数为a,前面的数为b
所以升序时 若a-b < 0，即返回结果始终后面的值大，因为如果后面的值小，返回a-b<0，会逆序。

同时注意都返回-1，整体逆序。都返回0，不变