**内容概述**
+ 对C++17特性全面覆盖
+ 启动和管理线程
+ 同步并发操作
+ 设计并发代码
+ 调试多线程应用

**About this book.**

本书是<span style="background:#d3f8b6">并发和多线程</span>机制指导书籍，基于C++最新标准(11，14，17)。从最基本的 <font color="#00b050">std::thread </font><font color="#00b050">std::mutex </font>和 <font color="#00b050">std::async</font> 的使用，到复杂的<font color="#00b050">原子操作和内存模型</font>。

## 1 thread类

### 1.1 构造函数

```cpp
#include <thread>
using namespace std;

int main()
{
    auto func = []() {};
    auto funcWithArg = [](int n) {};

    //通过无参数函数构造线程
    thread th1(func);

    //通过有参数函数及其参数构造线程
    thread th2(funcWithArg, 0);

    th1.join();
    th2.join();
}
```

thread类没有复制构造函数，但有移动构造函数。  
比如以下代码将th1这个线程对象转移至了th2，其中需要使用移动语义要用到move()将th1转换为右值。

```cpp
#include <iostream>
#include <thread>
using namespace std;

int main()
{
    auto func = []() {};

    thread th1(func);
    cout << "before move : th1 id " << th1.get_id() << endl;

    thread th2(move(th1));
    cout << "after move : th1 id " << th1.get_id() << endl;
    cout << "after move : th2 id " << th2.get_id() << endl;

    th2.join();
}
```

输出如下：

```text
before move : th1 id 20180
after move : th1 id 0
after move : th2 id 20180
```

### 1.2 其他成员函数：get_id、join、jionable、detach、swap

get_id()获取线程的id，对于没有传入一个函数进行构造的线程其id都是0。

```cpp
#include <thread>
using namespace std;

int main()
{
    auto func = []() {};

    thread th1(func);
    cout << "th1 id " << th1.get_id() << endl;

    thread th2;
    cout << "th2 id " << th2.get_id() << endl;

    th1.join();
}

stdout:
th1 id 21860
th2 id 0
```

jion()用于阻塞当前线程等待jion的线程执行完后返回，例如调用th1.jion()将当前线程线程直至th1线程执行完毕后才返回。以下代码在创建th1和th2之间使用了th1.jion()，则th1执行输出完之后才会创建th2并输出。

```cpp
#include <iostream>
#include <thread>
using namespace std;

void func(int id) {
    for (int i = 0; i < 3; i++)
        cout << "now it is thread " << id << endl;
}

int main()
{
    thread th1(func, 1);
    th1.join();

    thread th2(func, 2);
    th2.join();
}

stdout:
now it is thread 1
now it is thread 1
now it is thread 1
now it is thread 2
now it is thread 2
now it is thread 2
```

如果没有在th1和th2之间使用了`th1.jion()`，则th1的输出和th2的输出会是交错且无序的。

```cpp
#include <iostream>
#include <thread>
using namespace std;

void func(int id) {
    for (int i = 0; i < 3; i++)
        cout << "now it is thread " << id << endl;
}

int main()
{
    thread th1(func, 1);
    thread th2(func, 2);

    th1.join();
    th2.join();
}

stdout:
now it is thread 2now it is thread 1
now it is thread 1
now it is thread 1

now it is thread 2
now it is thread 2
```

joinable()从其名字就可看出用于判断当前线程是否可以join()，一般没有detach()的线程就是可以join()的，即便是这个线程已经执行完了，只是已经执行完的线程join()会立马返回。但是使用默认构造函数的线程是不可以join()的，可以理解成使用默认构造函数而没有传入一个函数的线程并不是一个真正的线程。另外已经join()了的也不能再次join()。

```cpp
#include <iostream>
#include <thread>
#include <chrono>
using namespace std;

void func(int id) {
    cout << "thread " << id << " down" << endl;
}

int main()
{
    thread th1(func, 1);

    //等待th1执行完毕
    this_thread::sleep_for(chrono::milliseconds(1));
    cout << "th1 joinable " << (th1.joinable() ? "true" : "false") << endl;

    thread th2;
    cout << "th2 joinable " << (th2.joinable() ? "true" : "false") << endl;

    th1.join();
    cout << "th1 joinable " << (th1.joinable() ? "true" : "false") << endl;
}

stdout:
thread 1 down
th1 joinable true
th2 joinable false
th1 joinable false
```

