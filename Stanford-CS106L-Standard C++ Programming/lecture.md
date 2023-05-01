[toc]

## 0 History and Philosophy of C++

**Assembly**

* do simple tasks with lots of code
* unportable



**C++ philosophy**

* only add features if they solve an actual problem
* allow programmer full control
* enforce safety at compile time whenever possible
* don't sacrifice performance except as a last resort



## 1 Streams I

**introduction**

stream:

I/O abstraction, a source/destination of characters of indefinite length



usage:

insert any primitive type into stream, for other types, explicitly tell C++ how to do this



idea:

> stream allows a C++ programmer to convert between the string representation of data, and the data itself.

关于这句话，我的理解是输出的过程是从 data 到 string representation 的转化，而输入就是从 string representation 得到 data 的过程



**output stream(`std::ostream`)**

* insertion `<<` converts data to string, and sends it to stream

* special: `std::ofstream`



**input stream(`std::istream`)**

* `>>` gets data from stream as a string and converts it into the appropriate type
* special: `std::ifstream`



**read data from file**

`>>`会一直读，直到出现空格或者换行符

`getline`会一直读整行，直到出现换行符，最后结果不包括换行符



## 2 Streams II

**buffering**

accumulate characters in temporary buffer, when buffer is full, write out all contents to output device at once

手动 flush buffer:

```c++
stream.flush(ch);	// 一般用这个
stream << std::flush;	// 要打印时用这个
stream << std::endl;	// 要换行用这个
```

有的 stream 是没有 buffer 的，比如`std::cerr`



**stream bits**

* good bit
* eof bit：到末尾了
* fail bit：前面的 stream 操作出错了
* bad bit：前面的操作出现了不可恢复的错误

```c++
while(true) {
    stream >> temp;
    if(temp.fail()) break;	// 读了数据之后，检查数据是否有效
    do_something(temp);
}
```

由于`>>`、`<<`会返回 stream，上面的代码可以简化为：

```c++
while(stream >> temp) {
    do_something(temp);
}
```



**stream manipulator**

* common: `endl`, `ws`(skip whitespaces until it finds another character), `boolalpha`(print true/false for bools)
* numeric: `hex`, `setprecision`
* padding: `setw`, `setfill`



**stringstream**

有时候，我们需要将 string 像 stream 的方式处理，这时就会用到 `stringstream`

使用场景：对 string 分词



## 3 Sequence Containers

![image-20230429204647109](assets/image-20230429204647109.png)



sequence containers: vector, list, deque



## 4 Associative Containers and Iterators

map, set, unordered_map, unordered_set



**iterator**

`iterator`的类型形如`set<int>::iterator`

`map<string, int>`的`iterator`指向`std::pair<string, int>`



## 5 Advanced Associative Containers

multimap 可以对同一个 key ，存储多个 value。比如一个人可能有多个地址



## 6 Templates

template function 可以隐式实例化（instantiation），也可以显式实例化

> * 显式实例化能提高编译效率，一是没有隐式实例化中的推断过程，二是把显式实例化放到一个文件中，让其他文件包含，使用时只需要实例化一次，减少实例化的次数。例如标准库中，`std::string`就是从`std::basic_string`显式实例化而来的
>
> * 使用显式实例化，可以把实现放到 cpp 文件中，而不必放在头文件中
>
>   ```cpp
>   // template.cpp
>   template<typename T>
>   T min(T a, T b) {
>       std::cout << "my min func" << std::endl;
>       return (a < b) ? a : b;
>   }
>     
>   template int min<int>(int, int);
>   ```
>
>   ```cpp
>   // template.h
>   template<typename T>
>   T min(T a, T b);
>   ```
>
>   ```cpp
>   // main.cpp
>   int main() {
>       std::cout << min(3, 5) << std::endl;
>       std::cout << min(3.14, 5.35) << std::endl;
>       std::cout << min('d', 'c') << std::endl;
>       return 0;
>   }
>   ```



## 7 Algorithm
