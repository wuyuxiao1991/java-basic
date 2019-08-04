## 口碑面试
### 卡夫卡如何实现削峰
消息队列中间通过一个队列在一端承接瞬时的流量洪峰，在另一端平滑地将消息推送出去。通过pull的方式？？？？
kalfka消息队列的topic（一种请求）逻辑上是一个queue，但是其实分了不同的partition存储在不同的broker上
监控日志打点其实也是通过kafka消息队列抛下来的
### 点对点是机器对机器么
如果是一个服务器对应一个tomcat机器，那么就是，rpc通信的一种机制，是应用中引入一个rpc的包：作用：   
具体找到这个包
+ 有负载均衡算法，包括随机、轮训、ip hash等,可以用f5和nginx实现（令桶牌）
+ 提供和zookeeper通信的方式，zookeeper只提供可以调用的点
https://blog.csdn.net/gx11251143/article/details/88574133 
https://www.cnblogs.com/shamo89/p/10029519.html
服务质量框架服务熔断一般是由于下游的故障，如果调用端继续尝试只会加重超时，这个时候就把服务的所有请求都反悔mock。降级就是为了保证核心业务正常运行而将其他业务的优先级降低的一种机制，比如秒杀活动里，可能秒杀找个接口的请求很多，担心把服务器拖垮，就直接返回无商品了
名单流量大了怎么办：扩容、控制流量、使用服务治理框架的流量控制，服务降级，熔断等功能

### 压测为什么不通过mq
应该压mq，为什么不直接压全量日志之前的流根据返回的异常重新尝卡夫卡为何做到削峰的，系统为何设计成这个样子，为什么不直接抛mq，记录日志的这个过程需要拆解开？？？？？

### 卡夫卡多次下抛浪费了计算能力，怎么办？
1.consumer会自动提交自己的消费位移（也可以手动提交），也就是说消费到了哪一条。
2.刚才你说的场景是已经被消费了，也就是说consumer消费成功了，那么consumer自己会记录这个位移。
3.你说的服务器崩了，说的应该是这个consumer崩溃了，那么会把该consumer负责的任务交给另外一个consumer来继续处理  

首先在高并发的时候可以用redis缓存去控，防止重复消费，然后如果是没有返回或者失败，卡夫卡自己提供了重试机制，可以靠数据库去限
可以用乐观锁、sessionid，redis？？

synchronized锁和ReentrantLock、redis都是悲观的锁，其实悲观锁和乐观锁的效果可能是一样，但是
像synchronize这种悲观锁在底层是通过jvm的字节码等实现的，影响效率性能的有开销的，并且会让线程持续等待，不建议使用
  ``` 
  public class Atom_Test {
    private static AtomicInteger i = new AtomicInteger(0);
    public static void main(String[] args) {
        for (int j = 0; j < 100; j++) {
            new Thread() {
                public void run() {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                    }
 
                    i.getAndIncrement();
                }
            }.start();
        }
 
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
        }
 
        System.out.println(i);
    }
} 
```
通过CAS来更新共享变量，如果CAS更新失败则证明其他线程修改了这个共享变量，自己循环重试直到更新成功就可以了。因为CAS操作不会挂起线程因此减少了线程挂起和唤醒的开销，在高并发情况下这个节省是非常可观的。循环重试虽然不会挂起线程但是会消耗CPU，因为线程需要一直循环重试，这也是CAS乐观锁的一个缺点。
CAS乐观锁还有另一个缺点就是无法解决“ABA”问题。
原理，getAndIncrement是一个原子操作，IncreementAndGet是返回的加之后的值
```public final int getAndIncrement() {
    for (;;) {
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next))  ／／CAS是CPU的原语，是多条指令构成，一定是连续执行的，是一条CPU原子指令
            return current;
    }
}

public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```


### 高并发下的幂等解决方案：
包括了创建订单、更新订单、删除订单等只能发生一次实质性的操作

### 如何在应用的阶段发现gc问题
设置了报警，如果gc次数过多以及内存占用过大可能就发生了内存泄露，dump 一下内存的快照
开源的有一个好像叫MAT工具的，Histogram查询，可以看到哪个类的实例
内存泄漏一般是代码的问题，缩小作用域，然后赋null，主要是发生在写代码的阶段，修改参数
### 缓存存什么
问我redis的缓存，就应该引出redis的锁的问题
重要的是把自己能说的知道的都说出来，而不是其他的
分裂和合并不要混淆了概念

