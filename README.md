# Spring笔记

### Spring配置文件一般命名为：applicationContext.xml

```xml
<!-- 声明bean标签，就是告诉Spring要创建某个对象
    id：对象的自定义名称，唯一值。spring通过这个名称找到对象
    class：类的引用（不能引用接口，Spring是通过反射机制创建对象，必须使用类
    Spring把创建好的类放进map中，Spring框架又一个map存放对象
    Spring.put（id的值，对象）
    一个bean标签声明一个对象
-->
<bean id="student" class="com.lushiwu.Student"/>
<!-- 上面语句等价于 Student student = new Student -->
```

### 使用Spring容器创建对象

```java
		/*创建表示Spring容器的对象，ApplicationContext就是Spring容器，通过容器获取对象
		* ClassPathXmlApplicationContext类是ApplicationContext接口的实现类
		* ClassPathXmlApplicationContext加载Spring配置文件
		* */
		String config = "applicationContext.xml";
    ApplicationContext ac = new ClassPathXmlApplicationContext(config);
		// 从容器中获取对象,用getBean（配置文件中的bean标签的id值）来获取对象
		SomeService serviceIml = (SomeService)ac.getBean("someServiceImpl");
		// 用对象名调用对象方法
		serviceIml.doSome();
		// getBeanDefinitionCount()获取对象数量
		// getBeanDefinitonNames（）获取对象方法名
```

### 在Spring配置文件给对象属性赋值

#### 使用Spring标签和属性赋值叫做基于XML的di实现

##### di分为set注入、构造注入对bean标签创建对象进行初始化

###### 1.set注入（设置注入）：Spring调用类的set方法，在set方法给属性赋值；set注入就是调用set方法。

```xml
<!-- 1.简单类型的注入（赋值）；Spring规定Java基本类型和String都是简单类型。
		一条property标签只能给一个属性赋值
-->
    <bean id="xx" class="yyy">
        <property name="属性名" value="赋值"/>
    </bean>
<!-- 2.引用类型注入 -->
		<bean id="xx" class="yyy">
        <property name="属性名" ref="bean标签id（对象名）"/>
    </bean>

```

###### 	2.构造注入：Spring调用类的有参构造方法，创建对象。在构造方法中赋值。

```xml
<!-- 构造注入使用<constructor-arg> 标签
		一个<constructor-arg>标签表示一个行参
		<constructor-arg>标签的属性：
				name:表示行参的位置，从左往右0开始
				value:行参类型是简单类型使用value
				ref:引用类型使用ref
-->	
		<bean id="xx" class="yyy">
        <constructor-arg name="行参名" value="赋值"/>
        <constructor-arg name="行参名" ref="bean标签id（对象名）"/>
      	<constructor-arg name="行参0" value="赋值"/>
        <constructor-arg name="行参1" ref="bean标签id（对象名）"/>
		</bean>
```

###### 	3.引用类型的自动注入byName、byType

​		1.byName（按名注入）：Java类中属性名和Spring容器（配置文件）bean标签的id名词一样，且类型相同

​		2.byType（按类型注入）：Java类中引用类型的数据类型和Spring（配置文件）bean的class属性是同源关系
​				1.引用类型的基本数据和bean的class的值一样的

​				2.引用类型的基本数据和bean的class的值父子类关系。

​				3.引用类型的基本数据和bean的class的值接口和实现类关系

```xml
   <!-- byType中，符合条件的bean只能有一个 -->
		<bean id="xx" class="yy" autowire="byName/byType">
        <property name="属性名" value="赋值"/>
    </bean>
```



### 基于注解的di：通过注解完成对象创建，属性赋值

##### 使用步骤：

​		1.加入maven依赖：spring-context ，spring-aop依赖，加载第一个依赖间接加入了第二个依赖

​		2.在类中加入spring的注解

​		3.在Spring配置文件中，加入一个组件扫描器标签，说明注解在你项目中的位置

​			组件扫描器：组件就是Java对象；

​					base-package：指定注解在你项目中的包名；

​					component-scan：Spring会扫描便利base-package指定的包；

​					把包中和子包中所有类，找到类中的注解，按照注解创建对象或者给属性赋值

```xml
    <context:component-scan base-package="包名"/>
```

@Component:创建对象，和bean等价；

​		属性：value是对象的名称，也就是bean的id值；

​					value的值唯一，创建的对象在Spring容器中只有一个

​		位置：在类的上面

@Value:简单类型的属性赋值

​		属性：value 是String类型的，表示简单类型的属性值

​		位置： 在属性定义的上面，无需set方法，推荐使用；也可以使用set方法上面	

