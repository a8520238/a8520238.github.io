---
title: java负数和浮点数
date: 2021-2-28
cover:
top_img:
categories: javaSE
tags: 
mathjax: true
katex: true
---
# 负数
1. 负数是按照补码存储，运算也是按照补码运算
```
int a = -3;
System.out.println("二进制输出"+Integer.toBinaryString(a));
		
System.out.println("八进制输出"+Integer.toOctalString(a));
System.out.printf("八进制输出"+"%010o\n",a);
//按10位十六进制输出，向右靠齐，左边用0补齐
```

```
二进制输出11111111111111111111111111111101
八进制输出37777777775
八进制输出37777777775
```

java负数补码保存，所以对java负数进行&操作时，其实是对补码进行&操作

- 左右移动的时候，高位补1， 低位补0
2. 1 << -1
- int 是32位，左移负数，意味着与0x1f与，也就是左移32-1位

# 浮点数
1. float是单精度类型,精度是8位有效数字，取值范围是10的-38次方到10的38次方，float占用4个字节的存储空间
2. double是双精度类型，精度是17位有效数字，取值范围是10的-308次方到10的308次方，double占用8个字节的存储空间