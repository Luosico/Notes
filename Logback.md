#  <center><font size=7>Logback</font></center>

----------
## 一、Logback的介绍 ##
**&nbsp;&nbsp;&nbsp;&nbsp;Logback是由log4j创始人设计的另一个开源日志组件,官方网站： http://logback.qos.ch。它当前分为下面下个模块:**  

> - **logback-core**：其它两个模块的基础模块
> - **logback-classic**：它是log4j的一个改良版本，同时它完整实现了slf4j API使你可以很方便地更换成其它日志系统如log4j或JDK14 Logging
> - **logback-access**：访问模块与Servlet容器集成提供通过Http来访问日志的功能

## 二、Logback配置介绍 ##
1. **Logger、appender及layout**
	- **Logger**: 作为日志的记录器，把它关联到应用的对应的context上后，主要用于存放日志对象，也可以定义日志类型、级别
	- **Appender**: 主要用于指定日志输出的目的地，目的地可以是控制台、文件、远程套接字服务器、 MySQL、PostreSQL、 Oracle和其他数据库、 JMS和远程UNIX Syslog守护进程等
	- **Layout**: 负责把事件转换成字符串，格式化的日志信息的输出
2. **logger context**: 各个logger 都被关联到一个 LoggerContext，LoggerContext负责制造logger，也负责以树结构排列各logger。其他所有logger也通过org.slf4j.LoggerFactory 类的静态方法getLogger取得。 getLogger方法以 logger名称为参数。用同一名字调用LoggerFactory.getLogger 方法所得到的永远都是同一个logger对象的引用
3. **有效级别及级别的继承**: Logger 可以被分配级别。级别包括：TRACE、DEBUG、INFO、WARN 和 ERROR，定义于ch.qos.logback.classic.Level类。如果 logger没有被分配级别，那么它将从有被分配级别的最近的祖先那里继承级别。root logger 默认级别是 DEBUG
4. **印方法与基本的选择规则**: 打印方法决定记录请求的级别。例如，如果 L 是一个 logger 实例，那么，语句 L.info("..")是一条级别为 INFO的记录语句。记录请求的级别在高于或等于其 logger 的有效级别时被称为被启用，否则，称为被禁用。记录请求级别为 p，其 logger的有效级别为 q，只有则当 p>=q时，该请求才会被执行。 该规则是 logback 的核心。级别排序为： TRACE < DEBUG < INFO < WARN < ERROR

## Logback的默认配置 ##
- 如果配置文件 logback-test.xml 和 logback.xml 都不存在，那么 logback 默认地会调用BasicConfigurator ，创建一个最小化配置。最小化配置由一个关联到根 logger 的ConsoleAppender 组成。输出用模式为%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n 的 PatternLayoutEncoder 进行格式化。root logger 默认级别是 DEBUG
- **Logback的配置文件**:Logback 配置文件的语法非常灵活。正因为灵活，所以无法用 DTD 或 XML schema 进行定义。尽管如此，可以这样描述配置文件的基本结构：以开头，后面有零个或多个元素，有零个或多个元素，有最多一个元素
- **Logback默认配置的步骤**
	1. 尝试在 classpath下查找文件 logback-test.xml
	2. 如果不存在，则尝试在classpath中查找 logback.groovy
	3. 如果文件不存在，则查找文件logback.xml
	4. 如果两个文件都不存在，logback用 BasicConfigurator自动对自己进行配置，这会导致记录输出到控制台
	
## 四、使用要求 ##
1. **必须导入的包：**
	> - logback-classic.jar
	> - slf4j-api.jar
	> - logback-core.jar
	
