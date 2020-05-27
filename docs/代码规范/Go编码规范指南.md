# Go编码规范指南

## 序言

看过很多方面的编码规范，可能每一家公司都有不同的规范，这份编码规范是写给我自己的，同时希望我们公司内部同事也能遵循这个规范来写Go代码。

如果你的代码没有办法找到下面的规范，那么就遵循标准库的规范，多阅读标准库的源码，标准库的代码可以说是我们写代码参考的标杆。

## 格式化规范

go默认已经有了gofmt工具，但是我们强烈建议使用goimport工具，这个在gofmt的基础上增加了自动删除和引入包.

```
go get golang.org/x/tools/cmd/goimports
```

不同的编辑器有不同的配置, sublime的配置教程：http://michaelwhatcott.com/gosublime-goimports/

LiteIDE默认已经支持了goimports，如果你的不支持请点击属性配置->golangfmt->勾选goimports

保存之前自动fmt你的代码。

## 行长约定

一行最长不超过80个字符，超过的请使用换行展示，尽量保持格式优雅。

## go vet

vet工具可以帮我们静态分析我们的源码存在的各种问题，例如多余的代码，提前return的逻辑，struct的tag是否符合标准等。

```
go get golang.org/x/tools/cmd/vet
```

使用如下：

```
go vet .
```

## package名字

保持package的名字和目录保持一致，尽量采取有意义的包名，简短，有意义，尽量和标准库不要冲突。

## import 规范

import在多行的情况下，goimports会自动帮你格式化，但是我们这里还是规范一下import的一些规范，如果你在一个文件里面引入了一个package，还是建议采用如下格式：

```
import (
    "fmt"
)
```

如果你的包引入了三种类型的包，标准库包，程序内部包，第三方包，建议采用如下方式进行组织你的包：

```
import (
    "encoding/json"
    "strings"

    "myproject/models"
    "myproject/controller"
    "myproject/utils"

    "github.com/astaxie/beego"
    "github.com/go-sql-driver/mysql"
)   
```

有顺序的引入包，不同的类型采用空格分离，第一种实标准库，第二是项目包，第三是第三方包。

在项目中不要使用相对路径引入包：

```
// 这是不好的导入
import “../net”

// 这是正确的做法
import “github.com/repo/proj/src/net”
```

## 变量申明

变量名采用驼峰标准，不要使用`_`来命名变量名，多个变量申明放在一起

```
var (
    Found bool
    count int
)
```

在函数外部申明必须使用var,不要采用`:=`，容易踩到变量的作用域的问题。

## 自定义类型的string循环问题

如果自定义的类型定义了String方法，那么在打印的时候会产生隐藏的一些bug

```
type MyInt int
func (m MyInt) String() string { 
    return fmt.Sprint(m)   //BUG:死循环
}

func(m MyInt) String() string { 
    return fmt.Sprint(int(m))   //这是安全的,因为我们内部进行了类型转换
}
```

## 避免返回命名的参数

如果你的函数很短小，少于10行代码，那么可以使用，不然请直接使用类型，因为如果使用命名变量很
容易引起隐藏的bug

```
func Foo(a int, b int) (string, ok){

}
```

当然如果是有多个相同类型的参数返回，那么命名参数可能更清晰：

```
func (f *Foo) Location() (float64, float64, error)
```

下面的代码就更清晰了：

```
// Location returns f's latitude and longitude.
// Negative values mean south and west, respectively.
func (f *Foo) Location() (lat, long float64, err error)
```

## 错误处理

错误处理的原则就是不能丢弃任何有返回err的调用，不要采用`_`丢弃，必须全部处理。接收到错误，要么返回err，要么实在不行就panic，或者使用log记录下来

### error 信息

error的信息不要采用大写字母，尽量保持你的错误简短，但是要足够表达你的错误的意思。

## 长句子打印或者调用，使用参数进行格式化分行

我们在调用`fmt.Sprint`或者`log.Sprint`之类的函数时，有时候会遇到很长的句子，我们需要在参数调用处进行多行分割：

下面是错误的方式：

```
log.Printf(“A long format string: %s %d %d %s”, myStringParameter, len(a),
    expected.Size, defrobnicate(“Anotherlongstringparameter”,
        expected.Growth.Nanoseconds() /1e6))
```

应该是如下的方式：

```
log.Printf( 
    “A long format string: %s %d %d %s”, 
    myStringParameter,
    len(a),
    expected.Size,
    defrobnicate(
        “Anotherlongstringparameter”,
        expected.Growth.Nanoseconds()/1e6, 
    ),
）   
```

## 注意闭包的调用

在循环中调用函数或者goroutine方法，一定要采用显示的变量调用，不要再闭包函数里面调用循环的参数

```
fori:=0;i<limit;i++{
    go func(){ DoSomething(i) }() //错误的做法
    go func(i int){ DoSomething(i) }(i)//正确的做法
￼}
```

http://golang.org/doc/articles/race_detector.html#Race_on_loop_counter

## 在逻辑处理中禁用panic

在main包中只有当实在不可运行的情况采用panic，例如文件无法打开，数据库无法连接导致程序无法
正常运行，但是对于其他的package对外的接口不能有panic，只能在包内采用。

强烈建议在main包中使用log.Fatal来记录错误，这样就可以由log来结束程序。

## 注释规范