@Autowired:Spring框架提供的注解，引用类型的赋值

​		默认使用的是byType自动注入

​		属性：required = true :表示引用类型赋值失败，程序报错，并终止执行。（推荐使用true）

​					required = false：表示引用类型赋值失败，程序正常执行，赋值为null

​		位置：在属性定义的上面，无需set方法，推荐使用；也可以使用set方法上面	

​		使用byName方式：

​				在属性上加@Autowrired和@Qualifier（没有先后顺序）

​				@Qualifier（value=“bean的id”）表示指定名称的bean或注解赋值

@Resource:来自JDK中的注解，引用类型的赋值

​		默认是先使用byName自动注入，失败后再使用byType

```java
@Component(value="对象名") // value可以省略不写，也可以不写默认是类名首字母小写
public class Student{
  	@Value(value = "赋值")
		private String name;
  	@Autowired
  	//@Qualifier（“bena的id值”）
  	private School	//引用类型属性的类必须加注解或配置bean标签 
    @Resource // 只使用byName方式加上后面代码(name = "bean的id值")
    private School
}
```

@Repository：在持久层类上使用，创建dao对象，dao对象是访问数据库的

@Service：在业务层类上使用，创建service对象，service对象是做业务处理可以有事务功能的

@Controller：在控制器上使用，创建控制器对象，能够接受用户提交的参数，显示请求的处理结果

以上三种注释语法与@Component语法相同。

>   ### 需要经常改的使用配置文件，不经常改的用注解
>
>   注解为主，配置为辅

### AOP面向切面编程

使用aspectj框架实现aop：

使用aop的目的是给已存在的类和方法，增加额外的功能。不改变原来的类的代码。

#### 切入点表达式：

```java
execution(访问权限 方法返回值 方法声明（参数） 异常类型)//访问权限和异常类型可以没有
//  *：代表0或多个任意字符
//  ..：用在包名后，表示当前包以及其子包路径；用在方法参数中，表示任意个参数。
execution（public **(..))// 任意公共方法
execution(* set*(..)) // 任何一个以set开始的方法
execution(* com.xyz.service.*.*(..)) // service包里任意类的任意方法
execution(* com.xyz.service..*.*(..)) // service包及其子包里任意类的任意方法 
execution(* *..service.*.*(..)) // 所有包下的service子包的所有类（接口）中所有的方法
```

#### 实现aop的步骤：

​	1.maven加入Spring、aspectj依赖，junit单元测试

​	2.创建目的类：接口和实现类。给类中的方法增加功能。

​	4.创建切面类：在类上加@Aspect注解，表示当前类是切面类。

​		切面类是给业务增加功能的类	

​		在类中定义方法，方法就是切面要执行的功能代码

​		在方法的方面加入aspect中的通知注解，例如@Before	

```java
@Aspect
public class MyAspect {
  // 方法要求：公共方法，没有返回值，名称自定义，可以有行参也可以没有
  
  /** @Pointcut:定义管理切入点，如果有多个重复的切入点，使用@Pointcut
  		当使用@Pointcut定义在一个方法上面，此时这个方法名称就是切入点表达式的别名
  		其他通知中，value的属性就可以使用这个方法名称，代替切入点表达式
  */
  @Pointcut(value="切入点表达式")
  public void mypt(){
    // 无需写代码
  }
  //使用切入点表达式
  @Before(value="mypt")
  public void myBefore(){
  	
  }
  
  // @Before前置通知注解  
  // JoinPoint：获取目标方法的方法名，方法参数；需要时加，不需要可以不加
  @Before(value="切入点表达式")
  public void myBefore(JoinPoint jp){ // JoninPoint参数必须是第一个位置
   	Sytem.out.println("方法的签名" + jp.getSignature()); // 方法返回值 方法声明（参数）
    System.out.println("方法名" + jp.getSignature().getName); // 方法名
    Object args[] = jp.getArgs; // 方法的参数
  	System.out.pritln("在目标方法前执行");
  }
  
  // @AfterReturning后置通知注解: returning 表示目标方法执行后的返回值
  @AfterReturning(value = "切入点表达式", returning = "res")
  public void myAfterReturing(object res){
    System.out.pritln("在目标方法后执行");
  }
 
  // @Around环绕通知:必须有一个返回值：推荐objce 
  @Around(value="切入点表达式")
  public Object myAround(ProceedingJoinPoint pjp){ // 固定参数ProceedingJoinPoint
		Objcet result = null;
    System.out.pritln("环绕通知在目标方法前执行");
    result = pjp.proced(); // 调用目标对象
    System.out.pritln("环绕通知在目标方法后执行");
    return result;
  }

  // @After最终通知 总是会执行，在目标方法执行之后
  @After(value="切入点表达式")
  public void myAfter(){
    	System.out.prinlt("执行最终通知")
      // 一般做资源清理
  }
}
```

