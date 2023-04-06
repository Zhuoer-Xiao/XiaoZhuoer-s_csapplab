# lab1 datalab

几年前写过一部分csapp的lab，因为当初基础不牢，很多时候都只能从别人的解答中得到灵感，以前做题的思路也没有记录，最近时间较为空闲，重刷一遍CSAPP，争取全靠自己解答。

## 位异或

```
int bitXor(int x, int y) {
  return (x&~y)|(~x&y);
}
```

当x的某位与y不同时该位为1，否则为0

## 最小补码

```
int tmin(void) {
  return (0x1<<31);
}
```

符号位为1且剩余位为0时最小。

## 判断是否为最大补码

```
int isTmax(int x) {
  return !(x^0x7FFFFFFF);
}
```

最大补码是0x7FFFFFFF，即符号位为0外其他全为1，将参数与该数异或，如果为最大值，结果则为0，取反即为1.

即我们想要判断两个数是否每一位都相等，可以将两个数异或，相等则为全0，取反则为1。

## 判断奇数位是否全为1

```
int allOddBits(int x) {
  return !((x&0xAAAAAAAA)^0xAAAAAAAA);
}
```

此处从0开始。

首先用0xAAAAAAAA作为掩码将奇数位全部取出，然后用上题相同原理进行判断。

## 相反数

```
int negate(int x) {
  return ((~x)+0x1);
}
```

即全部取反，末位+1，根据补码规则而来，可参考CSAPP第二章。

## 是否为ASCII码

```
int isAsciiDigit(int x) {
  return (!((x>>4)^0x00000003))&(((!!(x&0x00000008))^(!!(x&0x00000006)))|!(x&0x00000008));
}
```

ASCII码的范围是0x30 <= x <= 0x39。

所以ASCII码的前28位必定为0x00000030，所以先判断前28位是否正确`!((x>>4)^0x00000003))`，剩下分为两种情况，如果第四位为0，则肯定满足要求，即`!(x&0x00000008))`，如果第四位为1，二三位肯定为0，即`((!!(x&0x00000008))^(!!(x&0x00000006)))`。

## 三目运算符

```
int conditional(int x, int y, int z) {
  return ((~(!!x)+0x1)&y)|(~(~(!!x)+0x1)&z);
}
```

如果x不为0，则将其转换为全1，否则为全0。

通过`!!x`将x转换为1或0，`(~(!!x)+0x1)`，当`x=1`时为全1，`x=0`时为全0，将其作为掩码，得到结果。

## 小于等于

```
int isLessOrEqual(int x, int y) {
  return (((x>>31)&0x1)&!((y>>31)&0x1))|(!((((x>>31)&0x1)^(x>>31)&0x1))&(((y+(~x+1))>>31)^0x1));
}
```

首先取符号，如果x为负且y为正，直接返回true；如果两者符号相同，则用y-x，如果符号位为0，则返回true。

## 逻辑非

```
int logicalNeg(int x) {
  return !!!x;
}
```

使用两个!将其转换为0或1，再取非。

## 最少能用多少个补码表示

```c++
int howManyBits(int x) {
  int sign=x>>31;
  //统一搜索目标
  x=(sign&~x)|(~sign&x);
  int b16,b8,b4,b2,b1,b0;
  //先将x右移16位，判断最高16位有没有有效位，如果有的话，b16=16，接下来就判断这16位，否则的话判断低16位
  b16=!!(x>>16)<<4;
  x>>=b16;
  //判断8位中有没有
  b8=!!(x>>8)<<3;
  x>>=b8;
  //判断4位中有没有
  b4=!!(x>>4)<<2;
  x>>=b4;
  //判断2位中有没有
  b2=!!(x>>2)<<1;
  x>>=b2;
  //判断1位中有没有
  b1=!!(x>>1);
  x>>=b1;
  b0=x;
  //加上符号位
  return b16+b8+b4+b2+b1+b0+1;
}
```