注释可以帮我们很好的完成文档的工作，写得好的注释可以方便我们以后的维护。详细的如何写注释可以
参考：http://golang.org/doc/effective_go.html#commentary

### bug注释

针对代码中出现的bug，可以采用如下教程使用特殊的注释，在godocs可以做到注释高亮：

```
// BUG(astaxie):This divides by zero. 
var i float = 1/0
```

http://blog.golang.org/2011/03/godoc­documenting­go­code.html

## struct规范

### struct申明和初始化格式采用多行：

定义如下：

```
type User struct{
    Username  string
    Email     string
}
```

初始化如下：

```
u := User{
    Username: "astaxie",
    Email:    "astaxie@gmail.com",
}
```

### recieved是值类型还是指针类型

到底是采用值类型还是指针类型主要参考如下原则：

```
func(w Win) Tally(playerPlayer)int    //w不会有任何改变 
func(w *Win) Tally(playerPlayer)int    //w会改变数据
```

更多的请参考：https://code.google.com/p/go-wiki/wiki/CodeReviewComments#Receiver_Type

### 带mutex的struct必须是指针receivers

如果你定义的struct中带有mutex,那么你的receivers必须是指针

## ------------------------------------------------------

## Golang编码规范

### - gofmt

大部分的格式问题可以通过gofmt解决，gofmt自动格式化代码，保证所有的go代码与官方推荐的格式保持一致，于是所有格式有关问题，都以gofmt的结果为准。

### - 行长

一行最长不超过80个字符，超过的使用换行展示，尽量保持格式优雅。

### - 注释

在编码阶段应该同步写好变量、函数、包的注释，最后可以利用godoc导出文档。注释必须是完整的句子，句子的结尾应该用句号作为结尾（英文句号）。注释推荐用英文，可以在写代码过程中锻炼英文的阅读和书写能力。而且用英文不会出现各种编码的问题。
每个包都应该有一个包注释，一个位于package子句之前的块注释或行注释。包如果有多个go文件，只需要出现在一个go文件中即可。



```go
// ping包实现了常用的ping相关的函数
package ping
```

导出函数注释，第一条语句应该为一条概括语句，并且使用被声明的名字作为开头。



```go
// 求a和b的和，返回sum。
func Myfunction(sum int) (a, b int) {
```

### - 命名
  - 需要注释来补充的命名就不算是好命名。
  - 使用可搜索的名称：单字母名称和数字常量很难从一大堆文字中搜索出来。单字母名称仅适用于短方法中的本地变量，名称长短应与其作用域相对应。若变量或常量可能在代码中多处使用，则应赋其以便于搜索的名称。
  - 做有意义的区分：Product和ProductInfo和ProductData没有区别，NameString和Name没有区别，要区分名称，就要以读者能鉴别不同之处的方式来区分 。
  - 函数命名规则：驼峰式命名，名字可以长但是得把功能，必要的参数描述清楚，函数名名应当是动词或动词短语，如postPayment、deletePage、save。并依Javabean标准加上get、set、is前缀。例如：xxx + With + 需要的参数名 + And + 需要的参数名 + .....
  - 结构体命名规则：结构体名应该是名词或名词短语，如Custome、WikiPage、Account、AddressParser，避免使用Manager、Processor、Data、Info、这样的类名，类名不应当是动词。
  - 包名命名规则：包名应该为小写单词，不要使用下划线或者混合大小写。
  - 接口命名规则：单个函数的接口名以"er"作为后缀，如Reader,Writer。接口的实现则去掉“er”。



```csharp
type Reader interface {
        Read(p []byte) (n int, err error)
}
```

两个函数的接口名综合两个函数名



```csharp
type WriteFlusher interface {
    Write([]byte) (int, error)
    Flush() error
}
```

三个以上函数的接口名，抽象这个接口的功能，类似于结构体名



```csharp
type Car interface {
    Start([]byte)
    Stop() error
    Recover()
}
```

### - 常量

常量均需使用全部大写字母组成，并使用下划线分词：



```cpp
const APP_VER = "1.0"
```

如果是枚举类型的常量，需要先创建相应类型：

```go
type Scheme string

const (
    HTTP  Scheme = "http"
    HTTPS Scheme = "https"
)
```

如果模块的功能较为复杂、常量名称容易混淆的情况下，为了更好地区分枚举类型，可以使用完整的前缀：

```go
type PullRequestStatus int

const (
    PULL_REQUEST_STATUS_CONFLICT PullRequestStatus = iota
    PULL_REQUEST_STATUS_CHECKING
    PULL_REQUEST_STATUS_MERGEABLE
)
```

### - 变量

变量命名基本上遵循相应的英文表达或简写,在相对简单的环境（对象数量少、针对性强）中，可以将一些名称由完整单词简写为单个字母，例如：
\* user 可以简写为 u
\* userID 可以简写 uid
若变量类型为 bool 类型，则名称应以 Has, Is, Can 或 Allow 开头：

```csharp
var isExist bool
var hasConflict bool
var canManage bool
var allowGitHook bool
```

### - 变量命名惯例

变量名称一般遵循驼峰法，但遇到特有名词时，需要遵循以下规则：

```undefined
* 如果变量为私有，且特有名词为首个单词，则使用小写，如 apiClient
* 其它情况都应当使用该名词原有的写法，如 APIClient、repoID、UserID
* 错误示例：UrlArray，应该写成urlArray或者URLArray
```

