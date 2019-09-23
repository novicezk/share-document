title: 从零实现Java web框架
speaker: 同道-朱凯
css:
    - ./javaweb.css
plugins:
    - echarts

<slide class="bg-trans-dark aligncenter">
# 从零实现Java web框架 {.text-landing.text-shadow}
## 同道 · 朱凯 {.text-intro.animated.fadeInUp.delay-500}

<slide class="bg-trans-dark aligncenter">
### **一、一些常用技术**
- `Web服务器` Tomcat、Jetty等处理http协议的web容器
- `Spring常用功能` IOC、AOP、Spring MVC、计划任务、事件等
- `数据库相关` JPA、事务、数据库连接池

<slide class="bg-trans-dark aligncenter">
### **使用java基础类库该如何实现这些技术？**
[:fa-github: Github](https://github.com/novicezk/fast-rest)

<slide class="bg-trans-dark aligncenter">
## 二、实现简单的http服务器

<slide class="bg-trans-dark aligncenter">
### http服务器 [demo](https://github.com/novicezk/http-server-demo)
- `BIO` 使用java.net的 ServerSocket，同步阻塞
- `NIO` 使用java.nio.channels的 Selector、ServerSocketChannel，同步非阻塞

<slide class="bg-trans-dark aligncenter">
### 实现步骤
- Socket接收http请求
- 读取http报文内容，生成HttpRequest对象
- 开启新线程，处理业务逻辑
- 根据处理结果拼装HttpResponse
- 返回响应内容

<slide class="bg-trans-dark aligncenter">
## 三、实现spring的常用功能

<slide class="bg-trans-dark aligncenter">
### 3.1 java包扫描
```java
	public static List<Class> getAllClasses(String packageName) throws Exception {
		List<Class> classes = new ArrayList<>();
		String packageDirName = packageName.replace('.', '/');
		Enumeration<URL> dirs = ClassLoader.getSystemClassLoader().getResources(packageDirName);
		if (dirs.hasMoreElements()) {
			URL url = dirs.nextElement();
			if (dirs.hasMoreElements()) {
				throw new PackageRepeatException(packageName + " is repeated, please change the package name");
			}
			String protocol = url.getProtocol();
			if ("file".equals(protocol)) {
				String filePath = URLDecoder.decode(url.getFile(), "utf-8");
				findClassInPackageByFile(packageName, filePath, classes);
			} else if ("jar".equals(protocol)) {
				try {
					findClassInPackageByJar(packageDirName, url, classes);
				} catch (NoClassDefFoundError error) {
					throw new PackageRepeatException(packageName + " is repeated, please change the package name");
				}
			}
		}
		return classes;
	}
```

<slide class="bg-trans-dark aligncenter">
### 3.2 依赖注入
```java
	private static void registerComponentBean(Class beanClass, String registerName) {
		registerName = registerName.equals("") ? beanClass.getName() : registerName;
		ComponentBean componentBean = new ComponentBean();
		componentBean.setBeanClass(beanClass);
		componentBean.setSingleton(!ReflectUtil.existAnnotation(beanClass, UnSingleton.class));
		List<ChildBean> children = new ArrayList<>();
		Field[] fields = beanClass.getDeclaredFields();
		for (Field field : fields) {
			if (field.isAnnotationPresent(Autowired.class)) {
				String childBeanName = field.getAnnotation(Autowired.class).value();
				String beanName = childBeanName.equals("") ? field.getType().getName() : childBeanName;
				ChildBean childBean = new ChildBean();
				childBean.setFieldName(field.getName());
				childBean.setBeanClass(field.getType());
				if (ReflectUtil.existAnnotation(field.getType(), Component.class)) {
					childBean.setBeanFactory(ComponentBeanFactory.getInstance());
				} else if (ReflectUtil.existAnnotation(field.getType(), Configure.class)) {
					childBean.setBeanFactory(ConfigureBeanFactory.getInstance());
				} else if (Properties.class.isAssignableFrom(field.getType())) {
					beanName = childBeanName.equals("") ? Constants.DEFAULT_PROPERTIES : childBeanName;
					childBean.setBeanFactory(PropertiesBeanFactory.getInstance());
				}
				childBean.setRegisterName(beanName);
				children.add(childBean);
			}
		}
		componentBean.setChildren(children);
		componentBean.setRegisterName(registerName);
		ComponentBeanFactory.getInstance().registerBean(componentBean);
	}
```

<slide class="bg-trans-dark aligncenter">
### 3.3 AOP

#### AOP是面向切面的编程思想，AOP采用一种称为“横切”的技术，将涉及多业务流程的通用功能抽取并单独封装，形成独立的切面，在合适的时机将这些切面横向切入到业务流程指定的位置中。{.text-intro.animated.fadeInUp.delay-500}

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
### 3.4 Spring MVC
```java
	private static final Pattern webMethodPattern = Pattern.compile("\\{.*?}");

	private static void addWebMethod(Class webClass) {
		String webPath = "";
		if (webClass.isAnnotationPresent(RequestMapping.class)) {
			webPath = ((RequestMapping) webClass.getAnnotation(RequestMapping.class)).value();
		}
		Method[] methods = webClass.getMethods();
		for (Method method : methods) {
			if (method.isAnnotationPresent(RequestMapping.class)) {
				String methodPath = method.getAnnotation(RequestMapping.class).value();
				logger.info("Useful web method: {}{}", webPath, methodPath);
				Matcher matcher = webMethodPattern.matcher(methodPath);
				if (matcher.find()) {
					methodPath = methodPath.replaceAll("\\{[^}]*}", "([^/]+)");
				}
				webMethods.put(webPath + methodPath, method);
			}
		}
	}
```
<slide class="bg-trans-dark aligncenter">
### 3.5 计划任务
```java
public class TaskTrigger {
	private static final Logger logger = LoggerFactory.getLogger(TaskTrigger.class);
	private static final ScheduledThreadPoolExecutor executor = ExecutorFactory.getScheduledTaskExecutor();

	public static void registerTask(Method method) {
		Object bean = ComponentBeanFactory.getInstance().getBean(method.getDeclaringClass());
		Scheduled scheduled = method.getAnnotation(Scheduled.class);
		long fixedRate = scheduled.fixedRate();
		long fixedDelay = scheduled.fixedDelay();
		String cron = scheduled.cron();
		Runnable runnable = () -> {
			try {
				method.invoke(bean);
			} catch (Exception e) {
				logger.error("Scheduled task exec error", e);
			}
		};
		if (fixedRate > 0) {
			executor.scheduleAtFixedRate(runnable, fixedRate, fixedRate, scheduled.timeUnit());
			logger.info("Schedule rate task, method:{}", method);
		} else if (fixedDelay > 0) {
			executor.schedule(runnable, fixedDelay, scheduled.timeUnit());
			logger.info("Schedule delay task, method:{}", method);
		} else if (StringUtils.isNotBlank(cron)) {
			CronTask cronTask = new CronTask(cron, runnable, executor);
			executor.execute(cronTask);
			logger.info("Schedule cron task, method:{}, cron:{}", method, cron);
		}
	}
}
```

<slide class="bg-trans-dark aligncenter">
### 3.5 事件
```java
public class ListenerTrigger {
	private static final Logger logger = LoggerFactory.getLogger(ApplicationListener.class);
	private static final Map<String, ApplicationListener> listeners = Collections.synchronizedMap(new HashMap<>());

	public static void registerListener(String eventType, Method listenerMethod) {
		listeners.putIfAbsent(eventType, new ApplicationListener());
		listeners.get(eventType).attach(eventType, listenerMethod);
	}

	public static List<Future> publishEvent(ApplicationEvent event) {
		ApplicationListener listener = listeners.get(event.getEventType());
		if (listener == null) {
			logger.warn("Have no listener which event type is {}", event.getEventType());
			return Collections.emptyList();
		}
		return listener.asyncNotify(event);
	}

	public static List<Object> publishEventSync(ApplicationEvent event) throws Throwable {
		ApplicationListener listener = listeners.get(event.getEventType());
		if (listener == null) {
			logger.warn("Have no listener which event type is {}", event.getEventType());
			return Collections.emptyList();
		}
		return listener.syncNotify(event);
	}
}
```

<slide class="bg-trans-dark aligncenter">
## 四、实现简单的数据库操作

<slide class="bg-trans-dark aligncenter">
### 4.1 数据库连接池

<slide class="bg-trans-dark aligncenter">
### 4.2 类似spring data jpa的实现
```java
public interface CrudRepository<T, ID extends Serializable> {

	boolean save(T bean);

	boolean save(List<T> beanList);

	T findOne(ID id);

	boolean exists(ID id);

	boolean exists(T bean);

	boolean exists(Object[] properties);

	List<T> findAll();

	List<T> findAll(Object[] properties);

	List<T> findAll(List<ID> ids);

	long count();

	void delete(ID id);

	void delete(T bean);
}
```

<slide class="bg-trans-dark aligncenter">
## 五、其它
- 配置文件读取
- 静态文件服务
- 全局异常处理
- 文件上传下载
- SSL安全认证

<slide class="bg-trans-dark aligncenter">
### 课后练习
![](https://novicezk.github.io/q-code.jpg){:height="30%" width="30%"}