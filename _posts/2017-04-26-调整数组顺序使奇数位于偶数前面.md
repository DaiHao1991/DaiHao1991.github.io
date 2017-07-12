---
layout:     post
title:      调整数组顺序使奇数位于偶数前面
subtitle:   
date:       2017-04-26
author:     DH
header-img: img/post-bg-iWatch.jpg 
catalog: true
tags:
    - 算法
    - 数组
---
#### 题目

>输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，
所有的偶数位于位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。

#### 分析

有两种考虑方法，一种是空间换时间，一种是时间换空间。

#### 空间换时间

空间换时间。思路是，遍历原数组，遇到是奇数的就加入到新的数组中，并且要设置一个标志位一直指向新数组中待插入的那一位。然后，再次遍历数组，
遇到偶数就加入到新数组，并且将标志位向后移动一位，指向待插入的位置。最后，将新数组赋值给原数组。

```
public class Solution {
    public void reOrderArray(int [] array) {
        if(array.length == 0){
            return;
        }
        int [] result = new int[array.length];
        int index = 0;

        for(int i = 0;i < array.length;i++){
            if(array[i] % 2 == 1){
                result[index] = array[i];
                index++;
            }        
        }

        for(int i = 0;i < array.length;i++){
            if(array[i] % 2 == 0){
                result[index] = array[i];
                index++;
            }        
        }

        //array = result;
        for(int i = 0;i < array.length;i++){
            array[i] = result[i];
        }
    }
}		

```

这里，有个需要注意的地方，就是最后将排好的数组赋值给原数组时，我直接使用了array = result; 测试结果是不通过，
因为我是做iOS的，在ios中，这样的语句是让array指向result指向的内存。所以，我去查了Java的数组赋值，也问了学Java的同学。
Java中这句话的意思也是让array指向result指向的内存，array原来指向的内存将无法访问。按理说，这时候无论array、result谁对
数组进行操作，数组都会改变。但是牛客网上的测试就是不通过，后来在eclipse中手动测，又可以通过，不知道是什么原因。有知道的还请告知。 
这个方法的循环次数是2n，时间复杂度是n,空间复杂度是n。

#### 时间换空间
时间换空间。简单的说就是不开辟新的空间，直接在原数组上进行更多的移动操作。 
分析：用一个指针指向数组第一个数，并依次与后一个数进行比较，加入第一个数是偶数，第二个数是奇数，就进行交换。这样一趟下来，相邻的两个数
的前后位置就确定了。那么再次进行遍历，交换相邻两个奇数和偶数。这个时候，要注意第二次循环的跳出条件，因为最后一个数在上一次遍历的时候已经
确定了，多以不用比较最后一个数，具体为array.length - 1 -i 

```
public class Solution {
    public void reOrderArray(int [] array) {
        if(array.length == 0){
            return;
        }
        for(int i = 0;i < array.length - 1;i++){
            for(int j =0; j < array.length - 1 -i;j++){
                if(array[j] % 2 == 0 && array[j + 1] % 2 == 1){
                    int temp = array[j+1];
                    array[j+1] = array[j];
                    array[j] = temp; 
                }
            }
        }
    }
}		

```
