## synchronized关键字
1. 作用在变量上
用synchronized修饰成员变量,就直接限制了这个对象实例,synchronized(成员变量)就等于synchronized(this)
2. 作用在方法上
线程级别的
当一个线程访问实例对象中的synchronized同步代码块或同步方法时，该线程便获取了该实例的对象级别锁，其他线程这时如果要访问synchronized同步代码块或同步方法，便需要阻塞等待，直到前面的线程从同步代码块或方法中退出，释放掉了该对象级别锁。
3. 作用在类上（类锁）
全局的
类级别锁被特定类的所有示例共享，它用于控制对static成员变量以及static方法的并发访问。
synchronized(XX.class)实现了全局锁的效果
static synchronized方法也相当于锁住了代码段
4. 作用在同步代码块上
使用synchronized（obj）同步语句块，可以获取指定对象上的对象级别锁

## static关键字
1. 作用在变量上
2. 作用在方法上
3. 作用在类上

## final关键字
1. 作用在变量上
2. 作用在方法上
3. 作用在类上

## volitile
是一种稍弱的同步机制，在访问volatile变量时不会执行加锁操作，也就不会执行线程阻塞，因此volatilei变量是一种比synchronized关键字更轻量级的同步机制