下面列举了一些常见的特有名词：

```php
// A GonicMapper that contains a list of common initialisms taken from golang/lint
var LintGonicMapper = GonicMapper{
    "API":   true,
    "ASCII": true,
    "CPU":   true,
    "CSS":   true,
    "DNS":   true,
    "EOF":   true,
    "GUID":  true,
    "HTML":  true,
    "HTTP":  true,
    "HTTPS": true,
    "ID":    true,
    "IP":    true,
    "JSON":  true,
    "LHS":   true,
    "QPS":   true,
    "RAM":   true,
    "RHS":   true,
    "RPC":   true,
    "SLA":   true,
    "SMTP":  true,
    "SSH":   true,
    "TLS":   true,
    "TTL":   true,
    "UI":    true,
    "UID":   true,
    "UUID":  true,
    "URI":   true,
    "URL":   true,
    "UTF8":  true,
    "VM":    true,
    "XML":   true,
    "XSRF":  true,
    "XSS":   true,
}
```

### - struct规范

struct申明和初始化格式采用多行：

定义如下：

```cpp
type User struct{
    Username  string
    Email     string
}
```

初始化如下：

```go
u := User{
    Username: "test",
    Email:    "test@gmail.com",
}
```

- 控制结构

if
if接受初始化语句，约定如下方式建立局部变量

```go
if err := file.Chmod(0664); err != nil {
    return err
}
```

for
采用短声明建立局部变量

```go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

return
尽早return：一旦有错误发生，马上返回

```go
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.Stat()
if err != nil {
    f.Close()
    return err
}
codeUsing(f, d)
```

### - 错误处理
  - error作为函数的值返回,必须对error进行处理
  - 错误描述如果是英文必须为小写，不需要标点结尾
  - 采用独立的错误流进行处理

不要采用下面的处理错误写法

```go
    if err != nil {
        // error handling
    } else {
        // normal code
    }
```

采用下面的写法

```go
    if err != nil {
        // error handling
        return // or continue, etc.
    }
    // normal code
```

使用函数的返回值时，则采用下面的方式

```go
x, err := f()
if err != nil {
    // error handling
    return
}
// use x
```

### - panic

尽量不要使用panic，除非你知道你在做什么

### - import

对import的包进行分组管理，用换行符分割，而且标准库作为分组的第一组。如果你的包引入了三种类型的包，标准库包，程序内部包，第三方包，建议采用如下方式进行组织你的包

```go
package main

import (
    "fmt"
    "os"

    "kmg/a"
    "kmg/b"

    "code.google.com/a"
    "github.com/b"
)
```

在项目中不要使用相对路径引入包：



```swift
// 错误示例
import “../net”