5.创建Spring配置文件：声明对象，把对象交给容器统一管理

​		1）声明目标对象

​		2）声明切面对象

​		3）声明aspect框架中的自动代理生成器标签；自动代理生成器：用来完成代理对象的自动创建功能。

​			有接口默认使用JDK 动态代理，没接口使用cglib代理

```xml
		<!-- 自动代理生成器 -->   
		<aop:aspectj-autoproxy/>
  	<!-- 强制使用cglib代理 
					proxy-target-class="true" :告诉框架，要使用cglib动态代理
		-->
   <aop:aspectj-autoproxy proxy-target-class="true"/>
```
### Spring事物

事务是指一组sql语句的集合，集合中有多条sql语句，我们希望这些sql语句集合都能执行成功或者失败，

这些sql语句执行是一致，作为一个整体。

事务应该放在service类的业务方法上，业务方法会调用多个dao方法，执行sql语句

Spring提供处理事务的统一模型，能使用统一步骤，方式完成多种不同数据库访问技术的事务处理。

Spring处理事务的模型，使用的步骤都是固定的。把事务的信息提供给Spring就可以了

1）事务内部提交，回滚事物，使用事务管理器对象，代替你完成commit，rollback

​	事物管理器是一个接口和他的众多实现类。

​	接口：PlatformTransactionMannager，定义了事务重要方法 commit，rollback

​	实现类：Spring把每种数据库访问技术相对应的事物处理类都创建好了

​				mybatis访问数据库——Spring创建好的类是DataSourceTransactionMannger

​				hibernate访问数据库——Spring创建好的类是HibernateTransactionMannger

​	只用你告诉Spring你用的那种数据库的访问技术就可以使用，怎么告诉Spring呢？

​	声明数据库访问技术对应的事物管理器实现类，使用Spring配置文件中的bean标签声明

```xml
<bean id="xxx" class="...DataSoureceTransactionMannger">
```
#### 说明方法需要的事物

##### 		1.事物的隔离级别有4个值

​		DEFAULT：采用DB默认的事物隔离。

​		Mysql的默认为 REPEATABLE_READ

​		Oracle的默认为 READ_COMMITTED

​		READ_UNCOMMITTED: 读未提交，未解决任何并发问题。

​		READ_COMMITTED: 读已提交，解决脏读，存在不可重复读与幻读

​		REPEATABLE_READ ：可重复读，解决脏读，不可重复读，存在幻读

​		SERIALIZABLE： 串行化。不存在并发问题。



##### 		2.事物超时时间

​		表示一个方法最长执行时间，如果方法执行时超过了时间，事物就会滚，单位是秒，整数值，默认是-1

##### 		3.事物的传播行为

​		7个传播行为，表示你的业务方法调用时，事物之间时如何使用的。掌握前三个，后面几乎不用

| 事务传播行为类型          | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED      | 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。Spring默认事物传播方式。 |
| PROPAGATION_REQUIRES_NEW  | 新建事务，如果当前存在事务，把当前事务挂起。                 |
| PROPAGATION_SUPPORTS      | 支持当前事务，如果当前没有事务，就以非事务方式执行。         |
| PROPAGATION_NESTED        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。 |
| PROPAGATION_NEVER         | 以非事务方式执行，如果当前存在事务，则抛出异常。             |
| PROPAGATION_MANDATORY     | 使用当前的事务，如果当前没有事务，就抛出异常。               |
| PROPAGATION_NOT_SUPPORTED | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。   |

#### 事物提交事物，回滚事物的时机

​	1.当你的业务方法，执行成功，没有抛出异常，Spring在方法执行后提交事物。事物管理器commit

​	2.当你的业务方法抛出运行时异常或ERROR，Spring执行回滚，调用事物管理器的rollback

​		运行时异常定义：RuntimeException 和 他的子类都是运行时异常，例如NullPointException

​	3.当你的业务方法抛出非运行异常，主要是受查异常时，提交事物

​		受查异常：在你写的代码汇总，必须处理的异常，例如IOException。SQLException

### 总结Spring事物

1.管理事物的是事物管理器和他的实现类

2.Spring的事物是一个统一模型

​	1.指定要使用的事物管理器实现对象，使用<bean>

​	2.指定那些类，哪些方法需要加入事物的功能

​	3.指定方法需要的隔离级别，传播行为，超时

