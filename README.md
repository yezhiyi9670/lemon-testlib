# lemon-testlib

Lemon上的Codeforces风格spj库

还在担心将洛谷或Codeforces的Special Judge转移到Lemon要花许多功夫吗？还在担心由于Lemon的校验器不严谨而造成校验器出错吗？是时候试下lemon-testlib了。

（这是一个仿造[testlib](https://github.com/MikeMirzayanov/testlib/)的Special Judge评测库，目前是实验版本，并且只支持checker）

如果Special Judge没有必要读取选手的源代码，那么[LemonLime](https://github.com/iotang/Project_LemonLime)用户可以使用[这个头文件](https://github.com/iotang/Project_LemonLime/blob/master/testlib_for_lemons.h)（这个头文件有交互题、生成器等更多功能）。

## 安装说明

不要问我关于安装的具体事情，因为我并不了解你用的IDE（除非是Dev C++）。我可以告诉你，这个头文件的使用方法是这样的：

```cpp
#include "lemon-testlib.h"
using namespace ltl;
```

## 关于Special Judge

在OI的题目中，有的题目有部分分，而有的题目不止一种解。这时候，仅仅有一个输出文件就不能解决问题了，我们需要一个程序来判断解的正确性，并给出得分。

## 代码示例

### 浮点误差判定

题目要求输出一个浮点数。如果选手输出和答案误差小于`0.005`，认为答案正确。

```cpp
#include<bits/stdc++.h>
#include "lemon_testlib.h"
using namespace std;
using namespace ltl;

int main(int argc,char **argv) {
    registerTestlibCmd(argc,argv);

    double jans = ans.readReal();
    double pans = ouf.readReal();

    if(abs(jans - pans) < 0.005) {
        quitf(_ok,"答案是%lf",pans);
    }
    else {
        quitf(_wa,"答案是%lf，但是选手输出了%lf",jans,pans);
    }
}

```

评测结果示例：

情况|评测结果
-|-
答案正确|**答案正确** 答案是1.25
答案错误|**答案错误** 答案是1.25，但是选手输出了1.3
答案正确，但是输出不止一个浮点数|**答案错误** 选手输出中有多余的内容
没有输出|**答案错误** 期望读到一个实数，然而读取到EOF（选手输出）
输出了奇怪的东西|**答案错误** 期望读到一个实数，然而读取到ljxakioi（选手输出）

### 2048游戏

题目给了一个随机数生成器，然后给出了2048游戏中随机添加方块的方法。选手要求输出若干行，每行都是left、right、up或down中的一个。最后输出一行`ok`作为结束。

评测程序要在2048游戏上执行选手输出的操作（最多允许200000次操作）。完成后，如果得到512方块，得全分（5分）；如果没有512而同时有128和256，得60%；如果只有256，得40%；如果只有128，得20%；其余情况不得分。

详细题面请看[这里](https://www.luogu.org/problem/T97750)

```cpp
// !This is a checker program
// status: [Hide]
// oj:     [luogu]

#include<stdlib.h>
#include<iostream>
#include<string>
#include<time.h>
#include<assert.h>
#include "lemon_testlib.h"
using namespace std;
using namespace ltl;

namespace Problem {
    struct _rnd_mt13325 {
        signed sd;
        _rnd_mt13325() {
            sd = time(NULL);
        }
        inline signed g()
        {
            sd ^= sd << 13;
            sd ^= sd >> 7;
            sd ^= sd << 11;
            sd = (~sd);
            return sd;
        }
    };

    _rnd_mt13325 _rnd;
    void init() {
        scanf("%d",&_rnd.sd);
    }
    void setseed(int t) {
        _rnd.sd=t;
    }
    pair<int,int> rand() {
        return make_pair(abs(_rnd.g())%100<90 ? 2:4, abs(_rnd.g()));
    }
};

struct Grid {
    int arr[4][4];
    int tmp[4][4];
    string _env;

    int __(int i,int j,int k=-1) {
        if(k==-1) return arr[i][j];
        return arr[i][j]=k;
    }

    int _(int i,int j,int k=-1) {
        if(_env=="left") {
            return __(i,j,k);
        }
        else if(_env=="right") {
            return __(i,3-j,k);
        }
        else if(_env=="down") {
            return __(3-j,i,k);
        }
        else if(_env=="up") {
            return __(j, i, k);
        }
        else return -1;
    }

    bool addTile() {
        int cnt=0;
        for(int i=0;i<4;i++) {
            for(int j=0;j<4;j++) {
                if(!arr[i][j]) cnt++;
            }
        }
        pair<int,int> insData = Problem::rand();
        if(!cnt) return true;
        insData.second %= cnt;
        int id=0;
        bool inserted = false;
        for(int i=0;i<4;i++) {
            for(int j=0;j<4;j++) {
                if(!arr[i][j]) {
                    if(id == insData.second) {
                        arr[i][j] = insData.first;
                        inserted = true;
                    }
                    id++;
                }
            }
        }
        // return 1;
        return inserted;
    }

    int nextPos(int i,int j) {
        for(int k=j+1;k<4;k++) {
            if(_(i,k)) return k;
        }
        return 4;
    }

    bool move(string op) {
        _env = op;
        for(int i=0;i<4;i++) {
            for(int j=0;j<4;j++) {
                tmp[i][j] = arr[i][j];
            }
        }

        for(int i=0;i<4;i++) {
            for(int j=0;j<4;) {
                int k=nextPos(i,j);
                if(k>=4) break;
                if(_(i,j) == _(i,k)) {
                    _(i,j,_(i,j)+_(i,k));
                    _(i,k,0);
                }
                j = nextPos(i,j);
            }
        }

        for(int i=0;i<4;i++) {
            int k=0;
            for(int j=0;j<4;j++) {
                if(_(i,j)) _(i,k++,_(i,j));
            }
            for(int j=k;j<4;j++) {
                _(i,j,0);
            }
        }

        for(int i=0;i<4;i++) {
            for(int j=0;j<4;j++) {
                if(tmp[i][j]!=arr[i][j]) return 1;
            }
        }
        return 0;
    }

    void print() {
        cerr<<"----- Grid info -----"<<endl;
        for(int i=0;i<4;i++) {
            for(int j=0;j<4;j++) {
                if(arr[i][j]) fprintf(stderr,"%4d",arr[i][j]);
                else fprintf(stderr,"   -");
            }
            cerr<<endl;
        }
        cerr<<"---------------------"<<endl;
    }
}grid;

// inf,ouf,ans
int main(int argc,char *argv[]) {
    registerTestlibCmd(argc,argv);

    Problem::setseed(inf.readInt());

    // ofstream fout("ans.txt");

    assert(grid.addTile());
    assert(grid.addTile());
    // grid.print();
    for(int i=1;i<=200001;i++) {
        string op;
        op = ouf.readLine();
        // getline(cin,op);
        if(!op.empty() && op[op.length()-1]=='\r') op=op.substr(0,op.length()-1);
        if(op=="ok") {
            int has128=0;
            int has256=0;
            int has512=0;
            for(int i=0;i<4;i++) {
                for(int j=0;j<4;j++) {
                    if(grid.arr[i][j]==512) has512 = 1;
                    if(grid.arr[i][j]==256) has256 = 1;
                    if(grid.arr[i][j]==128) has128 = 1;
                }
            }
            if(has512) quitf(_ok,"游戏胜利，5分");
            if(has256 && has128) quitp(0.6,"游戏中同时存在256和128，3分");
            if(has256) quitp(0.4,"游戏中存在256，2分");
            if(has128) quitp(0.2,"游戏中存在128，1分");
            quitf(_wa,"游戏失败，0分");
        }
        if(op=="left" || op=="right" || op=="up" || op=="down") {
            if(grid.move(op)) assert(grid.addTile());
            // grid.print();
            // fout<<op<<endl;
        }
        else {
            if(!op.empty()) quitf(_wa,"在第 %d 行发现未预料的操作 %s",i,op.c_str());
            else quitf(_wa,"在第 %d 行发现空行",i);
        }
    }

    quitf(_wa,"最大步骤数 200000 已超出");
}

```

评测状态示例：

情况|评测结果
-|-
输出ok之前有非法内容|**答案错误** 在第 198 行发现未预料的操作 iakioi
输出ok之前有空行|**答案错误** 在第 241 行发现空行
操作完成后游戏中同时有256和128|**答案部分正确** 游戏中同时存在256和128，3分
输出ok后输出了奇怪的东西|**答案错误** 选手输出中有多余的内容
末尾没有输出ok|**答案错误** 在第 254 行发现空行

### Quine

要求选手只能提交C++程序`quine.cpp`，并且这份程序编译后能够输出自己的源代码。

```cpp
#include<bits/stdc++.h>
#include "lemon_testlib.h"
using namespace std;
using namespace ltl;

int main(int argc,char **argv) {
    registerTestlibCmd(argc,argv);
    registerSourceReader(argc,argv,{"quine.cpp"});

    int bytes = 0;
    while(1) {
        if(ouf.eof() && source.eof()) {
            quitf(_ok,"正确，%d字节",bytes);
        }
        else if(ouf.eof() || source.eof()) {
            if(ouf.eof()) quitf(_err,"输出过短，%d字节",bytes);
            if(source.eof()) quitf(_err,"输出过长，选手程序有%d字节",bytes);
        }
        char c1,c2;
        bytes++;
        if((c1 = ouf.readChar()) != (c2 = source.readChar())) {
            quitf(_err,"在选手输出中第%d个字节读到%c，期望%c",bytes,c1,c2);
        }
    }
}

```

## 函数说明

### 全局变量

- `testlibInput inf`  
测试数据中的输入文件

- `testlibInput ans`  
测试数据中的输出（答案）文件

- `testlibInput ouf`  
选手的输出文件

- `testlibInput source`  
选手提交的程序文件

- `int testscore`  
当前测试点的总分

- `FILE *fscore`  
[只写的文件指针] 当前测试点得分应当输出到这个文件。理论上Special Judge程序无需使用这个变量。

- `FILE *freport`  
[只写的文件指针] Special Judge的详细信息应当输出到这个文件。理论上Special Judge程序无需使用这个变量。

### 初始化

- `int main(int argc,char **argv)`  
这是main函数的正确写法（不要加多余的const）

- `void registerTestlibCmd(int argc,char **argv)`  
初始化testlib（传入的参数应当是main函数的两个参数）。初始化后，testlib中除了`testlibInput source`之外的其他变量和函数就可以使用了。

- `void registerSourceReader(int argc,char **argv,initializer_list<string> lst)`  
寻找并准备读取选手提交的源代码（一般的题目并没有这种需求，因此一般情况下无需调用）。传入的前两个参数是main函数的两个参数，第三个是允许选手提交的程序文件名构成的列表（这无需和Lemon选取选手程序的优先级相同）。  
如果选手程序无法找到，Special Judge程序将直接给出0分并退出。

### testlibInput读入

testlibInput是inf, ouf, ans和source四个全局变量。

testlibInput继承了ifstream，因此所有cin上可用的操作都可以用到testlibInput中。但是，我更建议使用下面的函数（这些函数都是“安全”的，如果读入时遇到意外会直接判选手0分，而自己本身不会出错）

- `char readChar()`  
读入一个字符（读入任何字符，不忽略空格）  
如果调用时已经到达文件末尾，将会给出0分并退出

- `char readChar(char c)`  
将光标移动到字符c之后  
如果调用时已经到达文件末尾，或光标后已经不存在字符c，将给出0分并退出

- `void readSpace()`  
将光标移动到光标后的下一段连续空格字符之后。  
注意此处空格字符指`isspace`为`true`  
如果光标后已经没有空格字符，将会给出0分并退出

- `std::string __readToken()`  
  `std::string readToken()`  
读入光标后下一段连续的非空格字符  
如果光标已经指向非空格字符，那么将从光标开始读，直到碰到空格字符。  
如果光标后已经没有非空格字符，`__readToken`返回空串，而`readToken`给出0分并退出。

- `<class T> T __readLong(int digits, unsigned limitlen, string llimit, string rlimit)`  
内部函数，请勿使用。

- `long long readLong()`  
  `int readInt()`  
读入光标后下一个64或32位整数  
该函数会先执行`__readToken`，随后判断读入的内容是否合法。如果不合法，将给出0分并退出。

- `long long readLong(long long L, long long R)`  
  `int readInt(long long L, long long R)`  
读入光标后下一个64或32位整数，并限制其范围。  
该函数会先执行`readLong()`或`readInt()`，随后判断获得的整数是否在范围内。如果超出，将给出0分并退出。

- `double __readReal(int minPrec, int maxPrec)`  
内部函数，请勿使用。

- `double readReal()`  
  `double readReal(double L, double R)`  
读入光标后下一个实数。其运作原理与`readInt`相同。  
注意目前只支持形如`1.2, 1.188, 3.14158265358979323846264, +1.800, -1.4`的实数，`inf, Infinity, NaN, nan, -1.#IND00, -0.NAN000, 3.245e+19, 9.999e-17`将不被支持。如果发现这种输出，也将给出0分并退出。

- `double readStrictReal(double L, double R, int minPrec, int maxPrec)`  
在`readReal(L,R)`的基础上对小数点后位数进行限制。（不满足限制即0分）

- `string readLine()`  
  `string readReal()`  
读入一行并返回。  
此函数将忽略空行。在已经到达文件末尾时，返回空字符串。

- `void readEoln()`  
将光标移到下一行开头。  
如果已经没有下一行，或者已经到达文件末尾，不会发生任何错误。

- `bool readEof()`  
直接将光标移到文件末尾。  
返回值：当前光标后是否还有非空格字符。

- `bool eof()`  
判断是否到达文件末尾。

### 判分并退出函数

- `quitf(int status, const char *str, arg...)`  
给出判分状态，然后按`printf(str, arg...)`的格式给出评测信息。  
status可以填写：`_ok` (AC)，`_wa` (WA)，`_err` (读入发生意外)  
目前填写`_wa`和`_err`效果相同。  
**注意！如果status是`_ok`，系统会自动调用`ouf.readEof()`。如果返回值为真将给出0分（选手输出中有多于内容）。要避免这种情况，请在给分前调用一次`ouf.readEof()`**

- `quitp(double status, const char *str, arg...)`  
给出部分分状态，然后按`printf(str, arg...)`的格式给出评测信息。  
status可以填写0到1之间的实数（包含0和1）。最后给出的得分是`(int)round(testscore * status)`。  
可以用于给出0分或满分，此时效果与`quitf`相同。  
**注意！如果status大于0，系统会自动调用`ouf.readEof()`。如果返回值为真将给出0分（选手输出中有多于内容）。要避免这种情况，请在给分前调用一次`ouf.readEof()`**

## 配置方式

将编写好的 Special Judge 程序存为 `${比赛文件夹}/data/${题目名称}/checker.cpp`，然后用 `-std=c++11` 选项编译成 `checker.exe`。

在Lemon中选择“自定义校验器”，并填写exe路径，如下图：

![](https://i.loli.net/2019/10/02/fHpg8964LTNeJQm.png)
