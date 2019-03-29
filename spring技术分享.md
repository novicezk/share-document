title: Spring 技术分享
speaker: 同道-朱凯
css:
    - ./spring.css
plugins:
    - echarts

<slide class="bg-trans-dark aligncenter">

# Spring 技术分享 {.text-landing.text-shadow}
## 同道 · 朱凯 {.text-intro.animated.fadeInUp.delay-500}


<slide class="bg-trans-dark aligncenter">

### **一、Spring 系列常用框架简介**
- `Spring Framework` 即通常所说的spring 框架，是一个开源的Java/Java EE全功能栈应用程序框架，其它spring项目也依赖于此框架 
- `Spring Data` 是一个数据访问及操作的工具包，封装了很多种数据及数据库的访问相关技术，包括：jdbc、Redis、MongoDB等
- `Spring Security` 安全访问控制解决方案的安全框架
- `Spring Session` session管理的开发工具包，可以把session保存到redis等，进行集群化session管理
- `Spring Boot` 简化Spring应用的初始搭建以及开发过程，使用特定的方式进行配置，使开发人员不再需要定义样板化的配置，实现快速开发
- `Spring Cloud` 为分布式系统开发提供工具集，基于Spring Boot，为应用开发中的配置管理、服务发现、断路器、智能路由、分布式会话等操作提供了一种简单的开发方式
{.build.moveIn}

<slide class="bg-trans-dark aligncenter">
## 二、Spring Framework 5.0

<slide class="bg-trans-dark">

:::column {.vertical-align}

