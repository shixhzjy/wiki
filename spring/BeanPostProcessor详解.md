### BeanPostProcessor的简介
BeanPostProcessor，Bean的后置处理器。在Spring创建每一个Bean的过程中，都会执行`BeanPostProcessor`的实现类。  
Spring容器实例化bean过程中，执行bean的构造方法后，可以通过定义一个或多个`BeanPostProcessor`接口的实现类，在执行显示的初始化方法之前与后添加自定义逻辑。
>`显示的初始化方法方式有：`
>* 配置文件中bean标签添加init-method属性指定Java类中初始化方法。
>* @PostConstruct注解指定初始化方法。
>* Java类实现InitailztingBean接口。

构造方法、InitializingBean、BeanPostProcessor的执行顺序：  
`构造方法-->BeanPostProcessor-->InitializingBean-->bean中的初始化方法。`

### BeanPostProcessor API
````Java
public interface BeanPostProcessor {  
  
    /** 
     * Apply this BeanPostProcessor to the given new bean instance <i>before</i> any bean 
     * initialization callbacks (like InitializingBean's {@code afterPropertiesSet} 
     * or a custom init-method). The bean will already be populated with property values.    
     */  
    //实例化、依赖注入完毕，在调用bean显示的初始化方法之前完成一些定制的初始化任务  
    Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;  
  
      
    /** 
     * Apply this BeanPostProcessor to the given new bean instance <i>after</i> any bean 
     * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}   
     * or a custom init-method). The bean will already be populated with property values.       
     */  
    //实例化、依赖注入、显示的初始化方法完毕后执行  
    Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;  
  
}
````
由API可以看出:  
1：后置处理器的postProcessorBeforeInitailization方法是在bean实例化、依赖注入之后及显示的初始化方法之前调用。</br>
2：后置处理器的postProcessorAfterInitailization方法是在bean实例化、依赖注入、显示的初始化方法之后调用。</br>

`注意：BeanFactory和ApplicationContext两个容器对待bean的后置处理器稍微有些不同。`
>* ApplicationContext容器会自动检测Spring配置文件中哪些bean所对应的Java类实现了BeanPostProcessor接口，并自动把它们注册为后置处理器。在创建bean过程中调用它们，所以部署一个后置处理器跟普通的bean没有什么太大区别。</br>
>* BeanFactory容器注册bean后置处理器时必须通过代码显示注册，在IoC容器继承体系中ConfigurableBeanFactory接口中定义了注册方法。</br>
````Java
    /**  
     * Add a new BeanPostProcessor that will get applied to beans created  
     * by this factory. To be invoked during factory configuration.  
     * <p>Note: Post-processors submitted here will be applied in the order of  
     * registration; any ordering semantics expressed through implementing the  
     * {@link org.springframework.core.Ordered} interface will be ignored. Note  
     * that autodetected post-processors (e.g. as beans in an ApplicationContext)  
     * will always be applied after programmatically registered ones.  
     * @param beanPostProcessor the post-processor to register  
     */    
    void addBeanPostProcessor(BeanPostProcessor beanPostProcessor);
````

### Spring如何调用多个BeanPostProcessor实现类
我们可以在Spring配置文件中添加多个BeanPostProcessor接口的实现类。在默认情况下Spring容器会根据后置处理器的定义顺序来依次调用。
````xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- bean定义 -->    
    <bean id="narCodeService" class="com.test.service.impl.NarCodeServiceImpl"/>
    
    <!-- 多个后置处理器 -->
    <bean id="postProcessor" class="com.test.spring.PostProcessor"/>
    <bean id="postProcessorB" class="com.test.spring.PostProcessorB"/>
</beans>
````
````Java
public class PostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        
        System.out.println("PostProcessor后置处理器执行postProcessBeforeInitialization");
        
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        
        System.out.println("PostProcessor后置处理器执行postProcessAfterInitialization");
        
        return bean;
    }
}

     
public class PostProcessorB implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        
        System.out.println("PostProcessorB后置处理器执行postProcessBeforeInitialization");
        
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        
        System.out.println("PostProcessorB后置处理器执行postProcessAfterInitialization");
        
        return bean;
    }
}
````
测试结果：
````Java
PostProcessor后置处理器执行postProcessBeforeInitialization
PostProcessorB后置处理器执行postProcessBeforeInitialization
PostProcessor后置处理器执行postProcessAfterInitialization
PostProcessorB后置处理器执行postProcessAfterInitialization
````
**注意:`BeanPostProcessor`接口中`postProcessBeforeInitialization`、`postProcessAfterInitialization`两个方法都不能返回`null`。</br>
如果返回`null`，那么在后续执行显示的初始化方法时将报空指针异常，或者通过`getBean()`方法获取不到bena实例对象。因为后置处理器从Spring IoC容器中取出bean实例对象没有再次放回IoC容器中。**

**指定后置处理器调用顺序**
在Spring机制中可以指定后置处理器调用顺序，让BeanPostProcessor接口的实现类也实现`Ordered`接口的`getOrder()`方法。该方法返回一整数，默认值为`0`，优先级最高，值越大优先级越低。</br>
````Java
public class PostProcessor implements BeanPostProcessor, Ordered {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        
        System.out.println("PostProcessor后置处理器执行postProcessBeforeInitialization");
        
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        
        System.out.println("PostProcessor后置处理器执行postProcessAfterInitialization");
        
        return bean;
    }
    
    @Override
    public int getOrder() {
        return 1;
    }
}

     
public class PostProcessorB implements BeanPostProcessor, Ordered {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        
        System.out.println("PostProcessorB后置处理器执行postProcessBeforeInitialization");
        
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        
        System.out.println("PostProcessorB后置处理器执行postProcessAfterInitialization");
        
        return bean;
    }
    
    @Override
    public int getOrder() {
        return 0;
    }
}
````
测试结果：
````Java
PostProcessorB后置处理器执行postProcessBeforeInitialization
PostProcessor后置处理器执行postProcessBeforeInitialization
PostProcessorB后置处理器执行postProcessAfterInitialization
PostProcessor后置处理器执行postProcessAfterInitialization
````

[参考：https://www.cnblogs.com/sishang/p/6576665.html](https://www.cnblogs.com/sishang/p/6576665.html)</br>
BeanPostProcessor与自定义注解、动态代理结合的实战用法，详见：[https://www.jianshu.com/p/1417eefd2ab1](https://www.jianshu.com/p/1417eefd2ab1)