// 正确的做法
import “github.com/repo/proj/src/net”
```

goimports会自动帮你格式化

### - 参数传递
  - 对于少量数据，不要传递指针
  - 对于大量数据的struct可以考虑使用指针
  - 传入参数是map，slice，chan不要传递指针，因为map，slice，chan是引用类型，不需要传递指针的指针
- 单元测试

单元测试文件名命名规范为 example_test.go
测试用例的函数名称必须以 Test 开头，例如：TestExample



## 如何写出优雅的 Golang 代码

链接：https://www.jianshu.com/p/596eaebd6e54



代码规范：使用辅助工具帮助我们在每次提交 PR 时自动化地对代码进行检查，减少工程师人工审查的工作量；

最佳实践

- 目录结构：遵循 Go 语言社区中被广泛达成共识的 [目录结构](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fgolang-standards%2Fproject-layout)，减少项目的沟通成本；
- 模块拆分：按照职责对不同的模块进行拆分，Go 语言的项目中也不应该出现 `model`、`controller` 这种违反语言顶层设计思路的包名；
- 显示与隐式：尽可能地消灭项目中的 `init` 函数，保证显式地进行方法的调用以及错误的处理；
- 面向接口：面向接口是 Go 语言鼓励的开发方式，也能够为我们写单元测试提供方便，我们应该遵循固定的模式对外提供功能；
  1. 使用大写的 `Service` 对外暴露方法；
  2. 使用小写的 `service` 实现接口中定义的方法；
  3. 通过 `func NewService(...) (Service, error)` 函数初始化 `Service` 接口；

单元测试：保证项目工程质量的最有效办法；

- 可测试：意味着面向接口编程以及减少单个函数中包含的逻辑，使用『小方法』；
- 组织方式：使用 Go 语言默认的 Test 框架、开源的 `suite` 或者 BDD 的风格对单元测试进行合理组织；
- Mock 方法：四种不同的单元测试 Mock 方法；
  - [gomock](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fgolang%2Fmock)：最标准的也是最被鼓励的方式；
  - [sqlmock](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FDATA-DOG%2Fgo-sqlmock)：处理依赖的数据库；
  - [httpmock](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fjarcoal%2Fhttpmock)：处理依赖的 HTTP 请求；
  - [monkey](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fbouk%2Fmonkey)：万能的方法，但是只在万不得已时使用，类似的代码写起来非常冗长而且不直观；
- 断言：使用社区的 [testify](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fstretchr%2Ftestify) 快速验证方法的返回值；



### go代码规范链接：

[https://blog.csdn.net/winter_wu_1998/article/details/102926479](https://blog.csdn.net/winter_wu_1998/article/details/102926479)



[跳到内容](https://github.com/cristaloleg/go-advice#start-of-content)



![img](https://github.githubassets.com/images/search-key-slash.svg)

[拉取要求](https://github.com/pulls)[问题](https://github.com/issues)

[市场](https://github.com/marketplace)

[探索](https://github.com/explore)



 

![x](https://avatars3.githubusercontent.com/u/15353707?s=60&v=4) 



## 无需任何代码即可学习Git和GitHub！

使用Hello World指南，您将开始一个分支，编写注释并打开拉取请求。

[阅读指南](https://guides.github.com/activities/hello-world/)



# [ista脚](https://github.com/cristaloleg)/**[去咨询](https://github.com/cristaloleg/go-advice)**

-  看 [68](https://github.com/cristaloleg/go-advice/watchers)
-  取消星标[1.5千](https://github.com/cristaloleg/go-advice/stargazers)
-  叉子[98](https://github.com/cristaloleg/go-advice/network/members)

- 

   码

- 

   问题 0

- 

   拉取要求 0

- 

   动作

- 

   专案 0

- 

   维基

- 

   安全 0

- 

   见解

Go advice的建议和技巧列表

[高朗](https://github.com/topics/golang)[走](https://github.com/topics/go)[忠告](https://github.com/topics/advice)[很棒](https://github.com/topics/awesome)[指南](https://github.com/topics/guide)[指导方针](https://github.com/topics/guidelines)[经验](https://github.com/topics/experience)

- [ 160次 提交](https://github.com/cristaloleg/go-advice/commits/master)
- [ 1个 分支](https://github.com/cristaloleg/go-advice/branches)
- [ 0 包](https://github.com/cristaloleg/go-advice/packages)
- [ 0 发布](https://github.com/cristaloleg/go-advice/releases)
- [ 16位 贡献者](https://github.com/cristaloleg/go-advice/graphs/contributors)
- [ 无执照](https://github.com/cristaloleg/go-advice/blob/master/LICENSE)

<details class="details-reset" style="box-sizing: border-box; display: block;"><summary title="点击查看语言详情" data-ga-click="Repository, language bar stats toggle, location:repo overview" style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none;"><div class="d-flex repository-lang-stats-graph" style="box-sizing: border-box; display: flex !important; width: 980px; overflow: hidden; white-space: nowrap; cursor: pointer; user-select: none; border-width: 0px 1px 1px; border-right-style: solid; border-bottom-style: solid; border-left-style: solid; border-right-color: rgb(221, 221, 221); border-bottom-color: rgb(221, 221, 221); border-left-color: rgb(221, 221, 221); border-image: initial; border-top-style: initial; border-top-color: initial; border-bottom-right-radius: 3px; border-bottom-left-radius: 3px;"><span class="language-color" aria-label="升至100.0％" itemprop="keywords" style="box-sizing: border-box; line-height: 8px; text-indent: -9999px; border-bottom-left-radius: 3px; border-bottom-right-radius: 3px; width: 978px; background-color: rgb(0, 173, 216);"><font style="box-sizing: border-box; vertical-align: inherit;"><font style="box-sizing: border-box; vertical-align: inherit;">走</font></font></span></div></summary></details>

<details class="details-reset details-overlay branch-select-menu " id="branch-select-menu" style="box-sizing: border-box; display: block;"><summary class="btn css-truncate btn-sm" data-hotkey="w" title="切换分支或标签" aria-haspopup="menu" role="button" style="box-sizing: border-box; display: inline-block; cursor: pointer; position: relative; padding: 3px 10px; font-size: 12px; font-weight: 600; line-height: 20px; white-space: nowrap; vertical-align: middle; user-select: none; background-repeat: repeat-x; background-position: -1px -1px; background-size: 110% 110%; border: 1px solid rgba(27, 31, 35, 0.2); border-radius: 0.25em; -webkit-appearance: none; color: rgb(36, 41, 46); background-color: rgb(239, 243, 246); background-image: linear-gradient(-180deg, rgb(250, 251, 252), rgb(239, 243, 246) 90%); list-style: none;"><i style="box-sizing: border-box; font-style: normal; font-weight: 500; opacity: 0.75;"><font style="box-sizing: border-box; vertical-align: inherit;"><font style="box-sizing: border-box; vertical-align: inherit;">科：</font></font></i><span>&nbsp;</span><span class="css-truncate-target" data-menu-button="" style="box-sizing: border-box; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; display: inline-block; max-width: 125px; vertical-align: top;"><font style="box-sizing: border-box; vertical-align: inherit;"><font style="box-sizing: border-box; vertical-align: inherit;">主</font></font></span><span>&nbsp;</span><span class="dropdown-caret" style="box-sizing: border-box; display: inline-block; width: 0px; height: 0px; vertical-align: middle; content: &quot;&quot;; border-style: solid; border-width: 4px 4px 0px; border-right-color: transparent; border-bottom-color: transparent; border-left-color: transparent;"></span></summary></details>

[新的拉取请求](https://github.com/cristaloleg/go-advice/pull/new/master)

建立新档案[上传文件](https://github.com/cristaloleg/go-advice/upload/master)[查找文件](https://github.com/cristaloleg/go-advice/find/master)

克隆或下载 

## 最新提交

[![yang](https://avatars1.githubusercontent.com/u/1710912?s=60&u=e83b54945e0289e43a17e9b7422dd71fbd7b71fa&v=4)](https://github.com/yangwenmai)

[yangwenmai ](https://github.com/cristaloleg/go-advice/commits?author=yangwenmai)[文档：更新zh（](https://github.com/cristaloleg/go-advice/commit/49798ebacb18acfc70f240bf8609a227f8ac2622)[＃49 ](https://github.com/cristaloleg/go-advice/pull/49)[）](https://github.com/cristaloleg/go-advice/commit/49798ebacb18acfc70f240bf8609a227f8ac2622) …

最新提交[49798eb](https://github.com/cristaloleg/go-advice/commit/49798ebacb18acfc70f240bf8609a227f8ac2622)2月25日

## 档案

| 类型 | 名称                                                         | 最新提交消息                                                 | 提交时间 |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ | -------- |
|      | [执照](https://github.com/cristaloleg/go-advice/blob/master/LICENSE) | [更新许可](https://github.com/cristaloleg/go-advice/commit/815c6095d5f6f1e59462a18385df6e1a2d4f2ff4) | 14个月前 |
|      | [自述文件](https://github.com/cristaloleg/go-advice/blob/master/README.md) | [关于未加密的文字（](https://github.com/cristaloleg/go-advice/commit/9c45c166444dd44ecc9f2fdb1157a9ff526bded2)[＃48 ](https://github.com/cristaloleg/go-advice/pull/48)[）](https://github.com/cristaloleg/go-advice/commit/9c45c166444dd44ecc9f2fdb1157a9ff526bded2) | 3个月前  |
|      | [README_KR.md](https://github.com/cristaloleg/go-advice/blob/master/README_KR.md) | [添加/韩语翻译（](https://github.com/cristaloleg/go-advice/commit/4b4f3fa2eb010d6aebd717511642608d6212eb20)[＃47 ](https://github.com/cristaloleg/go-advice/pull/47)[）](https://github.com/cristaloleg/go-advice/commit/4b4f3fa2eb010d6aebd717511642608d6212eb20) | 3个月前  |
|      | [README_ZH.md](https://github.com/cristaloleg/go-advice/blob/master/README_ZH.md) | [docs：更新zh（](https://github.com/cristaloleg/go-advice/commit/49798ebacb18acfc70f240bf8609a227f8ac2622)[＃49 ](https://github.com/cristaloleg/go-advice/pull/49)[）](https://github.com/cristaloleg/go-advice/commit/49798ebacb18acfc70f240bf8609a227f8ac2622) | 3个月前  |
|      | [你好](https://github.com/cristaloleg/go-advice/blob/master/hello.go) | [更新hello.go](https://github.com/cristaloleg/go-advice/commit/aeb5844ff2b5e16315ada322dad012c7cda008b5) | 17个月前 |

##  自述文件



## 内容

- [谚语](https://github.com/cristaloleg/go-advice#go-proverbs)
- [围棋禅](https://github.com/cristaloleg/go-advice#the-zen-of-go)
- [码](https://github.com/cristaloleg/go-advice#code)
- [并发](https://github.com/cristaloleg/go-advice#concurrency)
- [性能](https://github.com/cristaloleg/go-advice#performance)
- [模组](https://github.com/cristaloleg/go-advice#modules)
- [建立](https://github.com/cristaloleg/go-advice#build)
- [测试中](https://github.com/cristaloleg/go-advice#testing)
- [工具类](https://github.com/cristaloleg/go-advice#tools)
- [杂项](https://github.com/cristaloleg/go-advice#misc)

### 谚语

- 不要通过共享内存进行通信，而要通过通信共享内存。
- 并发不是并行性。
- 渠道编排；互斥锁序列化。
- 接口越大，抽象性越弱。
- 使零值有用。
- `interface{}` 什么也没说。
- Gofmt的风格不是每个人的最爱，但gofmt是每个人的最爱。
- 稍微复制胜于一点依赖。
- Syscall必须始终使用构建标记来保护。
- Cgo必须始终使用build标签来保护。
- Cgo不是Go。
- 对于不安全的包装，无法保证。
- 清晰胜于聪明。
- 反思从未明确。
- 错误是价值。
- 不要仅仅检查错误，而要优雅地处理它们。
- 设计架构，命名组件，记录细节。
- 文档供用户使用。
- 不要惊慌

作者：Rob Pike查看更多：[https](https://go-proverbs.github.io/) : [//go-proverbs.github.io/](https://go-proverbs.github.io/)

### 围棋禅

- 每个包装实现一个目的
- 明确处理错误
- 早日返回，而不是深入巢穴
- 并发给调用者
- 在启动goroutine之前，请知道它何时会停止
- 避免包装级别状态
- 简单很重要
- 编写测试以锁定包API的行为
- 如果您认为速度缓慢，请先通过基准测试进行验证
- 节制是一种美德
- 可维护性计数

作者：Dave Cheney查看更多：[https](https://the-zen-of-go.netlify.com/) : [//the-zen-of-go.netlify.com/](https://the-zen-of-go.netlify.com/)

### 码

#### 始终是`go fmt`您的代码。

社区使用官方的Go格式，不要重新发明轮子。

尝试减少代码熵。这将帮助所有人使代码易于阅读。

#### 多个if-else语句可以折叠到一个开关中

```
//不差，
如果 foo（）{
     // ... 
} 否则， 如果 bar  ==  baz {
     // ... 
} else {
     // ... 
} //更好地切换 {
 case foo（）：
     // .. 。情况吧== 巴兹：
     // ... 默认：
     // ... 
}


 
   
