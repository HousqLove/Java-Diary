# BigInteger

## 1. 概念
在java中，有CPU提供的整型最大范围是64位long类型整数。
使用long类型整数可以直接通过CPU指令进行计算，速度非常快。

如果使用的整数范围超过了long型，就只能用软件模拟一个大整数，java.math.BigInteger就是用来表示任意大小的整数的类，BigInteger内部用一个int[]来模拟一个非常大的整数。

和long整型相比，BigInteger不会有范围限制，但缺点是速度慢。

BigInteger属于不可变类，继承自Number，提供转换为基本类型的xxxValue()方法
```java
public class BigInteger extends Number implements           Comparable<BigInteger>
```
## 2. 使用
对BigInteger做运算的时候只能通过实例方法来进行。如：
```java
    BigInteger bi1 = new BigInteger("1234567");
    BigInteger bi2 = new BigInteger("1234567");
    BigInteger bi3 = bi1.add(bi2);
```
可以把BigInteger转换成其他基本类型, 使用xxxValue()方法，如果BigInteger超出了基本类型的范围，转换时会丢失高位信息。
如果想要准确的转换，可以使用xxxValueExact()超出范围会抛出ArithmeticException：
```java
    BigInteger bi1 = new BigInteger("1234567");
    long longValue = bi1.longValue();
    long longValueExact = bi1.longValueExact();
```
使用上述方法可以把BigInteger转换为基本类型，