detach()用于把一个线程从当前线程分离，比如th1.detach()之后，当前线程将不再可通过th1访问到这个线程，当然这个线程也不再能被join()。被分离的线程会自己执行完并释放资源，但是如果在主线程创建一个子线程，并把这个子线程分离，当主线程返回时子线程若还未返回，子线程也会被动结束。如以下代码中th1在主线程执行完之前还没有执行完，因此不能完整打印100次。

```cpp
#include <iostream>
#include <thread>
#include <chrono>
using namespace std;

void func() {
    for (int i = 0; i < 100; i++)
        cout << "still alive " << i << endl;
}

int main()
{
    thread th1(func);

    this_thread::sleep_for(chrono::milliseconds(1));
    th1.detach();
}
```

swap()即将两个线程交换，底层是通过句柄实现的，可以理解为指针a指向实际线程1，而th1指向指针a；指针b指向实际线程2，而th2指向指针b。当交换时，仅需要改为th1指向指针b，th2指向指针a即可实现th1和th2线程的互换。

```cpp
#include <iostream>
#include <thread>
#include <chrono>
using namespace std;

int main()
{
    auto func = []() {};

    thread th1(func);
    thread th2(func);
    cout << "th1 id " << th1.get_id() << endl;
    cout << "th2 id " << th2.get_id() << endl;

    //thread类中的swap方法
    th1.swap(th2);
    cout << "swap" << endl;
    cout << "th1 id " << th1.get_id() << endl;
    cout << "th2 id " << th2.get_id() << endl;

    //也可以使用std::swap()
    swap(th1, th2);
    cout << "swap" << endl;
    cout << "th1 id " << th1.get_id() << endl;
    cout << "th2 id " << th2.get_id() << endl;

    th1.join();
    th2.join();
}

stdout:
th1 id 17028
th2 id 15232
swap
th1 id 15232
th2 id 17028
swap
th1 id 17028
th2 id 15232
```

## 2 this_thread中的辅助函数

this_thread::get_id()：用于获取当前线程id，与th1.get_id()类似，后者获取th1的线程id。

this_thread::yield()：当前线程休眠并将CPU资源让给其他线程，但此时当前线程依然处于就绪状态，而不是阻塞挂起状态，随时可能被再次调度。

```cpp
#include <iostream>
#include <thread>
#include <chrono>
#include <ctime>
using namespace std;

void func() {
    for (int i = 0; i < 5; i++) {
        auto before = chrono::high_resolution_clock::now();
        this_thread::yield();
        auto after = chrono::high_resolution_clock::now();
        auto duration = chrono::duration_cast<chrono::nanoseconds>(after - before);
        cout << "i am back, duration " << duration.count() << " ns" << endl;
    }
}

int main()
{
    thread th1(func);
    th1.join();
}

stdout:
i am back, duration 1300 ns
i am back, duration 500 ns
i am back, duration 200 ns
i am back, duration 200 ns
i am back, duration 200 ns
```

sleep_until()：传入一个指定时刻，线程将休眠到给指定时刻后被唤醒。  
sleep_for()：传入一个时长，线程将这么长时间后再被唤醒。  
和前面的yield()不同，这两个sleep会阻塞当前线程，等休眠时间满足预设要求后才重新转为就绪态，再等待CPU调度，所以实际休眠时间可能比预设时间稍长。

```cpp
#include <iostream>
#include <thread>
#include <chrono>
#include <ctime>
using namespace std;

int main()
{
    //使用this_thread::sleep_until()休眠至当前时刻3us之后的时刻
    auto before = chrono::high_resolution_clock::now();
    this_thread::sleep_until(before + chrono::microseconds(3));
    auto after = chrono::high_resolution_clock::now();
    auto duration = chrono::duration_cast<chrono::microseconds>(after - before);
    cout << "i am back, duration " << duration.count() << " us" << endl;

    //使用this_thread::sleep_for()休眠3us
    before = chrono::high_resolution_clock::now();
    this_thread::sleep_for(chrono::microseconds(3));
    after = chrono::high_resolution_clock::now();
    duration = chrono::duration_cast<chrono::microseconds>(after - before);
    cout << "i am back, duration " << duration.count() << " us" << endl;
}

stdout:
i am back, duration 4 us
i am back, duration 7 us
```

