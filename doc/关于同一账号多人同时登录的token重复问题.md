最近在项目上遇到了一个同一账号多终端(或者说多用户)同时登录导致的token重复问题。可以在浏览器相应地做一些防止表单重复提交的操作，比如登录按钮点击一次后变成不可点击的状态，等待服务器的响应之后再恢复成点击状态。不过这也并不能解决同一账号多终端登录的问题。

####项目中的伪代码如下：

```
User user = getUser(username,password);
if(null != user) {
    String token = findToken(user);
    if(null != token) {
	user.deleteToken(user);
        token = null;
    }
    if(null == token){
        user.insertNewToken(user);
    }    
}else {
    throw new Exception("用户不存在");
}
```

####想一想：
为什么会出现token重复的问题？因为同一账号多用户(或者多终端）同时登录时都进入到了if(null == token)这个语句，所以就向数据库中插入了多个token，但是按道理，一个账号，只能对应一个token。

####那如何解决这个问题呢？

(1)我开始想到的是做一个同步，用synchronized同步语句块(目前的选择的方案)。

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

(2)使用锁ReentrantLock做同步

```

//定义锁
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

(3)使用数据库的乐观锁实现(测试发现这种方式不可取)。

在用户表上添加一个表示版本的字段。每次用户登录添加token时，都需要进行版本的比较，只有版本一致时，才进行添加token操作。当有两个线程进入到if(null == token)，第一个线程会修改版本号并插入token，而第二个线程则会因为版本号不一致，抛出异常。

这个方法不可取的原因是：没有办法控制两个线程不会同时进入if(version == user.getVersion()){}，所以这种方式还是会在登录的时候出现重复的Token。


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



参考文章：

https://blog.csdn.net/u014653197/article/details/76177277

https://blog.csdn.net/antony9118/article/details/52664125
