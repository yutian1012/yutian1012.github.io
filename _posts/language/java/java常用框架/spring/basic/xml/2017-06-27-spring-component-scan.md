---
title: spring扫描通配符
tags: [spring]
---

参考：http://docs.spring.io/spring/docs/3.0.x/spring-framework-reference/html/beans.html#beans-annotation-config

### base-package扫描语法

```
<context:component-scan base-package="com.xxx"/>
```

实现类：ComponentScanBeanDefinitionParser

参考：https://my.oschina.net/HeliosFly/blog/203149

1）查看该类下的parse方法解析xml元素

标签的解析是由ComponentScanBeanDefinitionParser类解析的

```
public BeanDefinition parse(Element element, ParserContext parserContext) {
   //获取context:component-scan 配置的属性base-package的值
   String[] basePackages = StringUtils.tokenizeToStringArray(element.getAttribute(BASE_PACKAGE_ATTRIBUTE),
           ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);
   //创建扫描对应包下的class文件的对象
   ClassPathBeanDefinitionScanner scanner = configureScanner(parserContext, element);
   //扫描对应包下的class文件并有注解的Bean包装成BeanDefinition
   Set<beandefinitionholder> beanDefinitions = scanner.doScan(basePackages);
   registerComponents(parserContext.getReaderContext(), beanDefinitions, element);
   return null;
}
```

首先，获取context:component-scan 配置的属性base-package的值，然后放到数组。然后，创建扫描对应包下的class和jar文件的对象ClassPathBeanDefinitionScanner，由这个类来实现扫描包下的class和jar文件并把注解的Bean包装成BeanDefinition。最后，BeanDefinition注册到Bean工厂。

2）doScan方法完成包下类的扫描操作

扫描是由ComponentScanBeanDefinitionParser的doScan方法来实现的，返回beandefinitionholder的set集合

```
protected Set<beandefinitionholder> doScan(String... basePackages) {
    //新建队列来保存BeanDefinitionHolder
    Set<beandefinitionholder> beanDefinitions = new LinkedHashSet<beandefinitionholder>();
    //循环需要扫描的包
    for (String basePackage : basePackages) {
        //进行扫描注解并包装成BeanDefinition
        Set<beandefinition> candidates = findCandidateComponents(basePackage);
        for (BeanDefinition candidate : candidates) {
            ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
            candidate.setScope(scopeMetadata.getScopeName());

            //为扫描到的BeanDefinition生成一个beanName
            String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
            if (candidate instanceof AbstractBeanDefinition) {
                postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
            }

            if (candidate instanceof AnnotatedBeanDefinition) {
                AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
            }

            //由BeanDefinition包装成BeanDefinitionHolder
            if (checkCandidate(beanName, candidate)) {
                BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
                definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
                beanDefinitions.add(definitionHolder);
                //对BeanDefinition进行注册
                registerBeanDefinition(definitionHolder, this.registry);
            }
        }
    }
    return beanDefinitions;
}
```

3）findCandidateComponents方法获取base包下的所有类的BeanDefinition

进行扫描注解并包装成BeanDefinition是ComponentScanBeanDefinitionParser由父类ClassPathScanningCandidateComponentProvider的方法findCandidateComponents实现的。返回beandefinition的set集合

```
public Set<beandefinition> findCandidateComponents(String basePackage) {
    Set<beandefinition> candidates = new LinkedHashSet<beandefinition>();
    try {
       //base-package中的值替换为classpath*:cn/test/**/*.class
        String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
                resolveBasePackage(basePackage) + "/" + this.resourcePattern;

        //获取所以base-package下的资源，重点
        Resource[] resources = this.resourcePatternResolver.getResources(packageSearchPath);
        boolean traceEnabled = logger.isTraceEnabled();
        boolean debugEnabled = logger.isDebugEnabled();
        for (Resource resource : resources) {
            if (traceEnabled) {
                logger.trace("Scanning " + resource);
            }
            if (resource.isReadable()) {
                try {
                    MetadataReader metadataReader = this.metadataReaderFactory.getMetadataReader(resource);
                     //对context:exclude-filter进行过滤
                     //判断是否候选的component
                    if (isCandidateComponent(metadataReader)) {
                       //包装BeanDefinition
                        ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
                        sbd.setResource(resource);
                        sbd.setSource(resource);
                        if (isCandidateComponent(sbd)) {
                            if (debugEnabled) {
                                logger.debug("Identified candidate component class: " + resource);
                            }
                            candidates.add(sbd);
                        }
                        else {
                            if (debugEnabled) {
                                logger.debug("Ignored because not a concrete top-level class: " + resource);
                            }
                        }
                    }
                    else {
                        if (traceEnabled) {
                            logger.trace("Ignored because not matching any filter: " + resource);
                        }
                    }
                }
                catch (Throwable ex) {
                    throw new BeanDefinitionStoreException(
                            "Failed to read candidate component class: " + resource, ex);
                }
            }
            else {
                if (traceEnabled) {
                    logger.trace("Ignored because not readable: " + resource);
                }
            }
        }
    }
    catch (IOException ex) {
        throw new BeanDefinitionStoreException("I/O failure during classpath scanning", ex);
    }
      return candidates;
}
```

首先，根据context:component-scan中属性的base-package="cn.test"配置转换为classpath*:cn/test/**/*.class，并扫描对应下的class和jar文件并获取类对应的路径，返回Resources。然后，根据指定的不扫描包，指定的扫描包配置进行过滤不包含的包对应下的class和jar。最后，封装成BeanDefinition放到队列里。

4）getResource从指定路径下获取Resource

根据packageSearchPath获取包对应下的class路径，是通过PathMatchingResourcePatternResolver类，findAllClassPathResources(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()));获取配置包下的class路径并封装成Resource，实现也是getClassLoader().getResources(path);实现的。返回Resource数组。

```
public Resource[] getResources(String locationPattern) throws IOException {
    Assert.notNull(locationPattern, "Location pattern must not be null");
    //判断是否已classpath开头
    if (locationPattern.startsWith(CLASSPATH_ALL_URL_PREFIX)) {
        // a class path resource (multiple resources for same name possible)
        if (getPathMatcher().isPattern(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()))) {
            // a class path resource pattern
            return findPathMatchingResources(locationPattern);
        }
        else {
            // all class path resources with the given name
            return findAllClassPathResources(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()));
        }
    }
    else {
        // Only look for a pattern after a prefix here
        // (to not get fooled by a pattern symbol in a strange prefix).
        int prefixEnd = locationPattern.indexOf(":") + 1;
        //如路径filesystem:xxxx/xxx/xx
        if (getPathMatcher().isPattern(locationPattern.substring(prefixEnd))) {
            // a file pattern
            return findPathMatchingResources(locationPattern);
        }
        else {
            // a single resource with the given name
            return new Resource[] {getResourceLoader().getResource(locationPattern)};
        }
    }
}
 
protected Resource[] findAllClassPathResources(String location) throws IOException {
    String path = location;
    if (path.startsWith("/")) {
        path = path.substring(1);
    }
    Enumeration<url> resourceUrls = getClassLoader().getResources(path);
    Set<resource> result = new LinkedHashSet<resource>(16);
    while (resourceUrls.hasMoreElements()) {
        URL url = resourceUrls.nextElement();
        result.add(convertClassLoaderURL(url));
    }
    return result.toArray(new Resource[result.size()]);
}
```