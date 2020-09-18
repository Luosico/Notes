# 小知识点

## switch

其后只能跟可以自动转换成整型的类型(char、byte、short、int)，或者String、enum，不能为实型（double、fload）或者long

| 支持   | char、byte、short、int、String、enum以及包装类 |
| ------ | ---------------------------------------------- |
| 不支持 | long、float、double                            |

```java
		int c = 8;
        switch (c) {
            case 8:
                System.out.println(c);
                break;
            case 7:
                System.out.println(c);
                break;
            default:
                System.out.println("default");
        }
```

