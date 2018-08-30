###一、spring boot嵌入式servlet容器运行原理:

##### 1）SpringBoot应用启动时运行主程序的run方法。

##### 2）调用SpringApplication的run()方法，其中调用refreshContext(context);

###### SpringBoot刷新IOC容器【创建IOC容器对象，并初始化容器，创建容器中的每一个组件】,如果是web应用创建AnnotationConfigEmbeddedWebApplicationContext,否则创建AnnotationConfigApplicationContext;
```java
    StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		FailureAnalyzers analyzers = null;
		configureHeadlessProperty();
		SpringApplicationRunListeners listeners = getRunListeners(args);
		listeners.starting();
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(
					args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners,
					applicationArguments);
			Banner printedBanner = printBanner(environment);
			context = createApplicationContext();
			analyzers = new FailureAnalyzers(context);
			prepareContext(context, environment, listeners, applicationArguments,
					printedBanner);
            //调用此方法
			refreshContext(context);
			afterRefresh(context, applicationArguments);
			listeners.finished(context, null);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass)
						.logStarted(getApplicationLog(), stopWatch);
			}
			return context;
		}
```

##### 3）在refreshContext(ConfigurableApplicationContext context)方法中调用refresh(ApplicationContext applicationContext)方法,在refresh(ApplicationContext applicationContext)方法中调用AbstractApplicationContext的refresh()方法刷新刚才创建好的ioc容器；

##### 4）在refresh()方法中，调用onRefresh()方法,子类的EmbeddedWebApplicationContext重写了父类的onRefresh()方法；
```java
synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}
EmbeddedWebApplicationContext的onRefresh()方法
@Override
	protected void onRefresh() {
		super.onRefresh();
		try {
			createEmbeddedServletContainer();
		}
		catch (Throwable ex) {
			throw new ApplicationContextException("Unable to start embedded container",
					ex);
		}
	}
```

##### 5)在子类的onRefresh方法中调用createEmbeddedServletContainer()方法，创建嵌入式servlet容器EmbeddedServletContainer
##### 6)在createEmbeddedServletContainer()方法中获取嵌入式的Servlet容器工厂：
```java
EmbeddedServletContainerFactory containerFactory = getEmbeddedServletContainerFactory();
```
###### 从ioc容器中获取EmbeddedServletContainerFactory组件,并创建子类的TomcatEmbeddedServletContainerFactory创建对象，后置处理器一看是这个对象，就获取所有的定制器来先定制Servlet容器的相关配置；

![](/img/1.png)

##### 7)使用容器工厂获取嵌入式的Servlet容器:this.embeddedServletContainer = containerFactory.getEmbeddedServletContainer(getSelfInitializer());
```java
	private void createEmbeddedServletContainer() {
		EmbeddedServletContainer localContainer = this.embeddedServletContainer;
		ServletContext localServletContext = getServletContext();
		if (localContainer == null && localServletContext == null) {
			EmbeddedServletContainerFactory containerFactory = getEmbeddedServletContainerFactory();
			//调用此方法，获取servlet容器
			this.embeddedServletContainer = containerFactory
					.getEmbeddedServletContainer(getSelfInitializer());
		}
		else if (localServletContext != null) {
			try {
				getSelfInitializer().onStartup(localServletContext);
			}
			catch (ServletException ex) {
				throw new ApplicationContextException("Cannot initialize servlet context",
						ex);
			}
		}
		initPropertySources();
	}
```
##### 8)子类TomcatEmbeddedServletContainerFactory通过getEmbeddedServletContainerFactory()创建Tomcat对象,通过run()方法启动Servlet容器；先启动嵌入式的Servlet容器，再将ioc容器中剩下没有创建出的对象获取出来；
```wiki
先创建IOC容器，同时创建嵌入式的Servlet容器，之后在去创建IOC容器的剩下的步骤，将自己写的类扫到容器中
```

###二、spring boot使用外部servlet容器运行原理:
####1>servlet3.0之后增加了如下的规则：
###1）服务器启动（web应用启动）时会找在当前web应用里面每一个jar包里面找到ServletContainerInitializer实例：
###2）ServletContainerInitializer的实现放在jar包的META-INF/services文件夹下，有一个名为javax.servlet.ServletContainerInitializer的文件，内容就是ServletContainerInitializer的实现类的全类名使用@HandlesTypes，在应用启动的时候加载我们感兴趣的类；
####2>原理:
#####1）首先启动tomcat容器
#####2）在Spring的web模块中有这个文件：
```java
org\springframework\spring-web\4.3.14.RELEASE\spring-web-4.3.14.RELEASE.jar!\METAINF\services\javax.servlet.ServletContainerInitializer：
其中这个文件中有org.springframework.web.SpringServletContainerInitializer这个类
```
#####3）SpringServletContainerInitializer将@HandlesTypes(WebApplicationInitializer.class)标注的所有这个类型的类都传入到onStartup方法的Set>；为这些WebApplicationInitializer类型的类创建实例；而WebApplicationInitializer的实现类如下图所示：
![](/img/2.png)
#####4）每一个WebApplicationInitializer都调用自己的onStartup；等同于创建一个SpringBootServletInitializer类的对象，并执行onStartup方法
#####5）SpringBootServletInitializer实例执行onStartup的时候会调用createRootApplicationContext方法创建容器
```java
protected WebApplicationContext createRootApplicationContext(
			ServletContext servletContext) {
    	//1、创建SpringApplicationBuilder
		SpringApplicationBuilder builder = createSpringApplicationBuilder();
		builder.main(getClass());
		ApplicationContext parent = getExistingRootWebApplicationContext(servletContext);
		if (parent != null) {
			this.logger.info("Root context already created (using as parent).");
			servletContext.setAttribute(
					WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, null);
			builder.initializers(new ParentContextApplicationContextInitializer(parent));
		}
		builder.initializers(
				new ServletContextApplicationContextInitializer(servletContext));
		builder.contextClass(AnnotationConfigEmbeddedWebApplicationContext.class);
    	//2、调用configure方法，子类重写了这个方法（ServletInitializer类），将SpringBoot的主程序类传了进来
		builder = configure(builder);
		builder.listeners(new WebEnvironmentPropertySourceInitializer(servletContext));
    	//3、使用builder创建一个Spring应用
		SpringApplication application = builder.build();
		if (application.getSources().isEmpty() && AnnotationUtils
				.findAnnotation(getClass(), Configuration.class) != null) {
			application.getSources().add(getClass());
		}
		Assert.state(!application.getSources().isEmpty(),
				"No SpringApplication sources have been defined. Either override the "
						+ "configure method or add an @Configuration annotation");
		// Ensure error pages are registered
		if (this.registerErrorPageFilter) {
			application.getSources().add(ErrorPageFilterConfiguration.class);
		}
    	//4、启动Spring应用
		return run(application);
	}
```

#####6）上段代码最后一行调用run()方法，在此方法中再次调用了SpringApplication的run()方法，后续流程，除了不去创建嵌入式tomcat容器外，会创建ioc容器，具体步骤参考上面嵌入式servlet容器运行原理的(2)-(4)，创建过程中不会在调用(4)的onrefresh()方法了。

```wiki
总结:先启动Servlet容器，再启动SpringBoot应用
```