### synchronize的偏向锁、轻量级锁、重量级锁原理
对于synchronized这个关键字，可能之前大家有听过，他是一个重量级锁，开销很大，建议大家少用，但到了jdk1.6之后，该关键字被进行了很多的优化，已经不像以前那样不给力了，建议大家多使用。JDK1.6以后，为了减少获得锁和释放锁所带来的性能消耗，提高性能，引入了“轻量级锁”和“偏向锁”。    
Java偏向锁(Biased Locking)是Java6引入的一项多线程优化。   
https://www.cnblogs.com/kancy/p/10482756.html    
https://blog.csdn.net/kirito_j/article/details/79201213     
偏向锁是只有一个线程的时候，当竞争时升级成为轻量级锁，利用自旋来减少CPU对于线程调度的切换，不会足赛线程而是稍微等待一会，如果并发高就升级为重量级的锁，防止CPU一直空转     
AQS，p7必问    
syhcronize的方法被锁住了，那么这个实例或者这个类的所有sychronixze的方法都只能等待该线程执行完再获得锁   
### 娇娇头条面试题：  
我面试的时候遇到的问题是假如让你做微信朋友圈，9亿用户，有查看别人朋友圈，查看所有好友最近朋友圈，评论，点赞，转发的功能，怎么分库分表，怎么设计，看看娇娇发的
加大并发极限值，压测！！！把qps和接口列出来，一个名池请求会导致5个同步下游请求？？？
把慢的接口整理出来，并看分布，有什么办法  
Xms，Xmx都是堆内存的一个设置，一般会设成一样的，避免gc导致内存空间的频繁变化    

### tps  qps用分库分表，怎么分，和用热数据，冷数据的区别

### 卡夫卡能否实现顺序消费
可以从sysemlog里看出来是什内存溢出，应用起不起来可能是classloasld，如果运行时突然拿到一个很大的内存对象也会，dump能看到    
consumer还是zookeeper维护位移？？？


### cglib的动态代理实现aop
反射是基于字节码的，所以效率很慢，解释性
cglib实现代理是通过创建子类，子类实现一个接口，通过反射对子类实例化
看jdk实现，和cglib实现代理实现aop的区别
静态代理  动态代理  mybatis和aop的代理的区别
### foreach和for循环适用的场景不同


## ebay电话面
### 为何要用这种线程池
### 线程数目怎么设置
### broker做了哪些事情
### innodb的bean log和那个什么log的区别
### 消息队列producer需要得到返回么
消息队列的机制，卡夫卡是开源，我们这边的高阶卡夫卡是把卡夫卡的封装了一层，在produce和consume的时候都封装了重试，     
produce的时候，request.required.acks=1是，表示一条消息发送给了某个broker，然后进入了某个partition，只要是主partition的成功接收并返回了，就表示发送成功，不然会抛异常，如果是request.required.acks=all，那么min.insync.replicas指定的副本如果达不到这个最小同步的（follower从partition去同步消息），=2标识至少有一个副本也同步到了，就抛异常，默认就是1，是卡夫卡的内置的配置，高阶的那些参数是需要客户端自己配置的的。这种参数都是配置在xml中，高阶卡夫卡做的工作是，如果（网络问题等）抛异常失败，那么会应用在本地记录临时文件不断尝试，在消费时，通过指定时间间隔（可以很多次）
一般来说，broker三个，那么parition的副本数是2，就可以知道（1,2）两台上有partition1，一个leader，一个follower，leader挂了foller变为leader
每一个应用是一个消费者组，所以一条消息只能够被一个consumer消费     
replica是包括leader的     
producer只会给partition的ldeader发送消息，consumer也只会从leader初拉去消息，其他额partition都是同步而已    
一般partition的个数不能多于一个group中服务器的个数，不然有的consumer收不到消息   

### zookeeper的作用:
+ broker、消费者等的注册和管理
+ partition的负载均衡，比如一个topic有三个partition，主的分布在不同的broker上，一个生产者会和多个broker链接
+ offset位移信息一般记录成一个内部topic，不再用zookeeper维护，因为zookeeper不擅长大量读写操作，这个内部消息维护的是groupid和topic的partition的关系

## sychronize的锁是公平的么，和reentranlock的区别
在线程不是那么多的情况下，前者性能较好，是基于jvm实现的，后者是一个接口
前者不可中断，会一直等待，可能出现死锁，前者是非公平锁（释放锁时所有线程共同竞争），后者是公平的（会释放的时候由之前排队的第一个线程来抢到cpu执行，后者是j.u.c的一个类（和Atomic一样），前者能修饰方法和代码块，后者只能写在代码块里