```

#### 传递信号`chan struct{}`而不是`chan bool`。

当您看到`chan bool`结构中的定义时，有时不容易理解如何使用该值，例如：

```
输入 Service  struct {
     deleteCh  chan  bool  //这是什么意思？
}
```

但是，我们可以通过将其更改为`chan struct{}`明确表示的内容来使其更加清晰：我们不在乎值（始终为`struct{}`），我们在乎可能发生的事件，例如：

```
键入 Service  struct {
     deleteCh  chan  struct {} //好的，如果事件比删除一些东西的话。
}
```

#### 更喜欢`30 * time.Second`而不是`time.Duration(30) * time.Second`

您不需要将无类型的const包装在类型中，编译器会找出来。还喜欢将const移到第一位：

```
// BAD
delay := time.Second * 60 * 24 * 60

// VERY BAD
delay := 60 * time.Second * 60 * 24

// GOOD
delay := 24 * 60 * 60 * time.Second
```

#### Use `time.Duration` instead of `int64` + variable name

```
// BAD
var delayMillis int64 = 15000

// GOOD
var delay time.Duration = 15 * time.Second
```

#### Group `const` declarations by type and `var` by logic and/or type

```
// BAD
const (
    foo = 1
    bar = 2
    message = "warn message"
)

// MOSTLY BAD
const foo = 1
const bar = 2
const message = "warn message"

