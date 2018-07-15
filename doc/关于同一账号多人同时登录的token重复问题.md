最近在项目上遇到了一个同一账号多终端(或者多人)同时登录导致的token重复问题。可以在浏览器相应地做一些防止表单重复提交的操作，比如登录按钮点击一次后变成不可点击的状态，等待服务器的响应之后再恢复成点击状态。不过这也并不能解决同一账号多终端登录的问题。

#### 项目中的伪代码如下：

```
User user = getUser(username,password);
if(null != user) {
    String token = findToken(user);
    if(null != token) {
	user.deleteToken(user);
        token = null;
    }
    //多个线程会同时进入这里
    if(null == token){
        user.insertNewToken(user);
    }    
}else {
    throw new Exception("用户不存在");
}
```

#### 想一想：为什么会出现token重复的问题？

因为同一账号多人(或者多终端）同时登录时都进入到了if(null == token)这个语句，所以就向数据库中插入了多个token，但是按道理，一个账号，只能对应一个token。

#### 那如何解决这个问题呢？

#### (1)我开始想到的是做一个同步，用synchronized同步语句块(目前的选择的方案)。

```
User user = getUser(username,password);
if(null != user) {
    //使用synchronized做同步
    synchronized(username.intern()){
        String token = findToken(user);
        if(null != token) {
	    user.deleteToken(user);
            token = null;
        }
        if(null == token){
            user.insertNewToken(user);
        }     
    }
}else {
    throw new Exception("用户不存在");
}
```
synchronized括号里的内容应该写什么呢？User实例(对象)？每个线程的User实例都是不一样的。username变量？试了不起作用。username并不是在编译期就可知的，而是在运行期从浏览器传递到后台。修改成username.intern()是可以成功的，intern()方法会在常量池中查找和创建字符串。同步字符串变量的intern()方法可能会导致性能问题，所以现在暂且先用着这个方案，看之后会出现什么问题。

##### (2)使用锁做同步

```
//定义一个排它重入锁
private ReentrantLock lock = new ReentrantLock();

User user = getUser(username,password);
if(null != user) {  
    try{
        //获取锁
        lock.lock();
        String token = findToken(user);
        if(null != token) {
	    user.deleteToken(user);
            token = null;
        }
        if(null == token){
            user.insertNewToken(user);          
        }     
    }catch(Exception e) {
        e.printStackTrace();
    }finally {
        //释放锁
        lock.unlock();
    }
}
```
注意：定义一个私有公共的ReentrantLock，获取锁是调用lock()方法，释放锁调用unlock()方法。如果对于每一个线程都需要加锁和释放锁，性能会比较低。

#### (3)使用数据库的乐观锁实现(测试发现这种方式不可取)。

在用户表上添加一个表示版本的字段。每次用户登录添加token时，都需要进行版本的比较，只有版本一致时，才进行添加token操作。当有两个线程进入到if(null == token)，第一个线程会修改版本号并插入token，而第二个线程则会因为版本号不一致，抛出异常。

这个方法不可取的原因是：没有办法控制两个线程不会同时进入if(version == user.getVersion()){}，所以这种方式还是会在登录的时候出现重复的token,也就是还是需要同步才能解决问题。


修改后的伪代码如下：

```
User user = getUser(username,password);
if(null != user) {
    String token = findToken(user);
    if(null != token) {
	user.deleteToken(user);
        token = null;
    }
    if(null == token){
        int version = getUserVersion(user.getId());    
        if(version == user.getVersion()) { 
             updateVersion(version+1,User);
             user.insertNewToken(user);
         }else {
             throw new Exception("同一账号不能同时登录");
         }
    }
}else {
    throw new Exception("用户不存在");
}
```

### 延伸

1.这个一个账号多人（多终端）登录产生重复token是一个线程安全问题。

(1).什么是线程安全。

如果多个线程访问同一个对象时，不需要考虑运行时的交替执行，不需要考虑同步，也不需要调用方的协调，每次调用这个对象都可以得到正确的结果，就称这个对象是线程安全的。

(2).如何保证线程安全。

2.synchronized和ReentrantLock有什么区别？

(1)synchronized是Java内存模型定义的一系列原语(包括volatile,final等)之一。ReentrantLock是Jdk API类。

(2)synchronized可以作用在方法上和代码块上。ReentrantLock只能作用在代码块上。

(3)ReentrantLock可以进行更细粒度的操作，比如实现为公平锁还是非公平锁。公平锁指的是，当一个线程拥有某个锁后释放，想要再获得这个锁，需要进入这个锁的等待队列的尾部排队等待再次获取这个锁。非公平锁指的是，这个线程释放了拥有的锁之后，想再次获得，会比其他线程更容易获得这个相同的锁。非公平锁和公平锁相比，减少了线程上下文的切换，但同时也可能造成一些线程很长时间都没有获得锁。

3.悲观锁和乐观锁有什么区别？

(1)悲观锁是设定冲突一定会发生，所以进行写入更新操作时只能有单一线程进行操作。

(2)乐观锁是设定冲突不一定发生，进行更新时，先判断旧值是否发生改变，如果没有发生改变，就设置成新值，更新成功；如果发生改变，就不操作，更新失败。


#### 参考文章：

https://blog.csdn.net/u014653197/article/details/76177277

https://blog.csdn.net/antony9118/article/details/52664125