## 3 互斥锁

### 3.1 mutex类

mutex表示互斥锁，用于实现线程之间的互斥执行，主要通过以下方法实现。 lock()：当前线程尝试获取mutex，如果此时mutex已经被其他线程获取，则当前线程被阻塞直至获取到mutex。若成功获取到mutex，则在当前线程释放mutex之前也会阻塞其他线程获取此mutex。 unlock()：当前线程释放mutex的所有权。 以下示例中线程th1和th2都尝试修改count，通过mutex实现了同时只能有一个线程在修改其值。

```c++
#include <iostream>
#include <thread>
#include <mutex>
using namespace std;

volatile int myCount = 0;
mutex mt;

void addCount(int threadIndex) {
    while (myCount < 5) {
        mt.lock();
        myCount++;
        cout << "round " << myCount << " thread " << threadIndex << " win" << endl;
        mt.unlock();
    }
}

int main()
{
    thread th1(addCount, 1);
    thread th2(addCount, 2);
    th1.join();
    th2.join();
}

stdout:
round 1 thread 2 win
round 2 thread 2 win
round 3 thread 2 win
round 4 thread 2 win
round 5 thread 2 win
round 6 thread 1 win
```

> 当一个变量被声明为volatile时，编译器会禁止对该变量进行优化，以确保每次访问变量时都会从内存中读取其值，而不是从寄存器或缓存中读取。

对于为什么循环条件是myCount小于5却实际输出了6次的问题：比如当myCount为4的时候，th1和th2两个线程都可以进入到while循环，th2抢夺到了mutex并把myCount加一变成5后自己退出。而对于th1也已经进入了while循环，只是在等待mutex，此时它获得mutex，于是再把myCount加一，所以最后myCount成了6。 myCount前面加一个volatile关键字，主要是避免线程使用到了过时的数据：th1和th2两个线程可能会在两个不同的CPU核心并行工作，因为CPU的高速和内存的低速不匹配，他们都会把myCount先读入高速缓存，然后再在缓存中操作myCount。这样就可能会导致th1修改了myCount，而th2中依然在使用之前读入高速缓存的值。volatile让编辑器不要对myCount进行优化，意思就是每次使用myCount时都真正从内存中读取。

try_lock()：有的时候我们可能希望尝试去看看能不能获取mutex，如果可以获取就执行一些互斥的操作，但如果不能获取就先做点其他事情过一会再来尝试获取，而不是直接就把自己阻塞了，对于这种需求可以通过try_lock()实现。

```c++
#include <iostream>
#include <thread>
#include <chrono>
#include <mutex>
using namespace std;

mutex mt;

void func1() {
    mt.lock();
    this_thread::sleep_for(chrono::microseconds(3));
    mt.unlock();
}

void func2() {
    while (true) {
        if (mt.try_lock()) {
            cout << "i get the mutex" << endl;
            mt.unlock();
            break;
        }
        else {
            cout << "i can't get the mutex, i will try it later" << endl;
        }
    }
}

int main()
{
    thread th1(func1);
    thread th2(func2);
    th1.join();
    th2.join();
}

stdout:
i can't get the mutex, i will try it later
i can't get the mutex, i will try it later
i can't get the mutex, i will try it later
i get the mutex
```

以上示例在th1中使用sleep_for把自己阻塞，但阻塞时不会自动释放获取到的mutex，要等到后面手动unlock时才会被释放。另外以上案例实际上th1和th2谁先拿到mutex会受系统调度影响，输出不一定相同。

### 3.2 recursive_mutex类

recursive_mutex表示递归互斥锁，其用法与mutex基本一致，区别在于使用recursive_mutex时，同一个线程可以多次获取并释放，当获取次数与释放次数匹配时才真正释放。

