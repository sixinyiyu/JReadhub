Sring MVC 流程


![流程图](http://wx2.sinaimg.cn/large/9c349f47gy1fxeoar9o0ej20yh0gf3zq.jpg)

1. client发送请求到派发器(DispatcherServlet)
2. 派发器请求HandlerMapping查找 Handler 【可以根据xml配置、注解进行查找】
3. 处理器映射器HandlerMapping向前端控制器返回Handler
4. 派发器请求处理器适配器去执行Handler
5. 处理器适配器HandlerAdapter去执行Handler
6. 处理器适配器HandlerAdapter执行完成后，返回ModelAndView
7. 前端控制器请求视图解析器去进行视图解析【根据逻辑视图名解析成真正的视图(jsp)】    
8. 视图解析器向前端控制器返回View
9. 控制器进行视图渲染【视图渲染即. 模型数据(在ModelAndView对象中)render到响应中】
10. 前端控制器向用户响应结果

首先看看DispatcherServlet的类结构图
![DispatcherServlet类继承关系](spring-web-process/DispatcherServlet-class.png)
在没使用Spring MVC前做不同的控制器要写很多Servlet然后在web.xml里配置servlet,而使用了Spring MVC后我们在web.xml里会做如下配置
```java
<servlet>
        <servlet-name>springServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>dispatchOptionsRequest</param-name>
            <param-value>true</param-value>
        </init-param>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>
                classpath:config/servlet-config.xml
            </param-value>
        </init-param>
        <!-- 加载优先级 容器启动的时候就加载 -->
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>springServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```
这里可以很直观的看到配置了DispatcherServlet,通过类集成图发现DispatcherServlet也继承自HttpServlet说白了就是一个Servlet；那说到Servlet肯定就会说它的生命周期中的方法init(ServletConfig config)、service(ServletRequest req, ServletResponse res)、destory()方法；具体逻辑就是自定义的doGet、doPost、doPut等方法了。我们还是顺着周期来看吧。

### init方法
首先是init方法；HttpServlet的init(ServletConfig config)调用了init()方法而init()方法是一个空实现,那找到HttpServlet的直接子类org.springframework.web.servlet.HttpServletBean的重写
```java
public final void init() throws ServletException {
        if (logger.isDebugEnabled()) {
            logger.debug("Initializing servlet '" + getServletName() + "'");
        }

        // Set bean properties from init parameters.
        try {
            PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
            BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
            ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
            bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
            initBeanWrapper(bw);
            bw.setPropertyValues(pvs, true);
        }
        catch (BeansException ex) {
            logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
            throw ex;
        }

        // Let subclasses do whatever initialization they like.
        initServletBean();

        if (logger.isDebugEnabled()) {
            logger.debug("Servlet '" + getServletName() + "' configured successfully");
        }
    }
```
上面的注释很清楚了,设置一些初始化参数,而initBeanWrapper为给自定义包装Servlet预留的,主要的逻辑就在initServletBean方法了注释说让子类做任何它想要的初始化,那下面的重点就到了org.springframework.web.servlet.FrameworkServlet.initServletBean方法
```java
this.webApplicationContext = initWebApplicationContext(); //这才是G点
initFrameworkServlet();
```
initWebApplicationContext见名知意初始化Web应用上下文那我们就开始走进新世界的大门去吧。
```java
protected WebApplicationContext initWebApplicationContext() {
        //<TODO> 这里得到根上下文(这里是Spring的上下文)
        WebApplicationContext rootContext =
                WebApplicationContextUtils.getWebApplicationContext(getServletContext());
        WebApplicationContext wac = null;

        if (this.webApplicationContext != null) {
            // A context instance was injected at construction time -> use it
            wac = this.webApplicationContext;
            if (wac instanceof ConfigurableWebApplicationContext) {
                ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
                if (!cwac.isActive()) {
                    // The context has not yet been refreshed -> provide services such as
                    // setting the parent context, setting the application context id, etc
                    if (cwac.getParent() == null) {
                        // The context instance was injected without an explicit parent -> set
                        // the root application context (if any; may be null) as the parent
                        cwac.setParent(rootContext);
                    }
                    configureAndRefreshWebApplicationContext(cwac);
                }
            }
        }
        if (wac == null) {
            // No context instance was injected at construction time -> see if one
            // has been registered in the servlet context. If one exists, it is assumed
            // that the parent context (if any) has already been set and that the
            // user has performed any initialization such as setting the context id
            //<TODO>假定其父容器已经设置并已经执行了初始化,并尝试获取容器
            wac = findWebApplicationContext();
        }
        if (wac == null) {
            // No context instance is defined for this servlet -> create a local one
            //<TODO>实在找不到就开始依托父容器创建本地化上下文
            wac = createWebApplicationContext(rootContext);
        }

        if (!this.refreshEventReceived) {
            // Either the context is not a ConfigurableApplicationContext with refresh
            // support or the context injected at construction time had already been
            // refreshed -> trigger initial onRefresh manually here.
            onRefresh(wac);
        }

        if (this.publishContext) {
            // Publish the context as a servlet context attribute.
            String attrName = getServletContextAttributeName();
            getServletContext().setAttribute(attrName, wac);
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Published WebApplicationContext of servlet '" + getServletName() +
                        "' as ServletContext attribute with name [" + attrName + "]");
            }
        }

        return wac;
    }
```
跟一下createWebApplicationContext方法,发现接着要关注的是configureAndRefreshWebApplicationContext中的ConfigurableApplicationContext.refresh()熟悉的同学就会恍然大悟了,这不跟Spring容器初始化过程一样了
```java
public void refresh() throws BeansException, IllegalStateException {
        synchronized (this.startupShutdownMonitor) {
            // Prepare this context for refreshing.
            //<TODO> 初始化一些数据清空缓存等
            prepareRefresh();

            // Tell the subclass to refresh the internal bean factory.
            //<TOFO>加载bean定义 这里是加载spring-servlet.xml
            ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

            // Prepare the bean factory for use in this context.
            prepareBeanFactory(beanFactory);

            try {
                // Allows post-processing of the bean factory in context subclasses.
                //<TODO>beanFactory后置处理器
                postProcessBeanFactory(beanFactory);

                // Invoke factory processors registered as beans in the context.
                //<TODO>也是钩子函数 这里是加载properties配置
                invokeBeanFactoryPostProcessors(beanFactory);

                // Register bean processors that intercept bean creation.
                //<TODO>注册bean的后置处理器
                registerBeanPostProcessors(beanFactory);

                // Initialize message source for this context.
                //<TODO>类似I18配置
                initMessageSource();

                // Initialize event multicaster for this context.
                initApplicationEventMulticaster();

                // Initialize other special beans in specific context subclasses.
                onRefresh();

                // Check for listener beans and register them.
                registerListeners();

                // Instantiate all remaining (non-lazy-init) singletons.
                /**
                 *<TODO>开始初始化剩下的(用户自定义的)非懒加载单例Bean
                 *beanFactory.preInstantiateSingletons()
                 */
                finishBeanFactoryInitialization(beanFactory);

                // Last step: publish corresponding event.
                finishRefresh();
            }

            catch (BeansException ex) {
                if (logger.isWarnEnabled()) {
                    logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
                }

                // Destroy already created singletons to avoid dangling resources.
                destroyBeans();

                // Reset 'active' flag.
                cancelRefresh(ex);

                // Propagate exception to caller.
                throw ex;
            }

            finally {
                // Reset common introspection caches in Spring's core, since we
                // might not ever need metadata for singleton beans anymore...
                resetCommonCaches();
            }
        }
    }
```
在finishBeanFactoryInitialization方法中初始化org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping这个Bean的时候会将所有控制器初始化并得到所有RequestMapping后续就会执行DispatcherServlet的onRefresh方法
```java
/**
     * This implementation calls {@link #initStrategies}.
     */
    @Override
    protected void onRefresh(ApplicationContext context) {
        initStrategies(context);
    }

    /**
     * Initialize the strategy objects that this servlet uses.
     * <p>May be overridden in subclasses in order to initialize further strategy objects.
     */
    protected void initStrategies(ApplicationContext context) {
        //初始解析器
        initMultipartResolver(context);
        initLocaleResolver(context);
        initThemeResolver(context);
        //初始化映射处理器
        initHandlerMappings(context);
        //初始化处理适配器
        initHandlerAdapters(context);
        //异常解析器
        initHandlerExceptionResolvers(context);
        //视图转换器
        initRequestToViewNameTranslator(context);
        //视图解析器
        initViewResolvers(context);
        initFlashMapManager(context);
    }
```
这里的HandlerMapping、HandlerAdapter下面会用到而这些默认是在配置文件配置好的
![Handler-Properties](spring-web-process/handler-info-properties.png)
上面就说到差不多了,那还是综上所述一下
** 
DispatcherServlet是整个Spring MVC的核心。它负责接收HTTP请求组织协调Spring MVC的各个组成部分,Spring是父容器、Spring Mvc是子容器。其主要工作有以下三项：
  1. 截获符合特定格式的URL请求。
  2. 初始化DispatcherServlet上下文对应的WebApplicationContext，并将其与业务层、持久化层的WebApplicationContext建立关联。
  3. 初始化Spring MVC的各个组成组件，并装配到DispatcherServlet中 
父容器对子容器的bean是不可见的,子容器对父容器的Bean是可见的
**

### service

查看FrameworkServlet.service方法直接调用父类的service方法然后根据HttpMethod来决定执行doPost、doGet等逻辑,FramworkServlet复写了所有的do方法其实现就是调用processRequest---->doService--->doDispatch(核心逻辑)
![doPost](spring-web-process/do-post.png)
![process](spring-web-process/process.png)
![doService](spring-web-process/do-service.png)
![doDispatchr](spring-web-process/do-dispatch.gif)

doDispatch流程
![dispatch-process](spring-web-process/dispatch-proces.png)

接着还要说说HandlerAdapter.handle方法 其实这里面也是做了很多事的。
在handle方法中会创建ServletInvocableHandlerMethod对象进行
* 参数解析
* 返回值处理器注册
* 绑定参数转换工厂
* 参数映射
* 等等...

更具体的大家可以阅读下面的代码
```java
protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
            HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

        ServletWebRequest webRequest = new ServletWebRequest(request, response);
        try {
            WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
            ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);

            ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);
            //对RequestParam、PathVariable等注解参数处理
            invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
            invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
            invocableMethod.setDataBinderFactory(binderFactory);
            invocableMethod.setParameterNameDiscoverer(this.parameterNameDiscoverer);

            ModelAndViewContainer mavContainer = new ModelAndViewContainer();
            mavContainer.addAllAttributes(RequestContextUtils.getInputFlashMap(request));
            modelFactory.initModel(webRequest, mavContainer, invocableMethod);
            mavContainer.setIgnoreDefaultModelOnRedirect(this.ignoreDefaultModelOnRedirect);

            AsyncWebRequest asyncWebRequest = WebAsyncUtils.createAsyncWebRequest(request, response);
            asyncWebRequest.setTimeout(this.asyncRequestTimeout);

            WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
            asyncManager.setTaskExecutor(this.taskExecutor);
            asyncManager.setAsyncWebRequest(asyncWebRequest);
            asyncManager.registerCallableInterceptors(this.callableInterceptors);
            asyncManager.registerDeferredResultInterceptors(this.deferredResultInterceptors);

            if (asyncManager.hasConcurrentResult()) {
                Object result = asyncManager.getConcurrentResult();
                mavContainer = (ModelAndViewContainer) asyncManager.getConcurrentResultContext()[0];
                asyncManager.clearConcurrentResult();
                if (logger.isDebugEnabled()) {
                    logger.debug("Found concurrent result value [" + result + "]");
                }
                invocableMethod = invocableMethod.wrapConcurrentResult(result);
            }

            invocableMethod.invokeAndHandle(webRequest, mavContainer);
            if (asyncManager.isConcurrentHandlingStarted()) {
                return null;
            }

            return getModelAndView(mavContainer, modelFactory, webRequest);
        }
        finally {
            webRequest.requestCompleted();
        }
    }
```