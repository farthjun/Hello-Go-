## **Go语言写第一个包&做第一次测试**

### 1. 创建工作空间

首先创建一个工作空间目录，并设置相应的GOPATH，例如

```
$mkdir $HOME/goProject
$export GOPATH=$HOME/goProject
```

***

### 2. 第一个程序——Hello world

首先要选择包路径（我们在这里使用 `github.com/user/hello`），并在工作空间内创建相应的包目录：

```
 $mkdir $GOPATH/src/github.com/farthjun/hello
```

接着在hello目录中创建hello.go文件，内容为以下Go代码：

```
package main

import "fmt"

func main() {
	fmt.Printf("Hello, world.\n")
}
```

常见的go指令有三种：

> go build：编译出可执行文件
>
> go install：go build+把编译后的可执行文件放到GOPATH/bin目录下
>
> go get：git clone + go install

这里我们运行go install， 可以发现GOPATH目录下多了一个bin文件，里面正是hello.go的可执行文件：

![1568516315476](C:\Users\JUN\Desktop\img\view_bin.png)

直接运行hello：

![1568516376418](C:\Users\JUN\Desktop\img\hello.png)

***

### 3. 第一个库

编写一个库，让hello程序使用它。

选择包路径（我们将使用 `github.com/farthjun/stringutil`） 并创建包目录：

```
$ mkdir $GOPATH/src/github.com/farthjun/stringutil
```

接着在stringutil目录下创建文件reverse.go(作用是反转字符串)，并作如下编辑：

```
//stringutil for string processing
package stringutil

//Reverse string
func Reverse(s string) string{
    r := []rune(s)
    for i,j := 0, len(r)-1; i< len(r)/2; i, j = i+1, j-1 {
        r[i], r[j] = r[j], r[i]
    }
    return string(r)
}
```

现在使用go build命令进行编译：

```
$ go build github.com/farthjun/stringutil
```

由于reverse.go没有包含package main，不会产生输出文件。我们还需要对hello.go作以下修改：

```
package main

import ("fmt"
        "github.com/farthjun/stringutil"
)

func main(){
    //fmt.Printf("hello, world\n")
    fmt.Printf(stringutil.Reverse("!oG ,olleH"))
}
```

再使用go install命令来安装hello程序，此时stringutil包也会被自动安装。再运行hello：

![1568517087251](C:\Users\JUN\Desktop\img\hello_go.png)

字符串被成功反转，证明我们的库是有效的。

做完这些，我们的工作空间又多了一个目录pkg：

![1568517260376](C:\Users\JUN\Desktop\img\pkg.png)

此时，我们的工作空间结构如下(同本github项目的目录树)：

```
bin/
	hello                 # 可执行命令
pkg/
	linux_amd64/          # 这里会反映出你的操作系统和架构
		github.com/user/
			stringutil.a  # 包对象
src/
	github.com/user/
		hello/
			hello.go      # 命令源码
		stringutil/
			reverse.go       # 包源码
```

***

### 4. 测试

Go拥有一个轻量级的测试框架，它由 `go test` 命令和 `testing` 包构成。

你可以通过创建一个名字以 `_test.go` 结尾的，包含名为 `TestXXX` 且签名为 `func (t *testing.T)` 函数的文件来编写测试。 测试框架会运行每一个这样的函数；若该函数调用了像 `t.Error` 或 `t.Fail` 这样表示失败的函数，此测试即表示失败。

我们可通过创建文件 `$GOPATH/src/github.com/farthjun/stringutil/reverse_test.go` 来为 `stringutil` 添加测试，其内容如下：

```
package stringutil

import "testing"

func TestReverse(t *testing.T){
    cases := []struct {
        in, want string
    } {
        {"Hello, world", "dlrow ,olleH"},
        {"Hello, go", "og ,olleH"},
        {"", ""},
    }
    for _, c := range cases {
        got := Reverse(c.in)
        if got != c.want{
            t.Errorf("Reverse(%q) == %q, want %q", c.in, got, c.want)
        }
    }
}
```

接着在stringutil目录下使用go test运行该测试：

```
$ go test
```

![1568517529002](C:\Users\JUN\Desktop\img\test.png)

说明通过测试。