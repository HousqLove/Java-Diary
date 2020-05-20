# 1. 概览
BigDecimal表示一个任意大小且精度完全准确的浮点数。

BigDecimal由任意经度的整数非标度值（非scale）和32位的整数标度（scale）组成。如果为0或正数，则标度是小数点后的位数，如果为负数，则该数的非标度值乘以10的负scale次幂。因此BigDecimal表示的数值是（unscaledValue x 10<sup>-scale</sup>）。

从Number继承，为不可变对象。
```java
public class BigDecimal extends Number implements Comparable<BigDecimal>
```


scale表示小数位。
```java
    BigDecimal bigDecimal = new BigDecimal("-123.4466700");
    System.out.println(bigDecimal);//-123.4466700
    System.out.println(bigDecimal.scale());//7
    BigDecimal bigDecimal1 = bigDecimal.stripTrailingZeros();
    System.out.println(bigDecimal1);//-123.44667
    System.out.println(bigDecimal1.scale());//5
```

BigDecimal可以通过stripTrailingZeros()方法可以将BigDecimal格式化为一个相等的，但去掉末尾的0的BigDecimal。如果scale()返回负数，则表示BigDecimal是一个整数，并且末尾有2个0.
```java
    BigDecimal bigDecimal2 = new BigDecimal("1234500");
    System.out.println(bigDecimal2);//1234500
    System.out.println(bigDecimal2.scale());//0
    BigDecimal bigDecimal3 = bigDecimal2.stripTrailingZeros();
    System.out.println(bigDecimal3);//1.2345E+6
    System.out.println(bigDecimal3.scale());//-2
```

可以对BigDecimal设置他的scale，如果精度比原始值低，那么按照指定的方法进行四舍五入。
```java

    BigDecimal bigDecimal4 = new BigDecimal("123.4567590");
    BigDecimal bigDecimal5 = bigDecimal4.setScale(4, RoundingMode.HALF_DOWN);
    BigDecimal bigDecimal6 = bigDecimal4.setScale(4, RoundingMode.HALF_UP);
    BigDecimal bigDecimal7 = bigDecimal4.setScale(4, RoundingMode.DOWN);
    System.out.println(bigDecimal4);//123.4567590
    System.out.println(bigDecimal5);//123.4568
    System.out.println(bigDecimal6);//123.4568
    System.out.println(bigDecimal7);//123.4567
```
对BigDecimal做加减乘时精度不会丢失，但是做除法时，存在除不尽的情况，必须指定经度和如何截断。
```java

    BigDecimal bigDecimal8 = new BigDecimal("123.456");
    BigDecimal bigDecimal9 = new BigDecimal("23.456759");
    BigDecimal bigDecimal10 = bigDecimal8.divide(bigDecimal9, 10, RoundingMode.HALF_DOWN);
    System.out.println(bigDecimal10);//5.2631311939
    BigDecimal bigDecimal11 = bigDecimal8.divide(bigDecimal9);
    System.out.println(bigDecimal11);//ArithmeticException
```

可以对BigDecimal做除法的时候求余数。
```java
    BigDecimal bigDecimal8 = new BigDecimal("123.456");
    BigDecimal bigDecimal9 = new BigDecimal("23.456759");
    BigDecimal[] bigDecimals = bigDecimal8.divideAndRemainder(bigDecimal9);
    System.out.println(bigDecimals[0]);//5
    System.out.println(bigDecimals[1]);//6.172205
```

在比较两个BigDecimal时，使用equals()方法时，不但要求值相等，还要求scale相等。所以使用compareTo()来比较。