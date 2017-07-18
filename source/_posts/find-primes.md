---
title: 查找素数
categories: 算法
date: 2017-07-07 12:13:17
tags:  算法
---

## 一个Eratosthenes 算法的实现

**质数的定义**
> 质数（prime number）又称素数，有无限个。质数定义为在大于1的自然数中，除了1和它本身以外不再有其他因数，这样的数称为质数。

**算法核心思想**：
这个算法的目的主要是找给定自然数N，找到N以内的所有质数。下面这个算法的实现的思想是：计算出质数P，然后将自然数N内所有能够整除P的数都剔除掉，之后计算出下一个质数，重复以上过程直到所有数都遍历过一次。

```cpp
void findPrimes(bool *isPrimeFlags, const unsigned int n) {
    isPrimeFlags[1] = false;
    
    for (unsigned int i = 2; i < n; i++) {
        isPrimeFlags[i] = true;
    }
    
    //初始化，设置第一个素数为2
    unsigned int p = 2;
    
    //设置j为p的平方的原因是
    //1. 例如素数7，能够整除它的数为14,21,28,35,42,49,56,63,etc. 以此类推，设能够整除
    //它的数为K, 则K = x * 7, 这里x < 7时，比如2，在算法循环初始的时候，已经被设置为false了，则没必要再
    //在素数为7的时候再比较一次了。
    //2. p * p 如果大于n，则能够提前退出而不用遍历所有从2到N的素数。例如找100内的所有素数的时候，找到11的时候，
    //11 * 11 > 100 就能够提前退出了。 
    unsigned int j = p * p;
    
    while (j < n) {
        
        //将所有能够整除p的数都设置为false
        while(j < n) {
            isPrimeFlags[j] = false;
            j += p;
        }
        
        //找到下一个素数
        while(!isPrimeFlags[++p] && p < n) {}
        j = p * p;
    }
}
```

Reference:
[https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes)
