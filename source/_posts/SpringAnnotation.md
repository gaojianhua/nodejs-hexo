
title: Spring注解原理探索
date: 2017-05-18 20:19:23
tags: Spring
categories: Spring
---

## 一. Java元注解释义

Question  
-注解在Java中如何起作用？
-Spring是如何识别注解？
-如何自定义注解为我所用？

<!-- more -->
```
Spring注解：
@Aotuwired @Required @Qualifier @Provider @Scope ...
Spring MVC 注解：
@Controller @Service @Repository @Component @RequestMapping @RequetBody @ResponseBody ...
```

java注解起源于JDK1.5
常见的注解有 @Override @Deprecated

1. 从@Override说起,源码如下

```
/*If a method is annotated with this annotation type 
 * compilers are required to generate an error message 
 * unless at least one of the following conditions hold:
 *
 * The method does override or implement a method declared in a
 * supertype.
 * The method has a signature that is override-equivalent to that of
 * any public method declared in {@linkplain Object}.
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {    // @interface 修饰注解类
}
```

元注解@Target,@Retention,@Documented,@Inherited 

* @Target 表示该注解用于什么地方，可能的 ElemenetType 参数包括： 
* ElemenetType.CONSTRUCTOR 构造器声明 
* ElemenetType.FIELD 域声明（包括 enum 实例） 
* ElemenetType.LOCAL_VARIABLE 局部变量声明 
* ElemenetType.METHOD 方法声明 
* ElemenetType.PACKAGE 包声明 
* ElemenetType.PARAMETER 参数声明 
* ElemenetType.TYPE 类，接口（包括注解类型）或enum声明 
* 
* @Retention 表示在什么级别保存该注解信息。可选的 RetentionPolicy 参数包括： 
* RetentionPolicy.SOURCE 保留在源码,注解将被编译器丢弃 
* RetentionPolicy.CLASS 注解在class文件中可用，但会被VM丢弃 
* RetentionPolicy.RUNTIME VM将在运行期也保留注释，因此可以通过反射机制读取注解的信息。 
* 
* @Documented 将此注解包含在 javadoc 中 
* 
* @Inherited 允许子类继承父类中的注解


** 1：如果定义一个注解需要被反射读取，则在定义这个注解的时候将添加@Retention(RetentionPolicy.RUNTIME) 元注解。**
** 2：如果想要自定义一个注解，就必须指定注解作用的位置。作用在 类，方法，属性域，构造函数等。 **
** 3：如果想要自定义注解，添加@interface 修饰类名 ** 

## 二. Java中如何自定义注解

SpringMVC中@RequestMapping 注解的源码示例

```
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)  // 注解一直保持到VM运行期，被反射读取。
@Documented
@Mapping    // SpringMVC定义的元注解，暂忽略此。
public @interface RequestMapping {
    String name() default "";
    
    @AliasFor("path")    // SpringMVC中定义的别名注解。
    String[] value() default {};

    @AliasFor("value")
    String[] path() default {};

    RequestMethod[] method() default {};

    String[] params() default {};

    String[] headers() default {};

    String[] consumes() default {};

    String[] produces() default {};
}
```
```
用法示例：
@RequestMapping(value="/user", methods=RequestMethod.GET)，
@RequestMapping可供选择的参数有：
name, value, path, method, params, headers, consumes, produces。

注解的每个参数对应着 @RequestMapping 类中的方法名。
每个参数指定一个默认值（default）。
```

** 1：注解定义格式：public @interface 注解名 {定义体} **
** 2：定义注解时，不得继承其他的注解或者接口。 **
** 3：注解类体中，每一个方法实际上声明了一个注解参数。方法名就是参数名，返回值类型就是参数类型。 **
** 4：注解参数支持的类型：8种基本类型（byte，short，int，long，float，double，char，boolean），String类型，Class类型，enum类型，Annotation类型，以上所有类型的数组。 **
** 5：访问修饰权限：public或者默认default。 **
** 6：注解元素必须要有默认值。在定义注解的默认值中指定，或者在使用注解的时候指定。非基本类型不能默认null。 **

