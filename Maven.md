# Maven

### 1、依赖原则

- 最短路径优先

  ```
  一个项目 Demo 依赖了两个 jar 包，其中 A-B-C-X(1.0) ， A-D-X(2.0) 。由于 X(2.0) 路径最短，所以项目使用的是 X(2.0)
  ```

- pom文件中申明的顺序优先

  ```
  如果 A-B-X(1.0) ，A-C-X(2.0) 这样的路径长度一样怎么办呢？这样的情况下，Maven 会根据 pom 文件声明的顺序加载，如果先声明了 B ，后声明了 C ，那就最后的依赖就会是 X(1.0) 
  ```

- 覆写优先

  ```
  子 pom 内声明的优先于父 pom 中的依赖
  ```



### 2、生命周期

- clean：清理项目
  - pre-clean：执行清理前需要完成的工作
  - clean：清理上一次构建生成的文件
  - post-clean：执行清理后需要完成的工作
- Default：构建项目
  - validate：验证工程是否正确，所有需要的资源是否可用
  - compile：编译项目的源代码
  - test：使用合适的单元测试框架来测试已编译的源代码。这些测试不需要已打包和布署
  - package：把已编译的代码打包成可发布的格式，比如 jar、war 等
  - integration-test：如有需要，将包处理和发布到一个能够进行集成测试的环境
  - verify：运行所有检查，验证包是否有效且达到质量标准
  - install：把包安装到maven本地仓库，可以被其他工程作为依赖来使用
  - deploy：在集成或者发布环境下执行，将最终版本的包拷贝到远程的repository，使得其他的开发者或者工程可以共享
- Site：建立和发布项目站点
  - pre-site：生成项目站点之前需要完成的工作
  - site：生成项目站点文档
  - post-site：生成项目站点之后需要完成的工作
  - site-deploy：将项目站点发布到服务器



### 3、Maven依赖冲突

​	依赖的包存在**不同的版本**，比如`A->B，A->C，而 B->D1.0，C->D2.0`

 - 解决方法
   	- 使用 `mvn dependency:tree`命令查看依赖树，找出存在依赖冲突的包
      	- 在引入包的时候使用`exclusions`来排除不需要的包
   	- 将想使用的包在pom文件的位置排在另一个引入不同版本的包的前面



### 4、循环依赖（相互依赖）

​	例如`A->B  B->C  C->A`

- 解决方法
  - 建一个辅助构建模块D：`D同时依赖于 A、B、C`，通过依赖D来解决，需要借助`build-helper-maven-plugin`插件把A、B、C三个模块整合在一起编译
  - 重构：
    - 平移：比如A和B互相依赖，那么可以将B依赖A的那部分代码，移动到工程B中，这样一来，B就不需要继续依赖A，只要A依赖B就可以了，从而消除循环依赖 
    - 下移：比如A和B互相依赖，同时它们都依赖C，那么可以将B和A相互依赖的那部分代码，移动到工程C里，这样一来，A和B相互之间都不依赖，只继续依赖C，也可以消除循环依赖 



### 5、继承

**父工程的打包方式必须为 pom ，子工程需指向父工程**

```xml
//父工程
<packaging>pom</packaging>
<moudles>
	<moudle>XXX</moudle>
</moudles>

//子工程
<parent>
        <artifactId>spring-cloud</artifactId>
        <groupId>com.luosico</groupId>
        <version>0.0.1-SNAPSHOT</version>
    	<relativepath>../pom.xml</relativepath> //默认为上一级目录下，若不是，要指出来
 </parent>
```

子工程默认继承 `<groupId>` 和 `<version>`，当然也可以指出，即显式声明，不推荐这种用法

`<relativepath>`的默认路径为 `../pom.xmnl`，若父工程路径不是，需要确保路径正确



#### 可继承的POM元素

- groupId    项目组ID
- version     项目版本
- **properties**     自定义的Maven属性
- **dependencies**    项目的依赖配置，**说明：依赖是可继承的**
- **dependencyManagerment**   项目的依赖管理配置
- repositories   项目的仓库配置



#### dependencies 与 **dependencyManagerment**(父工程中使用)

- dependencies

  声明的依赖都会自动传递到子工程中，缺点是有些子工程可能不需要这些依赖

- denpendencyManagement

  声明的依赖不会自动传递到子工程中，也不会给该父工程引入这些声明的依赖，这些依赖可以被子工程继承，子工程声明时不再需要声明 `version`