```c++
#include <iostream>
#include <thread>
#include <mutex>
#include <functional>
using namespace std;

recursive_mutex mt;

void func1(function<void(void)> doSomething) {
    mt.lock();
    cout << "i get the mutex" << endl;
    doSomething();
    mt.unlock();
}

void func2() {
    mt.lock();
    cout << "i get it again" << endl;
    mt.unlock();
}

int main()
{
    thread th1(func1, func2);
    th1.join();
}

stdout:
i get the mutex
i get it again
```

其中在线程th1中，应该是不能直接访问func2的，所以创建线程的时候把func2当作参数手动传进去。

### 3.3 timed_mutex类

与mutex同样有lock()、try_lock()、unlock()，但是加入了两个和时间相关的互斥锁。

try_lock_for()：传入一个时长t，在t时间之内会尝试获取mutex，注意不是尝试获取一个在t时间内有效的mutex！！！如果能够获取到mutex，就返回true，之后就和unlock()以及try_lock()一样；如果没有获取到到mutex，不会和try_lock()一样立即放回false，而是会阻塞一直等到获取到mutex返回true、或者等到时间超出设置的t后返回false。 以下代码中线程th2使用mt.try_lock_for(chrono::seconds(3))尝试在3秒内获取mutex，但最终没有成功，可以看到从开始尝试到返回false之间th2被阻塞了3秒钟。

```c++
#include <iostream>
#include <thread>
#include <mutex>
#include <chrono>
using namespace std;

timed_mutex mt;

void func1() {
    mt.lock();
    this_thread::sleep_for(chrono::seconds(5));
    mt.unlock();
}

void func2() {
    mt.lock();
    auto before = chrono::high_resolution_clock::now();
    if (mt.try_lock_for(chrono::seconds(3))){
        cout << "i get the mutex";
        mt.unlock();
    }
    else{
        auto after = chrono::high_resolution_clock::now();
        auto duration = chrono::duration_cast<chrono::seconds>(after - before);
        cout << duration.count() << " s passed, i can't get the mutex";
    }
}

int main()
{
    thread th1(func1);

    thread th2(func2);

    th1.join();
    th2.join();
}

stdout:
3 s passed, i can't get the mutex
```

try_lock_until()：传入一个时刻t，在时刻t之前会尝试获取mutex，同理和try_lock()的立即返回不同，在获取到之前同样会被阻塞，直至成功获取返回true，到了设定时刻仍未成功获取返回false。 可以使用和try_lock_for()类似的代码逻辑来测试其功能。

```c++
#include <iostream>
#include <thread>
#include <mutex>
#include <chrono>
using namespace std;

timed_mutex mt;

void func1() {
    mt.lock();
    this_thread::sleep_for(chrono::seconds(5));
    mt.unlock();
}

void func2() {
    mt.lock();
    auto before = chrono::high_resolution_clock::now();
    if (mt.try_lock_until(before + chrono::seconds(3))){
        cout << "i get the mutex";
        mt.unlock();
    }
    else{
        auto after = chrono::high_resolution_clock::now();
        auto duration = chrono::duration_cast<chrono::seconds>(after - before);
        cout << duration.count() << " s passed, i can't get the mutex";
    }
}

int main()
{
    thread th1(func1);

    thread th2(func2);

    th1.join();
    th2.join();
}

stdout:
3 s passed, i can't get the mutex
```

### 3.4 recursive_timed_mutex类

这一个一看名字就知道是把recursive_mutex和timed_mutex的功能相结合，用法都是一样的。

### 3.5 lock_guard类

实际上lock_guard仅仅是对锁的很小的一个封装，可以传入一个锁来构造一个lock_guard对象，在构造该对象时就会在构造函数中自动获取锁（和lock()一样，在成功获取之前会被阻塞），而在析构函数中就会释放该锁。实际使用时只需要在要用到锁的地方构造一个lock_guard对象就不用管了，因为当该函数执行完，处于该函数作用域内的lock_guard对象会自动被析构，也就自动释放锁资源了。 其实使用lock_guard比起直接操作锁也方便不了多少，主要是体现了一种资源管理的思想：在其构造时获取资源，在对象生命期控制对资源的访问使之始终保持有效，最后在对象析构的时候释放资源。也就是RAII。