当自定义注解类之后，便可以在类（ElementType.TYPE）、方法（ElementType.METHOD）上标注 @RequestMapping。

那怎样做才能让注解被Java程序所运行

## Java程序如何识别注解

Java 反射
java.lang.reflect 包，实现反射功能的工具类。
注解处理类库：java.lang.reflect.AnnotatedElement。

程序通过反射获取了某个类的AnnotatedElement对象之后, 程序就可以调用该对象如下的方法来访问Annotation的信息：


为了处理注解，** 注解处理器 ** 做3件事情：
- 读取配置文件中管理的bean
- 实例化bean
- 注解处理器获取实例bean中的注解并操作

```
前提假设：
我们已经自定义注解类，如 @RequestMapping 注解类,
并在合适的bean做出注解标注。
则编写自己的注解处理器。
```

```
public class ClassPathXMLApplicationContext {
    
    public ClassPathXMLApplicationContext(String configFileName) {
        // 读取配置文件中管理的bean
        readXMLConfigFile(configFileName);
        // 实例化bean
        instanceBean();
        // 向容器注册bean
        registerAnnotationBean();
    }

    // 读取配置文件中的bean
    private void readXMLConfigFile() {

    }

    // 实例化bean
    private void instanceBean() {

    }

    // 向容器注册bean
    private void registerAnnotationBean() {

    }
}
```

## Spring处理注解的源码分析
管理注解bean定义的两个容器：

- AnnotationConfigWebApplicationContext
- AnnotationConfigApplicationContext 的 Java doc 原语：
```
Standalone application context, accepting annotated classes as input.
... Alllows for registering classes one by one
using {@link #register(Class...)} as well as for classpath scanning using {@link #scan(String...)}.
```
这两个类是 直接依赖于注解 作为容器配置信息 的 IOC容器。

AnnotationConfigWebApplicationContext 是AnnotationConfigApplicationContext 的Web版本。
两者虽然实现方式略有差别，但处理注解的逻辑一样。
先扫描（scan(String...)）类路径（classpath），然后一一注册（register(Class<?>...)）。

简单点，以 AnnotationConfigApplicationContext 为例。

AnnotationConfigApplicationContext的源码：

```
package org.springframework.context.annotation;

public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry {
    // 读取器：读取注解的Bean，并注册到容器中。
    private final AnnotatedBeanDefinitionReader reader;
    // 扫描器：扫描类路径中注解的Bean，并注册到容器中。
    private final ClassPathBeanDefinitionScanner scanner;

    public AnnotationConfigApplicationContext() {
        // reader和scanner，功能类似，使用场景不同：annotatedClasses, basePakages.
        this.reader = new AnnotatedBeanDefinitionReader(this);
        this.scanner = new ClassPathBeanDefinitionScanner(this);
    }

    public AnnotationConfigApplicationContext(DefaultListableBeanFactory beanFactory) {
        super(beanFactory);
        this.reader = new AnnotatedBeanDefinitionReader(this);
        this.scanner = new ClassPathBeanDefinitionScanner(this);
    }

/*********************************
 *  1 区
 ***/
    public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
        this();
        register(annotatedClasses);
        refresh();
    }

    public AnnotationConfigApplicationContext(String... basePackages) {
        this();
        scan(basePackages);
        refresh();
    }
/*********************************/
    @Override
    public void setEnvironment(ConfigurableEnvironment environment) {
        super.setEnvironment(environment);
        this.reader.setEnvironment(environment);
        this.scanner.setEnvironment(environment);
    }

    public void setBeanNameGenerator(BeanNameGenerator beanNameGenerator) {
        this.reader.setBeanNameGenerator(beanNameGenerator);
        this.scanner.setBeanNameGenerator(beanNameGenerator);
        getBeanFactory().registerSingleton(
                AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR, beanNameGenerator);
    }

    public void setScopeMetadataResolver(ScopeMetadataResolver scopeMetadataResolver) {
        this.reader.setScopeMetadataResolver(scopeMetadataResolver);
        this.scanner.setScopeMetadataResolver(scopeMetadataResolver);
    }

/***************************
 *  2 区
 ***/
    public void register(Class<?>... annotatedClasses) {
        Assert.notEmpty(annotatedClasses, "At least one annotated class must be specified");
        this.reader.register(annotatedClasses);
    }

    public void scan(String... basePackages) {
        Assert.notEmpty(basePackages, "At least one base package must be specified");
        this.scanner.scan(basePackages);
    }
/***************************/
    @Override
    protected void prepareRefresh() {
        this.scanner.clearCache();
        super.prepareRefresh();
    }
}

```

