---
title: Multi Thread
date: 2016-06-12 14:38:01
tags:
---
## 线程安全对象周期管理
### 多线程下对象销毁可能存在竞态
- 不能由对象自身的mutex来保护
- 在即将析构一个对象时，从何而知此时此刻有别的线程在正在执行对象的成员函数
- 如何保证在执行成员函数期间，对象不会在另一个线程析构
- 在调用某个对象的成员函数之前，如果得知这个对象是否还活着？它的析构函数会不会碰巧执行到一半。
- 可以借助boost的`share_ptr`和`weak_ptr`完美解决。

### 对象创建
- 不要在构造函数中组册任何回调
- 不要在构造函数中将this指针传给跨线程对象
```c
//错误用法
class Foo: public Observer
{
public:
	Foo(Observer* s){
		s->register_(this);	//错误，非线程安全
	}
 	virtual void update();
};

//正确用法
class Foo: public Observer
{
public: 
	Foo();
	
	//另外定义一个函数，在构造之后执行回调函数的注册工作
	void observer(Observer* s){
		s->register_(this);
	}
}
```

### 销毁
```c
Foo::~Foo(){
	MutexLockGuard lock(mutex_);
	//Free internal state  ------------1
}
Foo::update(){
	MutexLockGuard lock(mutex_);  //-----------2
	//make use of internal state 
}

//Thread A
delete x;
x = NULL;

//Thread B
if(x){
	x->update();
}
```
崩溃分析
1. 线程A执行到1，已经持有互斥锁，继续往下执行
2. 线程B阻塞在2处。
3. 析构函数将mutex销毁，线程B有可能永远阻塞下去，也可能进入临界区，然后core dump。
### 死锁
```c
void swap(Conter &a, Conter &b){
	MutexLockGuard aLock(a.mutex_);
	MutexLockGuard bLock(b.mutex_);   //死锁发生位置
	int value = a.value_;
	a.value_ = b.value_;
	b.value_ = a.value_;
}
//如果线程A执行swap(a, b), 线程B执行swap(b, a)，就有可能死锁。
```
一个函数如果要锁住相同类型的多个对象，为了保证按相同的顺序加锁，我们可以比较mutex对象地址，始终先加锁地址较小的mutex。

### weak_ptr
```c
class Observable{
public:
	void register_(weak_ptr<Observer> x);
	void notifyObservers();
private:
	mutable MutexLock mutex_;
	std::vector<weak_ptr<Observer> > observers_;
};

void Observable::notifyObservers(){
	MutexLockGuard lock(mutex_);
	Iterator it = observers_.begin();
	while(it != observers_.end()){
		shared_ptr<Observer> obj(it->lock()); //尝试提升，这一步是线程安全的
		if (obj){
			obj->update();
			++it;
		}else{
			it = observers_.erase(it);	//删除指向下一个对象
		}
	}
}
```

## mutex
### 原则（RAII）
- 只用非递归mutex （不可重入mutex）
	同一线程多次对non-recusive mutex加锁会立刻导致死锁。
- 不手工调用lock()和unlock()函数，一切交给栈上的Guard对象的构造和析构函数负责
- 防止加锁顺序导致的死锁
- 不要在跨进程间使用mutex
- 加锁和解锁在同一个线程
```c
void post(const Foo& f){
	MutexLockGard lock(mutex);
	foos.push_back(f);
}

void traverse(){
	MutexLockGard lock(mutex);
	for(it=foos.begin(); it!=foos.end(); it++){
		it->doit();
	}
}
```
正常情况下没问题，但是一旦doit函数间接调用了post函数。结果
1. metex非递归，死锁。死锁比较容易debug，把各个线程的调用栈打印出来，很容易查看死锁，`thread apply all bt`。
2. mutex递归，push_back可能导致迭代器失效，程序可能会crash。

```c
class Inventory{
public:
	void add(Request* req){
		MutexLockGuard lock(mutex);
		request_.insert(req);
	}

	void remove(Request* req){
		MutexLockGuard lock(mutex);
		request_.remove(req);
	}

	void printall(){
		MutexLockGuard lock(mutex);
		print();
	}
private:
	std::set<Request*> requests_;
};

class Request{
public:
	void process(){
		MutexLockGuard lock(mutex);
		g_inventory.add(this);
	}

	~Request(){
		MutexLockGuard lock(mutex);
		sleep(1);
		g_inventory.remove(this);
	}

	void print(){
		MutexLockGuard lock(mutex);
	}
};

void threadFunc(){
	Request* req = new Requeset();
	req->process();
	delete req;
}

int main(){
	Thread thread(threadFunc);
	thread.start();
	usleep(500 * 1000);
	g_inventory.printAll();
	thread.join();
}
```