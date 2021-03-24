## SpringBoot

## SpringClound

### 常用类

#### Fegin




## 常见问题:
1. BeanFactory 和 ApplicationContext 的不同点



2. @PostConstruct 的使用
   如果想在生成对象时完成某些初始化操作，而偏偏这些初始化操作又依赖于依赖注入，那么久无法在构造函数中实现。为此，可以使用@PostConstruct注解一个方法来完成初始化，@PostConstruct注解的方法将会在依赖注入完成后被自动调用。

   调用的顺序Constructor > @PostConstruct > InitializingBean > init-method

3. @ConfigurationProperties 
   https://blog.csdn.net/yusimiao/article/details/97622666