## mybatis是怎么实现的

## 头条（娇娇）
### java
HashMap原理    
ArrayList，然后每个元素是一个LinskedList，每个元素是一个Entry，由key、value、next<Entry>组成    
散列表的形式，既利用了数组随机寻址方便的便利，也利用了链表插入方便的数据结构（为了解决hash冲突），一开始设定了长度，就确定了每个key hash之后的筒的位置，如果resize，会导致所有元素位置调整重新计算性能损耗大    
利用key的hashCode重新hash计算出当前对象的元素在数组中的下标，如果hash算出来位置有entry，则一一比较，没有相同的放在头部，有相同的，就替换掉value值，hash冲突应该尽可能小，一般负载因子0.75，是为了不要让hashMap存的太满，导致查询变慢很多，尽量散的hash算法，可以使得元素寻址迅速，类似动态数组    

### 集合有哪些
### 什么场景用多线程？线程数怎么设置？为什么？
 
### 网络
TCP三次握手？为什么要第三次？    
HTTPS实现    
HTTP session cookie？禁用cookie，session能用吗？localstorage    

### 数据库
ACID   
SQL注入   
mysql引擎？对比？索引实现？   
sql执行慢原因排查？什么时候索引失效？   
B+树、B树对比，B+树优点在哪儿？    
为什么mysql系统表、临时表用myisam?    

### 消息系统
RabbitMQ如何保证消息有序？    

### redis
主从模式？   
主从模式高延迟下如何同步？    
redis存储策略   
 
### os
linux文件系统iNode?    
linux文件权限？权限信息存哪儿？     

### 综合：
HTTP请求慢原因排查？     
设计微信朋友圈，可以建立好友关系，发布文章，可以评论、点赞、回复，根据需求设计数据库表、缓存，设计分表方案及缓存方案    

### 编程题
sql: 给成绩表，查出所有成绩均>80的学生名字    
翻转单词顺序列    
最长不重复子串    

## 美团电话面
项目经历+java基础+数据库基础+算法
对微服务的理解，微服务间的通信是什么     
这些服务的集中化管理已经是最少的，它们可以用不同的编程语言编写，并使用不同的数据存储技术。    
### 为什么要使用微服务？    
+ 变更周期是连在一起的 - 对应用程序的一小部分进行更改，需要重建和部署整个程序，减少构建时间，并
+ 时间长了很难保证模块的分离
+ 每个服务根据各自的特点和数据类型选择适合的存储方式和服务架构
+ 不在一个进程中，改为http方式或者restful的轻量级的方式去实现微服务间通信 进程间通信 IPC，可以是异步的，可以是一对多的，等  
微服务有清晰的模块边界，不同的语言，甚至不同的团队管理，微服务最好实现基础服务组件化    
一般来说，最好将基础服务，支持服务，核心业务区分开，当某一个服务升级后，如果调用方是不同的服务方，那么可以通过提供默认值或者url中嵌入版本号等方式避免调用失败，注意兼容性    
Docker可以很好滴用来构建微服务，有不同的容器来承载不同的服务，并在之间实现通信    
我们的应用拆分之后是微服务么？有什么区别？？？？？

### 项目经历：
+ 你主要担任的指责，你负责的部分是什么？
+ 简单介绍系统设计的主要考虑的点是哪些，系统的体量大概有多大

### java基础：
### static修饰一个静态方法和一个普通方法的区别
https://www.cnblogs.com/showstone/p/4438539.html
用一个synchronized修饰一个static方法时，是类锁，对于所有对象都能够锁住
用synchronized修饰一个非static方法时是对象锁，当两个线程是new了不同的对象的话，是不会锁住该方法的，但是一般
同一对象时的情况下，是可以锁住的

### sleep和wait的区别，cpu是在干嘛sleep时，sleep应用的场景
sleep()方法是Thread类的静态方法，
Thread.sleep只会让出CPU，不会导致锁行为的改变，经常用这个来避免假死，避免一个线程或长时间占用cpu资源，sleep0经常用于激发一次竞争。
 Object.wait不仅让出CPU，还会释放已经占有的同步资源锁
sleep()方法可以在任何地方使用
 wait()方法只能在synchronized方法或synchronized块中使用