![](https://lfvepclr.gitbooks.io/spring-framework-5-doc-cn/content/assets/spring-overview.png.pagespeed.ce.XVe1noRCMt.png)

----
### **Spring 模块**
1. `核心` Spring的核心类库，Spring的所有功能都依赖于该类库。包括IOC/DI、AOP、SpEL等
2. `数据访问/集成` 包括事务、JDBC、ORM等
4. `Web` 包括Spring MVC、WebSocket、Spring WebFlux等
5. `测试` spring-test 用于项目的单元和继承测试

:::

<slide class="bg-trans-dark aligncenter">
## 2.1 IOC/DI
### 由spring来负责控制对象的生命周期和对象间的关系 {.text-intro}

<slide class="bg-trans-dark">

## 常用注入方式 {.aligncenter}

- `构造方法注入`  Spring推荐首选此注入方式。因为它支持将应用程序组件以作为不可变对象来实现，并确保所需的依赖项不是null
```java
    private final Test2 test2;
    // @Autowired 只有一个构造函数可不加Autowired
	public Test1(Test2 test2) {
		this.test2 = test2;
	}
```
- `setter方法注入`  setter方法可以被继承，允许设置默认值。 也可用于解决循环依赖问题
```java
	private Test2 test2;

	@Autowired
	public void setTest2(Test2 test2) {
		this.test2 = test2;
	}
```
<slide class="bg-trans-dark">

## 两种IOC容器 {.aligncenter}

<slide class="bg-trans-dark">
- `BeanFactory` 基础类型IoC容器，提供完整的IoC服务支持。如果没有特殊指定，默认采用延迟初始化策略（lazy-load）。只有当客户端对象需要访问容器中的某个受管对象的时候，才对该受管对象进行初始化以及依赖注入操作。所以，相对来说，容器启动初期速度较快，所需要的资源有限。对于资源有限，并且功能要求不是很严格的场景，BeanFactory是比较合适的IoC容器选择。

```java
        var factory = new DefaultListableBeanFactory();
		var bean = BeanDefinitionBuilder.genericBeanDefinition(Test2.class).getBeanDefinition();
		factory.registerBeanDefinition("hhh", bean);
		System.out.println(factory.getBean(Test2.class));
```

<slide class="bg-trans-dark">
- `ApplicationContext` 在BeanFactory的基础上构建，是相对比较高级的容器实现，除了拥有BeanFactory的所有支持，ApplicationContext还提供了其他高级特性，比如事件发布、国际化信息支持等，ApplicationContext所管理的对象，在该类型容器启动之后，默认全部初始化并绑定完成。所以，相对于BeanFactory来说，ApplicationContext要求更多的系统资源，同时，因为在启动时就完成所有初始化，容器启动时间较之BeanFactory也会长一些。在那些系统资源充足，并且要求更多功能的场中，ApplicationContext类型的容器是比较合适的选择。
```java
        var ctx = new AnnotationConfigApplicationContext();
		ctx.scan("com.novice.learn.spring");
		ctx.refresh();
		System.out.println(ctx.getBean(Test1.class));
```

<slide class="bg-trans-dark aligncenter">

![](https://novicezk.github.io/spring-ioc-uml.jpg)

<slide class="bg-trans-dark aligncenter">
## 2.2 AOP

### AOP是Spring框架面向切面的编程思想，AOP采用一种称为“横切”的技术，将涉及多业务流程的通用功能抽取并单独封装，形成独立的切面，在合适的时机将这些切面横向切入到业务流程指定的位置中。{.text-intro.animated.fadeInUp.delay-500}

<slide class="bg-trans-dark">

:::column {.vertical-align}

![](https://novicezk.github.io/spring-aop-uml.jpg)

----
#### ProxyFactory 创建代理时会自动在JDK动态代理和CGLIB之间转换
- `JDK` 目标对象实现了接口，或只代理接口，没有设置目标对象
- `CGLIB` 目标对象没有实现接口，或强制使用

:::

<slide class="bg-trans-dark">
#### cglib 代理demo
利用ASM开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理
```java
public class CglibProxy implements MethodInterceptor {

	public <T> T getProxyInstance(Class<T> clazz) {
		var enhancer = new Enhancer();
		enhancer.setSuperclass(clazz);
		enhancer.setCallback(this);
		return clazz.cast(enhancer.create());
	}

	@Override
	public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
		System.out.println(method.getName() + "调用前");
		var result = proxy.invokeSuper(obj, args);
		System.out.println(method.getName() + "调用后");
		return result;
	}

	public static void main(String[] args) {
		var proxy = new CglibProxy();
		var test = proxy.getProxyInstance(TestClass.class);
		test.hello();
	}

}
```

<slide class="bg-trans-dark">
#### jdk 动态代理demo
利用拦截器(实现InvocationHanlder)加上反射机制生成一个实现代理接口的匿名类
```java
public class JdkProxy implements InvocationHandler {

	public <T> T getProxyInstance(Class<T> mapperInterface) {
		var object = Proxy.newProxyInstance(this.getClass().getClassLoader(), new Class[]{mapperInterface}, this);
		return mapperInterface.cast(object);
	}

	@Override
	public Object invoke(Object proxy, Method method, Object[] args) {
		System.out.println(method.getName() + "调用");
		return null;
	}

	public static void main(String[] args) {
		var proxy = new JdkProxy();
		var test = proxy.getProxyInstance(TestClass.class);
		test.hello();
	}
}
```

<slide class="bg-trans-dark aligncenter">
### 使用ProxyFactory生成代理类

<slide class="bg-trans-dark">
#### 代理对象未指定接口，使用CGLIB生成代理类
```java
        var factory = new ProxyFactory();
		factory.setTarget(new TestClass());
		factory.addAdvice(getMethodInterceptor());

		TestClass testClass = (TestClass) factory.getProxy();
		System.out.println(testClass.getClass().getName());
		testClass.hello();
```

<slide class="bg-trans-dark">
#### 代理对象指定接口TestInterface，目标类为实现TestInterface的TestImpl，使用JDK proxy生成代理类
```java
        var factory = new ProxyFactory();
		factory.setTarget(new TestImpl());
		factory.setInterfaces(TestInterface.class);
		factory.addAdvice(getMethodInterceptor());

		TestInterface testInterface = (TestInterface) factory.getProxy();
		System.out.println(testInterface.getClass().getName());
		testInterface.hello();
```

<slide class="bg-trans-dark">
#### 代理接口，不指定目标对象，使用JDK proxy生成代理类
```java
        var factory = new ProxyFactory();
		factory.setInterfaces(TestInterface.class);
		factory.setTargetClass(TestInterface.class);
		factory.addAdvice((MethodInterceptor) invocation -> {
			System.out.println(invocation.getMethod().getName() + "调用");
			return null;
		});
        var proxy = (TestInterface) factory.getProxy();
		System.out.println(proxy.getClass().getName());
		proxy.hello();
```

<slide class="bg-trans-dark aligncenter">
### aop 常用场景 

- Authentication 权限
- Error handling 错误处理
- logging, tracing, profiling and monitoring　记录跟踪　优化　校准
- Performance optimization　性能优化
- Persistence　　持久化
- Resource pooling　资源池
- Synchronization　同步
- Transactions 事务

<slide class="bg-trans-dark aligncenter">
#### 示例一 rest 接口的全局异常处理 
```java
@Slf4j
@ControllerAdvice
public class GlobalExceptionHandler {

	@ResponseBody
	@ExceptionHandler(Throwable.class)
	public Message<String> errorHandler(Throwable e) {
		log.error("system error", e);
        return MessageBuilder.build(MessageCode.SYSTEM_ERROR, e.getMessage());
	}
}
```

<slide class="bg-trans-dark">
#### 示例二 接口权限检查（建议改用Spring Security）
```java
@Slf4j
@Aspect
@Component
@RequiredArgsConstructor(onConstructor = @__(@Autowired))
public class RestGlobalHandler {
	private final StudentFilter studentFilter;
	private final HttpSession session;

	@Around("execution(* com.lenchy.apps.lenchy.learn.service.rest.*.*(..))" +
			"&& @annotation(com.homolo.framework.rest.ActionMethod)")
	public Object around(ProceedingJoinPoint pjp) {
		try {
			MethodSignature signature = (MethodSignature) pjp.getSignature();
			if (signature.getMethod().getAnnotation(PermitAll.class) == null) {
				Entity student = this.studentFilter.currentStudent();
				if (student == null) {
					return new ServiceResult(-5, "未登录或登录过期");
				}
				this.session.setAttribute("student", student);
			}
			return pjp.proceed();
		} catch (Throwable e) {
			LOGGER.error("{} rest error", pjp.getSignature().getName(), e);
			return new ServiceResult(ReturnResult.FAILURE, "系统异常");
		}
	}

}

```

<slide class="bg-trans-dark aligncenter">
## 三、Spring Data

<slide class="bg-trans-dark aligncenter">
#### Spring 的一个子项目。用于简化数据库访问，支持NoSQL和关系数据库存储。其主要目标是使数据库的访问变得方便快捷，Spring Data 包含多个子项目：
- `spring-data-commons`  提供共享的基础框架，适合各个子项目使用，支持跨数据库持久化
- `spring-data-jpa`  在ORM框架以及JPA规范的基础上，封装的一套JPA应用框架
- `spring-data-redis`  集成了 Redis，提供多个常用场景下的简单封装
- `spring-data-mongodb`  集成MongoDB 并提供基本的配置映射和资料库支持

<slide class="bg-trans-dark aligncenter">
#### `Repository`  是Spring Data 的核心接口，大部分数据库操作都是通过该接口操作。其他的Repository都继承实现该接口，如CrudRepository，其他特定技术的抽象如JpaRepository、 MongoRepository都是CrudRepository的子类。


<slide class="bg-trans-dark aligncenter">
### CrudRepository 
```java
public interface CrudRepository<T, ID extends Serializable> extends Repository<T, ID> {

  <S extends T> S save(S entity);      

  Optional<T> findById(ID primaryKey); 

  Iterable<T> findAll();               

  long count();                        

  void delete(T entity);               

  boolean existsById(ID primaryKey); 

  //...  

}
```

<slide class="bg-trans-dark aligncenter">
### 定义Repository接口及使用
```java
// 需要指定实体类型和id类型，可按规则定义查询方法
public interface TestUserRepository extends MongoRepository<TestUser, String> {
	List<TestUser> findByNameContains(String containsStr);
}

```
```java
@RestController
@RequiredArgsConstructor(onConstructor = @__(@Autowired))
public class TestController {
	private final TestUserRepository testUserRepository;

	@RequestMapping("/findName/{name}")
	public List<TestUser> findAllUser(@PathVariable String name) {
		return testUserRepository.findByNameContains(name);
	}
}

```
<slide class="bg-trans-dark aligncenter">
### `思考题`  
#### Repository接口没有具体实现类，是怎么操作数据库并返回数据的？


<slide class="bg-trans-dark aligncenter">
## 四、Spring Boot
#### Spring Boot是为了简化Spring应用的创建、运行、调试、部署等而出现的，使用它可以做到专注于Spring应用的开发，而无需过多关注XML的配置。简单来说，它提供了一堆依赖打包，并已经按照使用习惯解决了依赖问题---习惯大于约定。{.text-intro.animated.fadeInUp.delay-500}

<slide class="bg-trans-dark aligncenter">
### 如何创建一个Spring Boot项目

<slide class="bg-trans-dark aligncenter">
#### 1. 创建maven项目，添加配置
```xml
 	<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.1.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.2</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

<slide class="bg-trans-dark aligncenter">
#### 2. 创建App类，放在最外层的包下
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```
<slide class="bg-trans-dark aligncenter">
#### 3. 创建web/HelloController
```java
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String hello() {
        return "hello";
    }

}
```

<slide class="bg-trans-dark aligncenter">
#### 4. 直接运行Application的main函数，访问 `http://localhost:8080/hello`

<slide class="bg-trans-dark aligncenter">
#### spring boot代码结构
- root package结构：com.example.myproject
- 应用主类Application.java置于root package下，通常会在应用主类中做一些框架配置扫描等配置
- 实体（Entity）与数据访问层（Repository）置于com.example.myproject.domain包下
- 逻辑层（Service）置于com.example.myproject.service包下
- Web层（web）置于com.example.myproject.web包下

<slide class="bg-black aligncenter" image="https://source.unsplash.com/n9WPPWiPPJw/ .anim">

## Thanks for watching
### novicezk {.text-intro.animated.fadeInUp.delay-500}

[:fa-github: Github](https://github.com/novicezk){.button.ghost.animated.flipInX.delay-1200}