1. 宏定义 ’**#**‘ 和 ’**##**‘ 的使用说明: '**#**' 用于将代码转换成字符串; '**##**' 用于拼接代码; 使用范例:

```c++
#include <iostream>

#define PrintCode(x) #x
#define INT_TYPE(x) int##x##_t

int main(int argc, char *argv[])
{
	std::cout << PrintCode(int) << std::endl;	// 输出: int
	INT_TYPE(64) a;
	std::cout << sizeof(a) << std::endl;	// 输出 8 因为 a是 int64_t
	return 0;
}
```



2. 预处理指令

```c
#if FALSE	// 相当于 C语言中 if

#elif TRUE	// 相当于 C语言中 else if

#else		// 相当于 C语言中 else

#endif
```



3. C/C++ 不使用结构体字节对齐

```C++
/*减小内存占用的空间；结构体默认进行对齐，占用的空间比结构体内部成员变量字节加起来大，如果取消字节对齐，可以减小一部分空间。见下面具体例子。

直接将结构体作为通信协议（在低带宽下通讯）；在不同的平台下，保证结构体内基本数据的长度相同，同时取消结构体的对齐，就可以将定义的数据格式结构体直接作为数据通信协议使用。*/
#pragma pack (n)  // 编译器将按照n个字节对齐；
#pragma pack()   // 恢复先前的pack设置,取消设置的字节对齐方式
#pragma  pack(pop)// 恢复先前的pack设置,取消设置的字节对齐方式
#pragma  pack(1)  // 按1字节进行对齐 即：不行进行对齐
```

4. GUN C 允许声明长度为0的数组，零长度数组可用作结构的最后一个元素，该结构实际上是可变长度对象的标头，内存分配如下：

```c
struct line {
  int length;
  char contents[0];
};

struct line *thisline = (struct line *)malloc (sizeof (struct line) + this_length * sizeof(char));
thisline->length = this_length;
```

5. typedef 的前置声明：

```cpp
// a.h
class object
{
    ...
};
struct myStruct
{
    ...
};
typedef object defMyObject;
typedef myStruct defMyStruct;
```

```cpp
// b.h
typedef class object defMyObject;
typedef struct myStruct defMyStruct;
```

或者：

```cpp
// b.h
class object;
typedef object defMyObject;
struct myStruct;
typedef myStruct defMyStruct;
```



6. 将文件内容全部读出（包括回车符）

```c++
std::string Read_Image(const std::string &_csImagePath)
{
    std::ifstream file(_csImagePath.c_str(), std::ios::in);
    int iBufSize = 0;
    std::string sFileContent = "";
    char letter;
    if (file.is_open())
    {
        std::string sContent;
        while (file.get(letter))
        {
            iBufSize += 1;
            sFileContent += letter;
        }
    }
    return sFileContent;
}
```