Thread.sleep(0)是干什么用的呢？它的作用就是：强制操作系统触发一次CPU计算优先级并分配时间片的动作，避免了某一线程长时间占用CPU资源
+ 为什么wait()必须在同步（Synchronized）方法/代码块中调用？
答：调用wait()就是释放锁，释放锁的前提是必须要先获得锁，先获得锁才能释放锁。
+ 为什么notify(),notifyAll()必须在同步（Synchronized）方法/代码块中调用？
答：notify(),notifyAll()是将锁交给含有wait()方法的线程，让其继续执行下去，如果自身没有锁，怎么叫把锁交给其他线程呢；（本质是让处于入口队列的线程竞争锁）
wait()是将线程唤醒，如果用的同一把锁，就进入线程等待池，等待执行


### 单核cpu为何要使用多线程呢
并不是所有时候都要用多线程，反而可能会增加线程切换的开销，当计算密集型的时候，就不建议单核的cpu去处理多线程
有时候线程数比cpu数目还多，对于io密集型会好一点，因为cpu等待时间可以利用起来
一个cpu一次只能处理一个线程
https://www.cnblogs.com/caihuafeng/p/5438753.html

### volatile和synchronize的区别
volatile相当于synchronized的弱实现,但是volatile并不能保证线程安全的，也就是说volatile字段的操作不是原子性的，volatile变量只能保证可见性（一个线程修改后其它线程能够理解看到此变化后的结果），声明为volatile的变量在工作时会每次取这个变量的值都需要从主存中取,而不是用自己线程工作内存中的缓存，没个线程改了之后会通知主存，是内存上的同步，volatile本质是在告诉jvm当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取；
static和volatile的区别：
https://www.cnblogs.com/cvbaka/p/4764503.html
static变量在不同线程中修改的static变量，是会在工作内存中copy一份，不会同步到全局的，所以线程不安全

   synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
+ volatile仅能使用在变量级别；
   synchronized则可以使用在变量、方法、和类级别的

+ volatile仅能实现变量的修改可见性，并不能保证原子性；
   synchronized则可以保证变量的修改可见性和原子性

+ volatile不会造成线程的阻塞；
   synchronized可能会造成线程的阻塞。

+ volatile标记的变量不会被编译器优化；
   synchronized标记的变量可以被编译器优化

###  为何要用多线程，因为一个cpu密集型会有等待时间，cpu虽然一次只能处理一个任务，但是多线程处理比单线程串行处理快的原因
### 重载是什么
###  多线程的几个参数，参数设成某个默认值是怎么评估的
### 新生代中复制算法的具体原理是什么，如何判断一个对象没有引用，互相引用算不算引用？引用计数，gc root，可达性分析，
局部变量的互相引用
### kalfka消息队列的消息是安全的么，如果挂了的话会怎么办
### String,StringBuilder,StringBuffer的区别
StringBuffer线程安全，可修改，把所有修改数据的方法都加上了synchronized。但是保证了线程安全是需要性能的代价的。    
StringBuilder加出来的数会小于1000，因为会被覆盖掉    

### 你们的项目的设计难点是哪些？
数据量大，所以做了如下操作
+ 消息队列
+ 分库分表
+ sql server新特性 openjson为了兼容不同的业务类型，类似nosql部分更新


### 数据库基础：
join的几种，区别，三个表相连时

### 算法题：
两个队列如何找到相同的元素    
求两个队列的共同序列     
1，3，5，6，7，8，10    
2，4，7，9，10，11    
两个指针，2比1大，则i指针后移，找到==2的，或者是比2大的，就j往后移，找到==3的，或者比3大的，依次往复   
```
public  ArrayList<Integer> findSameList(List<Integer> arr1,List<Integer> arr2){
int i=0;
int j=0;
List<Integer> resultIndex=new ArrayList<Intger>();
while(i==arr1.length)||j==arr2.lenth){
List<Integer>  indexList=findFirstSameNum(arr1,arr2,i,j));
i=indexList[0]+1;
j=indexList[1]+1;
resultIndex.add(i);
}
List<Integer> result=new ArrayList<Integer>();
for(int i in resultIndex){
  	result.add(arr1[i]);
}
return result;
}
7，8，10
7，9，10，11
 public List<Integer> findSameIndexList(List<Integer> arr1,List<integer> arr2, i, j){
       List<Integer> list=new ArrayList();
while(arr1[j]>arr2[i]){
i++;)
}
while(arr1[i]>arr[j]){j++}
if(arr[i]==arr[j]){
list.add(i);
list.add(j);
return list;}
else{
findSameIndexList( arr1,arr2,i,j);
}
}
```