```c++
#include <iostream>
#include <thread>
#include <mutex>
using namespace std;

volatile int myCount = 0;
mutex mt;

void addCount(int threadIndex) {
    while (myCount < 5) {
        lock_guard<mutex> guard(mt);
        myCount++;
        cout << "round " << myCount << " thread " << threadIndex << " win" << endl;
    }
}

int main()
{
    thread th1(addCount, 1);
    thread th2(addCount, 2);
    th1.join();
    th2.join();
}
```

以上代码中guard会在每次执行到while的大括号自动析构，加入在while中除了把myCount加一，还有很多其他并不是互斥的操作，那么我们更加希望在myCount加一后就立马解锁，以免不必要的长时间阻塞其他人。但是这个需求使用lock_guard类是不能很方便办到的，这也是lock_guard的主要缺点：不够灵活。可以用以下的unique_lock更加灵活的实现这个需求，当然也可以直接操作锁实现。

### 3.6 unique_lock类

unique_lock在lock_guard的基础上，还能随时灵活的解锁上锁。在unique_lock的构造函数中会获取并上锁（还可以通过参数指定在构造时不立马上锁），之后可以直接通过unique_lock对象进行lock、unlock、try_lock等操作，在析构函数时会自动根据当前是否已经释放了锁来决定要不要释放锁。

```c++
mutex mt;

//构造时自动上锁
unique_lock<mutex> lock1(mt);

//构造时先不上锁，后面需要自己手动上
unique_lock<mutex> lock2(mt, defer_lock);

//lock, unlock, try_lock系列和锁自己的相关用法类似
lock1.lock();
lock1.unlock();

//unique_lock有移动构造函数（但是没有复制构造函数，lock_guard二者都没有）
unique_lock<mutex> lock3(move(lock1));
```

如果仅仅是操作锁的情况下，lock_guard和unique_lock能做的事，都可以直接操作锁来实现，甚至可能直接操作锁还更加方便灵活。但是lock_guard和unique_lock更多的是体现了一种资源管理的思路，从某种角度上还能减少因为忘记释放锁而导致的死锁问题，对于大型项目开发还是有一定好处的。另外，unique_lock还能被用到条件变量上。

## 4 条件变量

条件变量的主要意义在于，提供了一种更加灵活的使线程状态切换为阻塞/就绪的方式，具体的说：可以实现在某指定表达式为false时阻塞线程，当其变为true时使线程由阻塞转为就绪。

### 4.1 condition_variable类

condition_variable类仅有默认构造函数。
wait：wait有两个版本
1. 仅传入一个unique_lock\<mutex\>对象：调用后会把当前线程阻塞，直至其他线程通过condition_variable对象唤醒当前线程；
2. 传入一个unique_lock\<mutex\>对象以及一个返回bool的表达式或函数：调用后若表达式的值或函数返回值为false才会阻塞当前线程，直至表达式的值或函数返回值为true且有其他线程通过condition_variable对象唤醒当前线程。当线程因为调用wait被阻塞时，会自动给mutex解锁，当线程被唤醒时也会自动获取mutex（如果没有获取到就继续阻塞）。

notify_all：尝试唤醒所有在等待当前condition_variable对象的线程，当然同一个互斥锁同时只能一个线程拥有，没有获取到锁的依然会被阻塞。另外如果等待线程调用wait时还传入了表达式，则表达式当前要为true才能被唤醒。

notify_one：唤醒随机某一个在等待当前condition_variable对象的线程，同样如果等待线程调用wait时还传入了表达式，则表达式当前要为true才能被唤醒。

以下代码展示了wait和notify_all的基础用法，线程th1在state为true时输出，th2在state为false时输出，实现了两个线程有序的交替输出。


```c++
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
using namespace std;

mutex mt;
condition_variable cv;
volatile bool state = true;

void func(int index, bool needState) {
    for (int i = 0; i < 3; i++) {
        unique_lock<mutex> lck(mt);
        cv.wait(lck, [needState]() {return state == needState; });
        cout << " i am thread " << index << endl;
        state = !state;
        cv.notify_all();
    }
}

int main()
{
    thread th1(func, 1, true);
    thread th2(func, 2, false);
    th1.join();
    th2.join();
}

stdout:
 i am thread 1
 i am thread 2
 i am thread 1
 i am thread 2
 i am thread 1
 i am thread 2
```

