## 命令行传参

```cpp
#include <iostream>

using namespace std;

int main(int argc, char **argv)
{
    for(int i = 0; i < argc; i++)
        cout << argv[i] << endl;
    cout << "Hello, world!" << endl;
    return 0;
}
```

```bash
> g++ main.cpp -o main.out
> ./main.out abc hhh
./main.out
abc
hhh
Hello, world!
```



## 获取程序运行返回值

```bash
> echo $?
0
```