总结：不要像死记硬背的
synchronized是Java中的关键字，是一种同步锁。它修饰的对象有以下几种： 
+ 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象； 
+ 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象； 
+ 修改一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象； 
+ 修改一个类，其作用的范围是synchronized后面括号括起来的部分，作用主的对象是这个类的所有对象。


## 华泰证券电话面
### Spring的bean的生命周期
bean的装载和实例化的流程：    
BeanFactory定义了IoC容器的最基本形式，并且提供了IoC容器所应该遵守的最基本的服务契约。在Spring的代码中，BeanFactory只是一个接口类，并没有给出容器的具体实现，具体类可以上前面的截图，如DefaultListableBeanFactory、XmlBeanFactory、ApplicationContext等都可以看成是容器的附加了某种功能的具体实现，也就是容器体系中的具体容器产品。    
https://www.cnblogs.com/baizhanshi/p/6755716.html    
+ Spring （boot）扫描@Component的组件（@Service组件是一样的，只是标识了不同的业务层级的逻辑），找到的bean相当于之前在xml中定义的bean
+ 通过反射构造一个类（知道类名），并newInstance（），遍历实例的属性Field，反射调用set方法，从beanPool中找到对应属性的实例并set进来，如果找不到递归地调用getbean实例化    
+ applicationContact这个工厂就是会将getbean这个实例化的过程放在初始化的时候，而不是AutoWire的时候，以免错误
https://blog.csdn.net/w_linux/article/details/80086950？？？？？？？如何理解Spring到底是不是一定是工厂创建的，如果是那么fatoryAare是干嘛？？
正常不要用反射，因为是解释性，效率不如直接调用，他体现了多态
反射是拿到字节码文件，Class.forName(Test.Class),然后对字节码方法或者属性的匹配，而从去进行一些操作，适合于不知道类的签名情况下

### 平时工作用到的哪些网络组件
 tomcat、F5，nginx，服务器,F5（不差钱的）是基于硬件的，但是ngnix是软负载均衡，也可以解决大部分问题，软件就可以实现，服务器还可以用来跑应用集群
F5的功能更为全面复杂，不止做了负载均衡,nginx解决浏览器跨域问题

### jdk1.8有没有了解
### jvm内存 深copy 浅copy
+  普通对象直接p2=p1，那么就是将引用地址复制，就是浅复制，任何类都继承了Object类，内有一个clone方法（protected），
+  重写clone方法来实现深度复制（不完全 ），但是必须同时implements clonable（一个标识接口）， 如果直接用super.clone 的方法，没有重写clone方法的逻辑（因为之前是protected），都会建一个内存里的对象的空间，但是字面量是copy了，引用变量存的是地址，所以指向的还是同一个地方，修改其中一个还是会修改到另一个。这样clone跟直接new出来的区别是，属性会不是初始值了
+ 如果要实现深克隆，那么需要递归到每一个层的引用变量，可见https://www.cnblogs.com/liqiangchn/p/9465186.html,重写

### Spring的aop的设计模式
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

### redis用到了哪些api
setnx，get，getset
如何实现getset的
会有个问题，就是多个线程getset导致修改掉了数据
比如说有一个请求线程锁了，然后还没处理完没释放调redisLock，那么可能超时了但是redis中依旧有这把锁，所以需要哥特天、判断超时

### http1 http2 的区别 



## paypal
### hashmap如果链表太长了咋办
linkedhashmap和hashmap比，他的取元素顺序和拿元素一样，底层数据结构一样
如果hashmap链表长度很长，就会自动转化为红黑树，就是一种二插搜索树，但是对于插入的代价高于链表，所以超过阈值才会变
### java有哪三种特性
### 进程间通信有哪些方法
### 快排的最差时间复杂度
挖坑机制，每次如果只排序好一个元素，就是冒泡的时间度
### 广度遍历和深度遍历的区别
###redis一般可以存储哪几种数据类型   
字符串一般存储在常量池里   
```a= “12“; b="12", a==b，true```
因为指向常量池，new出来的就是在栈中创建一个指向堆的对象，放在方法区不会被回收 s.intern()用于扩充常量池，总的来说，创建方式不一样，引用都在栈内，有的是编译期就已经创建好，存放在字符串常 量池中，而有的是运行时才被创建.使用new关键字，存放在堆中。   
基本数据类型都存在栈中，跟常量池一样，如果有了就直接指向 int i=9，int j=9，那么都指向栈的9的位置，
```final int x=9；``` 指向常量池的9