// GOOD
const (
    foo = 1
    bar = 2
)

const message = "warn message"
```

This pattern works for `var` too.

-  every blocking or IO function call should be cancelable or at least timeoutable

-  

  implement

   

  ```
  Stringer
  ```

   

  interface for integers const values

  - https://godoc.org/golang.org/x/tools/cmd/stringer

-  check your defer's error

```
  defer func() {
      err := ocp.Close()
      if err != nil {
          rerr = err
      }
  }()
```

-  don't use `checkErr` function which panics or does `os.Exit`

-  use panic only in very specific situations, you have to handle error

-  

  don't use alias for enums 'cause this breaks type safety

  - https://play.golang.org/p/MGbeDwtXN3

```
  package main
  type Status = int
  type Format = int // remove `=` to have type safety

  const A Status = 1
  const B Format = 1

  func main() {
	println(A == B)
  }
```

-  

  if you're going to omit returning params, do it explicitly

  - so prefer this `_ = f()` to this `f()`

-  the short form for slice initialization is `a := []T{}`

-  

  iterate over array or slice using range loop

  - instead of `for i := 3; i < 7; i++ {...}` prefer `for _, c := range a[3:7] {...}`

-  use backquote(`) for multiline strings

-  skip unused param with _

```
  func f(a int, _ string) {}
```

-  If you are comparing timestamps, use `time.Before` or `time.After`. Don't use `time.Sub` to get a duration and then check its value.
-  always pass context as a first param to a func with a `ctx` name
-  few params of the same type can be defined in a short way

```
  func f(a int, b int, s string, p string)
  func f(a, b int, s, p string)
```

-  

  the zero value of a slice is nil

  - https://play.golang.org/p/pNT0d_Bunq

  ```
    var s []int
    fmt.Println(s, len(s), cap(s))
    if s == nil {
      fmt.Println("nil!")
    }
    // Output:
    // [] 0 0
    // nil!
  ```

  - https://play.golang.org/p/meTInNyxtk

```
  var a []string
  b := []string{}

  fmt.Println(reflect.DeepEqual(a, []string{}))
  fmt.Println(reflect.DeepEqual(b, []string{}))
  // Output:
  // false
  // true
```

-  

  do not compare enum types with

   

  ```
  <
  ```

  ,

   

  ```
  >
  ```

  ,

   

  ```
  <=
  ```

   

  and

   

  ```
  >=
  ```

  - use explicit values, don't do this:

```
  value := reflect.ValueOf(object)
  kind := value.Kind()
  if kind >= reflect.Chan && kind <= reflect.Slice {
    // ...
  }
```

-  use `%+v` to print data with sufficient details

-  

  be careful with empty struct

   

  ```
  struct{}
  ```

  , see issue:

   

  https://github.com/golang/go/issues/23440

  - more: https://play.golang.org/p/9C0puRUstrP

```
  func f1() {
    var a, b struct{}
    print(&a, "\n", &b, "\n") // Prints same address
    fmt.Println(&a == &b)     // Comparison returns false
  }

  func f2() {
    var a, b struct{}
    fmt.Printf("%p\n%p\n", &a, &b) // Again, same address
    fmt.Println(&a == &b)          // ...but the comparison returns true
  }
```

-  

  wrap errors with

   

  http://github.com/pkg/errors

  - so: `errors.Wrap(err, "additional message to a given error")`

-  

  be careful with

   

  ```
  range
  ```

   

  in Go:

  - `for i := range a` and `for i, v := range &a` doesn't make a copy of `a`
  - but `for i, v := range a` does
  - more: https://play.golang.org/p/4b181zkB1O

-  

  reading nonexistent key from map will not panic

  - `value := map["no_key"]` will be zero value
  - `value, ok := map["no_key"]` is much better

