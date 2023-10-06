C++的 **bitset** 在 `<bitset>` 头文件中，它是一种类似数组的结构，它的每一个元素只能是０或１，每个元素仅用１bit空间。

一般来说bitset会让你的算法复杂度 /32

### 构造函数
bitset常用构造函数有四种，如下

```c++
bitset<4> bitset1;　　//无参构造，长度为４，默认每一位为０

bitset<8> bitset2(12);　　//长度为８，将12转换成二进制保存到后面，前面用０补充

string s = "100101";
bitset<10> bitset3(s);　　//长度为10，前面用０补充

char s2[] = "10101";
bitset<13> bitset4(s2);　　//长度为13，前面用０补充

cout << bitset1 << endl;　　//0000
cout << bitset2 << endl;　　//00001100
cout << bitset3 << endl;　　//0000100101
cout << bitset4 << endl;　　//0000000010101
```

### 特别注意

1. biset里十进制数进来以后直接用cout输出是正常的二进制。  
```c++
bitset<8> bitset2(12);
cout << bitset2 << endl;　　
// 输出： 00001100
```
但是实际上里面的二进制是倒序的。  
如果循环从0位输出到最后一位，如：
```c++
for (int i=0;i<8;++i) {
	cout<<b[i];
}
puts("");
```

输出的是倒序的：

```c++
00110000
```

如何实现2进制向10进制的转换，而且二进制的表示形式是一种逆序的形式，即低位在前。

```c++
int bin2dec(const string& bin)
{
    std::bitset<8> bs(string(bin.rbegin(), bin.rend()));
    return bs.to_ulong();
}
```

2. 用字符串构造时，字符串只能包含 ‘0’ 或 ‘1’ ，否则会抛出异常。  
构造时，需在<>中表明bitset 的大小(即size)。  
在进行有参构造时，若参数的二进制表示比bitset的size小，则在前面用０补充(如上面的栗子)；若比bitsize大，**参数为整数时取后面部分，参数为字符串时取前面部分**(如下面栗子)：

```c++
bitset<2> bitset1(12);　　//12的二进制为1100（长度为４），但bitset1的size=2，只取后面部分，即00

string s = "100101";　　
bitset<4> bitset2(s);　　//s的size=6，而bitset的size=4，只取前面部分，即1001

char s2[] = "11101";
bitset<4> bitset3(s2);　　//与bitset2同理，只取前面部分，即1110

cout << bitset1 << endl;　　//00
cout << bitset2 << endl;　　//1001
cout << bitset3 << endl;　　//1110
```

### 可用的操作符

bitset对于二进制有位操作符，具体如下

```c++
bitset<4> foo (string("1001"));
bitset<4> bar (string("0011"));

cout << (foo^=bar) << endl;       // 1010 (foo对bar按位异或后赋值给foo)
cout << (foo&=bar) << endl;       // 0010 (按位与后赋值给foo)
cout << (foo|=bar) << endl;       // 0011 (按位或后赋值给foo)

cout << (foo<<=2) << endl;        // 1100 (左移２位，低位补０，有自身赋值)
cout << (foo>>=1) << endl;        // 0110 (右移１位，高位补０，有自身赋值)

cout << (~bar) << endl;           // 1100 (按位取反)
cout << (bar<<1) << endl;         // 0110 (左移，不赋值)
cout << (bar>>1) << endl;         // 0001 (右移，不赋值)

cout << (foo==bar) << endl;       // false (0110==0011为false)
cout << (foo!=bar) << endl;       // true  (0110!=0011为true)

cout << (foo&bar) << endl;        // 0010 (按位与，不赋值)
cout << (foo|bar) << endl;        // 0111 (按位或，不赋值)
cout << (foo^bar) << endl;        // 0101 (按位异或，不赋值)
```
**此外，可以通过 \[ \] 访问元素(类似数组)，注意最低位下标为０**，如下：
```c++
bitset<4> foo ("1011");
cout << foo[0] << endl;　　//1
cout << foo[1] << endl;　　//1
cout << foo[2] << endl;　　//0
```
当然，通过这种方式对某一位元素赋值也是可以的，栗子就不放了。

### 可用函数

```c++
bitset<8> foo ("10011011");

cout << foo.count() << endl;　　//5　　（count函数用来求bitset中1的位数，foo中共有５个１
cout << foo.size() << endl;　　 //8　　（size函数用来求bitset的大小，一共有８位

cout << foo.test(0) << endl;　　//true　　（test函数用来查下标处的元素是０还是１，并返回false或true，此处foo[0]为１，返回true
cout << foo.test(2) << endl;　　//false　　（同理，foo[2]为０，返回false

cout << foo.any() << endl;　　//true　　（any函数检查bitset中是否有１
cout << foo.none() << endl;　　//false　　（none函数检查bitset中是否没有１
cout << foo.all() << endl;　　//false　　（all函数检查bitset中是全部为１
```

补充说明一下：test函数会对下标越界作出检查，而通过 \[ \] 访问元素却不会经过下标检查，所以，在两种方式通用的情况下，选择test函数更安全一些  
另外，含有一些函数：

```C++
bitset<8> foo ("10011011");

cout << foo.flip(2) << endl;　　//10011111　　（flip函数传参数时，用于将参数位取反，本行代码将foo下标２处"反转"，即０变１，１变０
cout << foo.flip() << endl;　　 //01100000　　（flip函数不指定参数时，将bitset每一位全部取反

cout << foo.set() << endl;　　　　//11111111　　（set函数不指定参数时，将bitset的每一位全部置为１
cout << foo.set(3,0) << endl;　　//11110111　　（set函数指定两位参数时，将第一参数位的元素置为第二参数的值，本行对foo的操作相当于foo[3]=0
cout << foo.set(3) << endl;　　  //11111111　　（set函数只有一个参数时，将参数下标处置为１

cout << foo.reset(4) << endl;　　//11101111　　（reset函数传一个参数时将参数下标处置为０
cout << foo.reset() << endl;　　 //00000000　　（reset函数不传参数时将bitset的每一位全部置为０
```

同样，它们也都会检查下标是否越界，如果越界就会抛出异常  
最后，还有一些类型转换的函数，如下：

```C++
bitset<8> foo ("10011011");

string s = foo.to_string();　　//将bitset转换成string类型
unsigned long a = foo.to_ulong();　　//将bitset转换成unsigned long类型
unsigned long long b = foo.to_ullong();　　//将bitset转换成unsigned long long类型

cout << s << endl;　　//10011011
cout << a << endl;　　//155
cout << b << endl;　　//155
```