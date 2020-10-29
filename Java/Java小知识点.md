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

**如果在case分支的末尾没有break语句，那么会接着执行下一个case分支语句**



## static代码块、代码块、构造方法的执行顺序

```java
public class Main {
    {
        System.out.println("前");
    }
    static {
        System.out.println("static");
    }

    public Main(){
        System.out.println("构造方法");
    }
    
    {
        System.out.println("后");
    }

    public static void main(String[] args) {
        new Main();
        new Main();
    }


}
```

结果为：

```
static
前
后
构造方法
前
后
构造方法
```

**static代码块最先执行，且只执行一次，代码块每次创建对象都会调用，且会在构造方法前调用**

**static代码块在类加载的初始化阶段执行`<clinit>()`方法是调用**