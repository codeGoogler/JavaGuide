### Spring 

### Bean的生命周期

这里不会对Bean的生命周期进行详细的描述，只描述一下大概的过程。

Bean的生命周期指的就是:在Spring中， Bean是如何生成的?

### **被Spring管理的对象叫做Bean。Bean的生成步骤如下:**

![ ](https://upload-images.jianshu.io/upload_images/14371339-573b3809f377fc1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 1. Spring扫描class得到BeanDefinition
- 2.根据得到的BeanDefinition去生成bean
- 3.首先根据class推断构造方法
- 4.根据推断出来的构造方法，反射，得到一个对象(暂时叫做原始对象)
- 5.填充原始对象中的属性(依赖注入)
- 6.如果原始对象中的某个方法被AOP了，那么则需要根据原始对象生成一一个代理对象
- 7.把最终生成的代理对象放入单例池(源码中叫做singletonObjects) 中，下次getBean时就直接从单例池拿即可

可以看到，对于Spring中的Bean的生成过程，步骤还是很多的，并且不仅仅只有上面的7步，还有很多很多，比如
Aware回调、初始化等等，这里不详细讨论。

可以发现，在Spring中, 构造一个Bean, 包括了new这个步骤(第4步构造方法反射)。

得到一个原始对象后, Spring需要给对象中的属性进行依赖注入，那么这个注入过程是怎样的?

比如上文说的A类，A类中存在一个B类的b属性， 所以，当A类生成了-个原始对象之后，就会去给b属性去赋值,此
时就会根据b属性的类型和属性名去BeanFactory中去获取B类所对应的单例bean。如果此时BeanFactory中存 在B对
应的Bean,那么直接拿来赋值给b属性;如果此时BeanFactory中不存在B对应的Bean,则需要生成一个B对应的
Bean,然后赋值给b属性。



### mybaties的三级缓存策略



至此，总结一下三级缓存:

1. singletonObjects:缓存某个beanName对应的经过了完整生命周期的bean

2. earlySingletonObjects:缓存提前拿原始对象进行了AOP之后得到的代理对象,原始对象还没有进行属性注入
和后续的BeanPostProcessor等生命周期

3. singletonFactories:缓存的是一个ObjectFactory, 主要用来去生成原始对象进行了AOP之后得到的代理对
象，在每个Bean的生成过程中，都会提前暴露一个厂，这个工厂可能用到，也可能用不到，如果没有出现循环
依赖依赖本bean,那么这个工厂无用，本bean按照自己的生命周期执行，执行完后直接把本bean放入
singletonObjects中即可，如果出现f循环依赖依赖了本bean,则另外那个bean执行0bjectFactory提 交得到一
个AOP之后的代理对象(如果有AOP的话，如果无需AOP,则直接得到-一个原始对象)。

4.其实还要一个缓存， 就是earlyProxyReferences, 它用来记录某个原始对象是否进行过AOP了。

### 无aop的时候的流程图

![image.png](https://upload-images.jianshu.io/upload_images/14371339-121d0bccf377f65b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