请注意：
AnnotationCnofigApplicationContext 类的源码中的 “1 区”：
```
AnnotationConfigApplicationContext的基本功能在构造函数中完成。

实例化reader 和scanner；然后调用 register(Class<?>... annotatedClasses) 或 scan(String... basePackages) 方法；最后刷新。

而 register(Class<?>... annotatedClasses) 方法调用reader.register(Class<?>... annotatedClasses)；

scan(String... basePackages) 方法调用scanner.scan(String... basePackages)。

AnnotatedBeanDefinitionReader 和 ClassPathBeanDefinitionScanner 功能类似，二者取其一。
```
AnnotationConfigApplicationContext 类的源码中的 “2 区”：
```
如下两个方法完成了Spring的扫描注册功能：
reader.register(Class<?>... annotatedClasses);
scanner.scan(String... basePackages);
```
reader.register(Class<?>... annotatedClasses) 的源码：
```
public void register(Class<?>... annotatedClasses) {
        for (Class<?> annotatedClass : annotatedClasses) {
            registerBean(annotatedClass);
        }
    }

public void registerBean(Class<?> annotatedClass) {
        registerBean(annotatedClass, null, (Class<? extends Annotation>[]) null);
    }


// 注解功能实现区
@SuppressWarnings("unchecked")
public void registerBean(Class<?> annotatedClass, String name, Class<? extends Annotation>... qualifiers) {
    AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(annotatedClass);
    if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
        return;
    }

    ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
    abd.setScope(scopeMetadata.getScopeName());
    String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));
        
    AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
    if (qualifiers != null) {
        for (Class<? extends Annotation> qualifier : qualifiers) {
            if (Primary.class == qualifier) {
                abd.setPrimary(true);
            } else if (Lazy.class == qualifier) {
                abd.setLazyInit(true);
            } else {
                abd.addQualifier(new AutowireCandidateQualifier(qualifier));
            }
        }
    }

    BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
    definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
    BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
}

```

scanner.scan(String... basePackages) 源码：


```
public int scan(String... basePackages) {
        int beanCountAtScanStart = this.registry.getBeanDefinitionCount();

        doScan(basePackages);    // 备注：重点关注此处

        // Register annotation config processors, if necessary.
        if (this.includeAnnotationConfig) {
            AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
        }

        return (this.registry.getBeanDefinitionCount() - beanCountAtScanStart);
    }


// 注解功能实现区
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
    Assert.notEmpty(basePackages, "At least one base package must be specified");
    /**
     * BeanDefinitionHoder: Holder for a BeanDefinition with name and aliases.
     * BeanDefinition: A BeanDefinition describes a bean instance,   
     *   which has property values, constructor argument values, and 
     *   further information supplied by concrete implementations.
     */
    Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<BeanDefinitionHolder>();
    for (String basePackage : basePackages) {  // 多个包路径
        Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
        for (BeanDefinition candidate : candidates) {
            ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
            candidate.setScope(scopeMetadata.getScopeName());
            String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
            if (candidate instanceof AbstractBeanDefinition) {
                postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
            }
            if (candidate instanceof AnnotatedBeanDefinition) {
                AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
            }
            if (checkCandidate(beanName, candidate)) {
                BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
                definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
                beanDefinitions.add(definitionHolder);
                registerBeanDefinition(definitionHolder, this.registry);
            }
        }
    }
    return beanDefinitions;
}
```