和前面的try_lock类似，wait同样有wait_for和wait_until这两个版本。 wait_for：阻塞时间超过了指定时间后会自动返回唤醒。 wait_until：过了指定时刻若还被阻塞就自动返回唤醒。 这两个方法和wait类似，同样都有参数为锁和时间的两个参数版本，以及参数为锁、时间和返回bool值的表达式的三个参数版本。对于传入了表达式的版本，可以非常方便的通过函数的返回值区分是成功被唤醒、还是超时自动唤醒。当超时自动返回时，即便表达式的值为false也会唤醒，并自动获取mutex。当然具体唤醒时间同样还是会受系统线程调度影响，不一定完全和设定时间相同。以下示例中wait_for和wait_until都在阻塞时间超出设定值之后自动返回了，即便此时传入的tmp依然为false，但并不影响超时唤醒。另外也可以看到超时唤醒时wait_for和wait_until都返回了false，且实际阻塞时间受系统线程调度影响与设定值不完全符合。

```c++
#include <iostream>
#include <thread>
#include <mutex>
#include <chrono>
#include <condition_variable>
using namespace std;

mutex mt;
condition_variable cv;

void func() {
    unique_lock<mutex> lck(mt);
    auto tmp = []() {return false; };

    auto before = chrono::high_resolution_clock::now();
    if (cv.wait_for(lck, chrono::microseconds(5), tmp)) {
        cout << "i am back in 5 us";
    }
    else {
        auto after = chrono::high_resolution_clock::now();
        auto duration = chrono::duration_cast<chrono::microseconds>(after - before);
        cout << "time out, " << duration.count() << " us have past" << endl;
    }

    before = chrono::high_resolution_clock::now();
    if (cv.wait_until(lck, before + chrono::microseconds(5), tmp)) {
        cout << "i am back in 5 us";
    }
    else {
        auto after = chrono::high_resolution_clock::now();
        auto duration = chrono::duration_cast<chrono::microseconds>(after - before);
        cout << "time out, " << duration.count() << " us have past" << endl;
    }
}

int main()
{
    thread th1(func);
    th1.join();
}

stdout:
time out, 11687 us have past
time out, 15459 us have past
```

### 4.2 condition_variable_any 类

前文的condition_variable有一大限制，即只能使用unique_lock\<mutex\>，也即只能使用最基础的锁mutex。但有时候确实有需要用到例如recursive_mutex等其他类型的锁，这个时候就可以使用condition_variable_any类。在具体的用法上condition_variable_any与condition_variable相似度很高，不再赘述。

## 5 原子操作

### 5.1 从一个使用示例引入

假设存在某种场景，在该场景中有多个线程会操作修改同一个数据，若不对这些线程的操作进行一定的约束，最终该数据的结果可能就不是其理想的值。比如以下示例，多个线程都会对myCount先自增在自减多次，理想情况下myCount最终结果应该不变，但实际上如果没有对各线程操作myCount时进行一定约束，myCount最终的结果会是一个不确定的值。出现这种结果是因为同时可能有不止一个线程在对myCount进行修改，例如th1在修改的过程中，th2也试图修改myCount，最终myCount的结果将不确定。

```c++
#include <iostream>
#include <vector>
#include <thread>
using namespace std;

volatile int myCount;

void func() {
    for (int i = 0; i < 200000; i++) {
        myCount++;
        myCount--;
    }
}

int main()
{
    myCount = 0;
    int threadCount = 10;
    vector<thread> rec(threadCount);

    for (int i = 0; i < threadCount; i++) {
        rec[i] = thread(func);
    }

    for (int i = 0; i < threadCount; i++) {
        rec[i].join();
    }

    cout << myCount << endl;
}

stdout:
8
```