即我们要找到最高的有效位，正数的最高有效位为1，负数为0，首先将其取反，统一用1判断。

此处的算法为二分搜索，一共32位，故需6次判断。

## 浮点数部分

为了便于理解，此处将CSAPP中对浮点数的描述贴上：

![1679821584778](D:\projects\XiaoZhuoer-s_csapplab\datalab\1679821584778.png)

![1679821598594](D:\projects\XiaoZhuoer-s_csapplab\datalab\1679821598594.png)

![1679821604988](D:\projects\XiaoZhuoer-s_csapplab\datalab\1679821604988.png)

![1679821627720](D:\projects\XiaoZhuoer-s_csapplab\datalab\1679821627720.png)

![1679821709165](D:\projects\XiaoZhuoer-s_csapplab\datalab\1679821709165.png)

## float乘2

```
unsigned float_twice(unsigned uf) {
//首先取浮点数各个部分
  unsigned sign=(uf&0x1<<31);
  unsigned exp=(uf&0x7f800000)>>23;
  unsigned frac=uf&0x7FFFFF;
  //如果为情况3，直接返回
  if(exp==0xFF){
    return uf;
  }else if (exp==0x0)
  {//如果为非规格化，直接尾数乘2
    frac<<=1;
  }else{//否则阶码加1，但是可能阶码变成255
    exp+=1;
  }
  if(exp==0xFF){
    //overflow，溢出，直接转换为情况3
    return sign|0x7F800000;
  }
  return sign|(exp<<23)|frac;
}
```

## int转换为float

```
unsigned float_i2f(int x) {
    if (!x) {
        return x;  // 如果 x 为 0，直接返回 0
    }
    else if (x == 0x80000000) {
        return 0xcf000000;  // 如果 x 为最小的负数，直接返回单精度浮点数 0xcf000000
    }
    else {
        int s = x >> 31 & 1;  // 获取符号位，1 表示负数，0 表示正数
        if (s) {
            x = -x;  // 如果 x 是负数，取绝对值
        }

        // 计算指数部分，exp = i + 127，其中 i 是离 x 最高位最近的非零位的位置
        int i = 30;
        while (!(x >> i)) {
            i--;
        }
        int exp = i + 127;

        // 计算尾数部分，frac = (x << (31 - i)) >> 8，其中 frac 的位数为 23 位
        x = x << (31 - i);
        int frac = (x >> 8) & 0x7fffff;

        // 处理舍入，类似于四舍五入，最低有效位为1则入
        x = x & 0xff;
        int dalta = x > 128 || ((x == 128) && (frac & 1));
        frac += dalta;
        if (frac >> 23) {
            frac=frac&0x7fffff;
            exp++;
        }

        // 组装单精度浮点数并返回
        return (s << 31) | (exp << 23) | frac;
    }
}
```



## float转换为int

```
int float_f2i(unsigned uf) {
    unsigned exp = (uf & 0x7f800000) >> 23;  // 获取指数部分
    int sign = uf >> 31 & 0x1;  // 获取符号位
    unsigned frac = uf & 0x7FFFFF;  // 获取尾数部分
    int E = exp - 127;  // 计算指数值，减去偏置

    if (E < 0) {  // 如果指数值小于 0，直接返回 0，因为int不支持小数
        return 0;
    }
    else if (E >= 31) {  // 如果指数值大于等于 31，直接返回 0x80000000u，溢出
        return 0x80000000u;
    }
    else {
        frac = frac | 1 << 23;  // 将尾数部分转换为带隐含位的小数
        if (E < 23) {  // 如果指数值小于 23，需要进行舍入
            frac >>= (23 - E);
        }
        else {  // 如果指数值大于等于 23，需要进行移位操作，低位补0
            frac <<= (E - 23);
        }
    }

    if (sign) {  // 如果是负数，返回相反数
        return -frac;
    }
    return frac;  // 如果是正数，直接返回整数部分
}
```

