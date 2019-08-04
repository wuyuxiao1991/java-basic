## Spring的bean的生命周期
bean的装载和实例化的流程：    
BeanFactory定义了IoC容器的最基本形式，并且提供了IoC容器所应该遵守的最基本的服务契约。在Spring的代码中，BeanFactory只是一个接口类，并没有给出容器的具体实现，具体类可以上前面的截图，如DefaultListableBeanFactory、XmlBeanFactory、ApplicationContext等都可以看成是容器的附加了某种功能的具体实现，也就是容器体系中的具体容器产品。    
https://www.cnblogs.com/baizhanshi/p/6755716.html    
+ Spring （boot）扫描@Component的组件（@Service组件是一样的，只是标识了不同的业务层级的逻辑），找到的bean相当于之前在xml中定义的bean
+ 通过反射构造一个类（知道类名），并newInstance（），遍历实例的属性Field，反射调用set方法，从beanPool中找到对应属性的实例并set进来，如果找不到递归地调用getbean实例化    
+ applicationContact这个工厂就是会将getbean这个实例化的过程放在初始化的时候，而不是AutoWire的时候，以免错误
https://blog.csdn.net/w_linux/article/details/80086950？？？？？？？如何理解Spring到底是不是一定是工厂创建的，如果是那么fatoryAare是干嘛？？
正常不要用反射，因为是解释性，效率不如直接调用，他体现了多态
反射是拿到字节码文件，Class.forName(Test.Class),然后对字节码方法或者属性的匹配，而从去进行一些操作，适合于不知道类的签名情况下

## 平时工作用到的哪些网络组件
 tomcat、F5，nginx，服务器,F5（不差钱的）是基于硬件的，但是ngnix是软负载均衡，也可以解决大部分问题，软件就可以实现，服务器还可以用来跑应用集群
F5的功能更为全面复杂，不止做了负载均衡,nginx解决浏览器跨域问题

## jdk1.8有没有了解
## jvm内存 深copy 浅copy
+  普通对象直接p2=p1，那么就是将引用地址复制，就是浅复制，任何类都继承了Object类，内有一个clone方法（protected），
+  重写clone方法来实现深度复制（不完全 ），但是必须同时implements clonable（一个标识接口）， 如果直接用super.clone 的方法，没有重写clone方法的逻辑（因为之前是protected），都会建一个内存里的对象的空间，但是字面量是copy了，引用变量存的是地址，所以指向的还是同一个地方，修改其中一个还是会修改到另一个。这样clone跟直接new出来的区别是，属性会不是初始值了
+ 如果要实现深克隆，那么需要递归到每一个层的引用变量，可见https://www.cnblogs.com/liqiangchn/p/9465186.html,重写

## Spring的aop的设计模式
ioc和aop都是为了解决代码耦合度过高的问题
https://www.cnblogs.com/lyb0103/p/7611826.html／？？Spring是通过利用反射创建bean的
aop是代理模式，用了动态代理，一个用户不想直接访问一个对象，而是想通过一个终结去访问这个对象，增添一些额外的操作
Spring用到了哪些设计模式：
+ 工厂模式，这个很明显，在各种BeanFactory以及ApplicationContext创建中都用到了；
+ 模版模式，这个也很明显，在各种BeanFactory以及ApplicationContext实现中也都用到了；
+ 代理模式，在Aop实现中用到了JDK的动态代理，aop和mybatis
+ 单例模式，这个比如在创建bean的时候。
+ 原型模式： Clone 来创建对象比直接new好直接操作二进制流     
....

为了实现aop切面日志，需要加哪些注解

## redis用到了哪些api
setnx，get，getset
如何实现getset的
会有个问题，就是多个线程getset导致修改掉了数据
比如说有一个请求线程锁了，然后还没处理完没释放调redisLock，那么可能超时了但是redis中依旧有这把锁，所以需要哥特天、判断超时

## http1 http2 的区别 