-  

  do not use raw params for file operation

  - instead of an octal parameter like `os.MkdirAll(root, 0700)`
  - use predefined constants of this type `os.FileMode`

-  

  don't forget to specify a type for

   

  ```
  iota
  ```

  - https://play.golang.org/p/mZZdMaI92cI

```
  const (
    _ = iota
    testvar         // will be int
  )
```

vs

```
  type myType int
  const (
    _ myType = iota
    testvar         // will be myType
  )
```

#### Don’t use `encoding/gob` on structs you don’t own.

At some point structure may change and you might miss this. As a result this might cause a hard to find bug.

#### Don't depend on the evaluation order, especially in a return statement.

```
// BAD
return res, json.Unmarshal(b, &res)

// GOOD
err := json.Unmarshal(b, &res)
return res, err
```

#### To prevent unkeyed literals add `_ struct{}` field:

```
type Point struct {
	X, Y float64
	_    struct{} // to prevent unkeyed literals
}
```

For `Point{X: 1, Y: 1}` everything will be fine, but for `Point{1,1}` you will get a compile error:

```
./file.go:1:11: too few values in Point literal
```

There is a check in `go vet` command for this, there is no enough arguments to add `_ struct{}` in all your structs.

#### To prevent structs comparison add an empty field of `func` type

```
type Point struct {
	_ [0]func()	// unexported, zero-width non-comparable field
	X, Y float64
}
```

#### Prefer `http.HandlerFunc` over `http.Handler`

To use the 1st one you just need a func, for the 2nd you need a type.

#### Move `defer` to the top

This improves code readability and makes clear what will be invoked at the end of a function.

#### JavaScript parses integers as floats and your int64 might overflow.

Use `json:"id,string"` instead.

```
type Request struct {
	ID int64 `json:"id,string"`
}
```

### Concurrency

-  

  best candidate to make something once in a thread-safe way is

   

  ```
  sync.Once
  ```

  - don't use flags, mutexes, channels or atomics

-  to block forever use `select{}`, omit channels, waiting for a signal

-  

  don't close in-channel, this is a responsibility of it's creator

  - writing to a closed channel will cause a panic

-  

  ```
  func NewSource(seed int64) Source
  ```

   

  in

   

  ```
  math/rand
  ```

   

  is not concurrency-safe. The default

   

  ```
  lockedSource
  ```

   

  is concurrency-safe, see issue:

   

  https://github.com/golang/go/issues/3611

  - more: https://golang.org/pkg/math/rand/