对于这个问题，可以想到的解决方法有，我们限定同时只能有一个线程在访问修改myCount，就能避免两个或者多个线程同时想要修改myCount，这也是之前加锁的解决思路。从另外一个角度想，既然问题的关键是一个线程在修改的过程中（没有修改完成），出现了另外一个线程也试图修改myCount，才最终导致出现问题，那么如果说对myCount的修改是一个不可分割的过程，就不会出现某线程修改到一半，另外一个线程也试图修改的问题。换句话说，如果对myCount的修改是一个不可分割的过程，那么仅存在还未修改和修改完成两种状态，没有修改到了一般这种状态，也就没有前述问题。原子操作就实现了以上思路，将myCount换成一个原子操作对象，随后自增和自减就都成为不可分割的过程，进而也解决了上述myCount值不对的问题。

```c++
#include <iostream>
#include <vector>
#include <thread>
#include <atomic>
using namespace std;

atomic<int> myCount;

void func() {
    for (int i = 0; i < 200000; i++) {
        myCount++;
        myCount--;
    }
}

int main()
{
    myCount = 0;
    int threadCount = 100;
    vector<thread> rec(threadCount);

    for (int i = 0; i < threadCount; i++) {
        rec[i] = thread(func);
    }

    for (int i = 0; i < threadCount; i++) {
        rec[i].join();
    }

    cout << myCount << endl;
}

stdout:
0
```

针对当前案例谈谈原子操作底层是如何解决此问题的：首先如果所有对myCount的操作都在同一个线程而不是在多个线程中，那即便myCount就是普通的int也不会出错，因为在上一个修改没有执行完之前肯定不会开始下一个修改。但对于多线程的情况，不同线程可能在不同的CPU核心上并行，确实存在多个线程在同一时刻访问myCount的可能性。而对于myCount的修改虽然我们描述为原子操作，但它必然不是瞬间完成的，而是或多或少有一个执行的时间，那原子操作是怎么解决“线程1在修改myCount的期间，位于另外一个CPU核心的另外一个线程同时也要修改myCount”这一问题的呢？实际上和“加锁确保同时只有一个线程在访问myCount”的思路是一样的，简单来说使用原子操作时，当某个线程对myCount进行修改时，其他同时想要对myCount进行存在冲突的访问操作都会被阻塞。

### 5.2 atomic使用简介

首先是构造函数，如果想要对一个int进行原子操作，可以构造一个atomic_int对象；如果想要对char进行原子操作，则可以构造一个atomic_char……不过更加方便统一的，可以通过atomic\<Type\>来构造一个可以对Type类型进行原子操作的对象，例如以上示例的atomic\<int\>。