2. **基础模板**

		package chapters.introduction;
		
		import org.slf4j.Logger;
		import org.slf4j.LoggerFactory;
		
		public class HelloWorld1 {
		
		public static void main(String[] args) {
		
		    Logger logger = LoggerFactory.getLogger("chapters.introduction.HelloWorld1");
		    logger.debug("Hello world.");
		
	 		}
		}
	运行结果:  

		20:49:07.962 [main] DEBUG chapters.introduction.HelloWorld1 - Hello world.

	以上示例未引用任何logback类。在大多数情况下，就日志记录而言，您的类仅需要导入SLF4J类。因此，绝大多数（如果不是全部）您的类将使用SLF4J API，并且不会考虑logback的存在。  
	根据logback的默认配置策略，当未找到默认配置文件时，logback将ConsoleAppender在根记录器中添加一个。  
	Logback可以使用内置状态系统报告有关其内部状态的信息。可以通过称为的组件访问在logback生存期内发生的重要事件 StatusManager。目前，让我们通过调用类的静态print()方法来指示logback打印其内部状态 StatusPrinter 。
	

		package chapters.introduction;
			
		import org.slf4j.Logger;
		import org.slf4j.LoggerFactory;
		import ch.qos.logback.classic.LoggerContext;
		import ch.qos.logback.core.util.StatusPrinter;
			
		public class HelloWorld2 {
			
			public static void main(String[] args) {
			Logger logger = LoggerFactory.getLogger  ("chapters.introduction.HelloWorld2");
		    logger.debug("Hello world.");
		
		    // print internal state
		    LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory();
		    StatusPrinter.print(lc);
		  }
		}
	运行结果：

		12:49:22.203 [main] DEBUG chapters.introduction.HelloWorld2 - Hello world.
		12:49:22,076 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback.groovy]
		12:49:22,078 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback-test.xml]
		12:49:22,093 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback.xml]
		12:49:22,093 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Setting up default configuration.

	Logback解释说，未能找到 logback-test.xml和logback.xml配置文件（稍后讨论），因此使用默认策略（基本）对自身进行了配置ConsoleAppender。An Appender是可以视为输出目标的类。附加程序存在于许多不同的目的地，包括控制台，文件，Syslog，TCP套接字，JMS等。用户还可以根据自己的具体情况轻松创建自己的Appender。
	
	请注意，如果出现错误，logback将自动在控制台上打印其内部状态。
	
	前面的例子很简单。在更大的应用程序中的实际登录不会有太大不同。记录语句的一般模式不会改变。只有配置过程会有所不同。但是，您可能需要根据需要自定义或配置登录。后续章节将介绍Logback配置。
	
	请注意，在上面的示例中，我们已指示logback通过调用该StatusPrinter.print()方法来打印其内部状态 。Logback的内部状态信息对于诊断与Logback相关的问题非常有用。

3. **使用步骤**
	1. 配置登录环境。您可以通过几种或多或少的复杂方法来执行此操作
	2. 在希望执行日志记录的每个类中，Logger通过调用org.slf4j.LoggerFactory该类的 getLogger()方法，将当前类名或该类本身作为参数传递来检索实例 
	3. 通过调用其记录方法(即debug()，info()，warn()和error()方法)使用此记录器实例。这将在配置的附加程序上生成日志记录输出

4. **使用级别**

			TRACE < DEBUG < INFO < WARN < ERROR
	设置级别

			logger.setLevel(Level. INFO);
	使用的级别不能低于设置的级别

		import ch.qos.logback.classic.Level;
		import org.slf4j.Logger;
		import org.slf4j.LoggerFactory;
		....
		
		// get a logger instance named "com.foo". Let us further assume that the
		// logger is of type  ch.qos.logback.classic.Logger so that we can
		// set its level
		ch.qos.logback.classic.Logger logger = (ch.qos.logback.classic.Logger) LoggerFactory.getLogger("com.foo");
		//set its Level to INFO. The setLevel() method requires a logback logger
		logger.setLevel(Level. INFO);
		
		Logger barlogger = LoggerFactory.getLogger("com.foo.Bar");
		
		// This request is enabled, because WARN >= INFO
		logger.warn("Low fuel level.");
		
		// This request is disabled, because DEBUG < INFO. 
		logger.debug("Starting search for nearest gas station.");
		
		// The logger instance barlogger, named "com.foo.Bar", 
		// will inherit its level from the logger named 
		// "com.foo" Thus, the following request is enabled 
		// because INFO >= INFO. 
		barlogger.info("Located nearest gas station.");
		
		// This request is disabled, because DEBUG < INFO. 
		barlogger.debug("Exiting gas station search");