-  when you need an atomic value of a custom type use [atomic.Value](https://godoc.org/sync/atomic#Value)

### Performance

-  

  do not omit

   

  ```
  defer
  ```

  - 200ns speedup is negligible in most cases

-  

  always close http body aka

   

  ```
  defer r.Body.Close()
  ```

  - unless you need leaked goroutine

-  filtering without allocating

```
    b := a[:0]
    for _, x := range a {
    	if f(x) {
		    b = append(b, x)
    	}
    }
```

#### To help compiler to remove bound checks see this pattern `_ = b[7]`

-  

  ```
  time.Time
  ```

   

  has pointer field

   

  ```
  time.Location
  ```

   

  and this is bad for go GC

  - it's relevant only for big number of `time.Time`, use timestamp instead

-  

  prefer

   

  ```
  regexp.MustCompile
  ```

   

  instead of

   

  ```
  regexp.Compile
  ```

  - in most cases your regex is immutable, so init it in `func init`

-  

  do not overuse

   

  ```
  fmt.Sprintf
  ```

   

  in your hot path. It is costly due to maintaining the buffer pool and dynamic dispatches for interfaces.

  - if you are doing `fmt.Sprintf("%s%s", var1, var2)`, consider simple string concatenation.
  - if you are doing `fmt.Sprintf("%x", var)`, consider using `hex.EncodeToString` or `strconv.FormatInt(var, 16)`

-  

  always discard body e.g.

   

  ```
  io.Copy(ioutil.Discard, resp.Body)
  ```

   

  if you don't use it

  - HTTP client's Transport will not reuse connections unless the body is read to completion and closed

```
    res, _ := client.Do(req)
    io.Copy(ioutil.Discard, res.Body)
    defer res.Body.Close()
```

-  

  don't use defer in a loop or you'll get a small memory leak

  - 'cause defers will grow your stack without the reason

-  don't forget to stop ticker, unless you need a leaked channel

```
  ticker := time.NewTicker(1 * time.Second)
  defer ticker.Stop()
```

-  

  use custom marshaler to speed up marshaling

  - but before using it - profile! ex: https://play.golang.org/p/SEm9Hvsi0r

```
  func (entry Entry) MarshalJSON() ([]byte, error) {
	buffer := bytes.NewBufferString("{")
	first := true
	for key, value := range entry {
		jsonValue, err := json.Marshal(value)
		if err != nil {
			return nil, err
		}
		if !first {
			buffer.WriteString(",")
		}
		first = false
		buffer.WriteString(key + ":" + string(jsonValue))
	}
	buffer.WriteString("}")
	return buffer.Bytes(), nil
  }
```

-  `sync.Map` isn't a silver bullet, do not use it without a strong reasons
  - more: https://github.com/golang/go/blob/master/src/sync/map.go#L12
-  storing non-pointer values in `sync.Pool` allocates memory
  - more: https://github.com/dominikh/go-tools/blob/master/cmd/staticcheck/docs/checks/SA6002
-  to hide a pointer from escape analysis you might carefully(!!!) use this func:
  - source: https://go-review.googlesource.com/c/go/+/86976

```
  // noescape hides a pointer from escape analysis.  noescape is
  // the identity function but escape analysis doesn't think the
  // output depends on the input. noescape is inlined and currently
  // compiles down to zero instructions.
  func noescape(p unsafe.Pointer) unsafe.Pointer {
  	x := uintptr(p)
  	return unsafe.Pointer(x ^ 0)
  }
```

-  for fastest atomic swap you might use this `m := (*map[int]int)(atomic.LoadPointer(&ptr))`

-  

  use buffered I/O if you do many sequential reads or writes

  - to reduce number of syscalls

-  

  there are 2 ways to clear a map:

  - reuse map memory

```
	for k := range m {
		delete(m, k)
	}
```

- allocate new

```
	m = make(map[int]int)
```

### Modules

-  if you want to test that `go.mod` (and `go.sum`) is up to date in CI https://blog.urth.org/2019/08/13/testing-go-mod-tidiness-in-ci/

### Build

-  strip your binaries with this command `go build -ldflags="-s -w" ...`

-  

  easy way to split test into different builds

  - use `// +build integration` and run them with `go test -v --tags integration .`

-  

  tiniest Go docker image

  - https://twitter.com/bbrodriges/status/873414658178396160
  - `CGO_ENABLED=0 go build -ldflags="-s -w" app.go && tar C app | docker import - myimage:latest`

-  

  run

   

  ```
  go format
  ```

   

  on CI and compare diff

  - this will ensure that everything was generated and committed

-  

  to run Travis-CI with the latest Go use

   

  ```
  travis 1
  ```

  - see more: https://github.com/travis-ci/travis-build/blob/master/public/version-aliases/go.json

-  check if there are mistakes in code formatting `diff -u <(echo -n) <(gofmt -d .)`

### Testing

-  prefer `package_test` name for tests, rather than `package`
-  `go test -short` allows to reduce set of tests to be runned

```
  func TestSomething(t *testing.T) {
    if testing.Short() {
      t.Skip("skipping test in short mode.")
    }
  }
```

-  skip test depending on architecture

```
  if runtime.GOARM == "arm" {
    t.Skip("this doesn't work under ARM")
  }
```

-  

  track your allocations with

   

  ```
  testing.AllocsPerRun
  ```

  - https://godoc.org/testing#AllocsPerRun

-  

  run your benchmarks multiple times, to get rid of noise

  - `go test -test.bench=. -count=20`

### Tools

-  quick replace `gofmt -w -l -r "panic(err) -> log.Error(err)" .`

-  

  ```
  go list
  ```

   

  allows to find all direct and transitive dependencies

  - `go list -f '{{ .Imports }}' package`
  - `go list -f '{{ .Deps }}' package`

-  

  for fast benchmark comparison we've a

   

  ```
  benchstat
  ```

   

  tool

  - https://godoc.org/golang.org/x/perf/cmd/benchstat

-  [go-critic](https://github.com/go-critic/go-critic) linter enforces several advices from this document

-  `go mod why -m ` tells us why a particular module is in the `go.mod` file

-  `GOGC=off go build ...` should speed up your builds [source](https://twitter.com/mvdan_/status/1107579946501853191)

-  

  The memory profiler records one allocation every 512Kbytes. You can increase the rate via the

   

  ```
  GODEBUG
  ```

   

  environment variable to see more details in your profile.

  - by https://twitter.com/bboreham/status/1105036740253937664

### Misc

-  dump goroutines https://stackoverflow.com/a/27398062/433041

```
  go func() {
    sigs := make(chan os.Signal, 1)
    signal.Notify(sigs, syscall.SIGQUIT)
    buf := make([]byte, 1<<20)
    for {
      <-sigs
      stacklen := runtime.Stack(buf, true)
      log.Printf("=== received SIGQUIT ===\n*** goroutine dump...\n%s\n*** end\n", buf[:stacklen])
    }
  }()
```

-  

  check interface implementation during compilation

  ```
  var _ io.Reader = (*MyFastReader)(nil)
  ```

-  

  if a param of len is nil then it's zero

  - https://golang.org/pkg/builtin/#len

-  anonymous structs are cool

```
  var  hits  struct { 
    sync。Mutex 
    n  int 
  } 命中。Lock（）
   命中。n ++ 命中。解锁（）
  
  
```

-  

  ```
  httputil.DumpRequest
  ```

   是非常有用的东西，不要创建自己的东西

  - https://godoc.org/net/http/httputil#DumpRequest

- 要获取调用堆栈，我们已经`runtime.Caller` https://golang.org/pkg/runtime/#Caller

-  封送任意JSON，您可以封送 `map[string]interface{}{}`

- 配置您的位置，

  ```
  CDPATH
  ```

  以便您可以

  ```
  cd github.com/golang/go
  ```

  从任何董事那里进行

  - 将此行添加到您的`bashrc`（或类似的）`export CDPATH=$CDPATH:$GOPATH/src`

-  切片中的简单随机元素

  - `[]string{"one", "two", "three"}[rand.Intn(3)]`

- 