atomic类定义了很多原子操作方法，例如： 以上示例代码的 myCount++; 实际上即 myCount.fetch_add(1); 以上示例代码的 myCount--; 实际上即 myCount.fetch_sub(1); fetch_add方法实现获取并加一，fetch_sub实现获取并减一，均为原子操作。 atomic类更多原子操作方法可以直接查看微软官方文档[atomic](https://link.zhihu.com/?target=https%3A//docs.microsoft.com/zh-cn/cpp/standard-library/atomic%3Fview%3Dmsvc-170)

### 5.3 使用原子操作实现两个线程交替打印

通过原子操作类型定义一个状态state，我们让state为1时第一个线程打印，state为2时第二个线程打印，具体如下，也是很多题解的类似思路。

```c++
class FooBar {
private:
    int n;
    atomic<int> state;

public:
    FooBar(int n) {
        this->n = n;
        state.store(1);
    }

    void foo(function<void()> printFoo) {
        
        for (int i = 0; i < n; i++) {
            while(state.load() != 1){
                this_thread::yield();
            }
        	// printFoo() outputs "foo". Do not change or remove this line.
        	printFoo();
            state.store(2);
        }
    }

    void bar(function<void()> printBar) {
        
        for (int i = 0; i < n; i++) {
            while(state.load() != 2){
                this_thread::yield();
            }
        	// printBar() outputs "bar". Do not change or remove this line.
        	printBar();
            state.store(1);
        }
    }
};
```

以上方案实际上就是两个线程都通过轮询当前state状态来实现同步的，虽然说当状态与理想不一致时使用了yield让自己休眠，但本质还是轮询。比起通过条件变量来实现的方法，这种方法耗时可能会更少，直观上来说轮询一般也能更快的得知state的改变；但是从CPU资源的有效利用率来讲，这种方案直观上感觉不如条件变量，因为还是耗费了大量的CPU资源用于反复查询state状态，虽然自己可能更快了，但是对其他程序和CPU来讲不够友好。因此，个人认为原子操作应该更多主要用在多个线程对同一块内存读写上，而使用原子操作搭配轮询实现调节多个线程执行顺序的这种用法应该不是一个好的选择。

## 6 信号量

### 6.1 信号量简介


信号量也是多线程同步的一种方式，有以下3中类型： 二进制信号量：某资源同时最多只能由一个线程拥有，也即信号量只能为1（表示当前没有线程获取该资源），或者为0（表示当前资源已经被某个线程获取），和mutex几乎一个道理。 整型信号量：某资源同时最多可以被多个线程拥有，其值代表当前还能有几个线程可以获取该资源，当其值小于0时，尝试获取该资源的线程将被阻塞。 记录型信号量：在整型信号量的基础上，还带有一个队列，队列中记录了当前没有获取到资源而被阻塞的线程，当有线程释放资源时，可以方便的唤醒阻塞资源。 严格来说C++语言本身没有信号量，但是利用mutex和condition_variable可以很方便的实现整型信号量，另外由于condition_variable本身就有阻塞线程的记录和唤醒，因此也能直接实现记录型信号量的功能。

### 6.2 利用mutex和condition_variable实现信号量

思路总结： 
1. 在构造时传入一个值，表示当前资源同时最多可以被多少个线程拥有。 
2. 每次wait就将该值减一，如果减一后依然为非负数，则说明当前想要拥有该资源的线程数还没有超过设定的最大值，就成功获取：如果减一后为负数，则阻塞等待资源被其他线程释放。 
3. 每次signal时将该值加一，如果加一后值小于等于0，说明存在等待该资源被阻塞的线程，便通知其中某一个唤醒。

```c++
#include <iostream>
#include <thread>
#include <atomic>
#include <mutex>
#include <vector>
using namespace std;

class MySemaphore
{
public:
    MySemaphore(int n = 1) {
        count = n;
    }

    void wait() {
        unique_lock<mutex> lck(mt);
        count--;
        if (count < 0)
            cv.wait(lck);
    }

    void signal() {
        unique_lock<mutex> lck(mt);
        count++;
        if (count <= 0)
            cv.notify_one();
    }

private:
    int count;
    mutex mt;
    condition_variable cv;
};

void func(int index, MySemaphore* sem) {
    (*sem).wait();
    cout << "thread " << index << " get the resource" << endl;

    cout << "thread " << index << " release the resource" << endl;
    (*sem).signal();
}

int main()
{
    MySemaphore sem(3);

    int threadCount = 10;
    vector<thread> ths(threadCount);

    for (int i = 0; i < threadCount; i++) {
        ths[i] = thread(func, i, &sem);
    }

    for (int i = 0; i < threadCount; i++) {
        ths[i].join();
    }
}

stdout：
thread 0thread 2 get the resource get the resource
thread 0 release the resource

thread thread 1 get the resource
thread 1 release the resource
2 release the resource
thread 3 get the resource
thread 3 release the resource
thread 5 get the resource
thread 5 release the resource
thread 6 get the resource
thread 6 release the resource
thread 7 get the resource
thread 7 release the resource
thread 8 get the resource
thread 8 release the resource
thread 9 get the resource
thread 9 release the resource
thread 4 get the resource
thread 4 release the resource
```

以上实现方案中，MySemaphore类里的count的加一和减一的时机和大部分信号量的实现相同，count可能为负。但我个人更加倾向于下面这种方案，count不会为负，更加更加清晰。

```c++
class MySemaphore
{
public:
    MySemaphore(int n = 1) {
        count = n;
    }

    void wait() {
        unique_lock<mutex> lck(mt);
        cv.wait(lck, [this]() {return count > 0; });
        count--;
    }

    void signal() {
        unique_lock<mutex> lck(mt);
        count++;
        cv.notify_one();
    }

private:
    int count;
    mutex mt;
    condition_variable cv;
};
```