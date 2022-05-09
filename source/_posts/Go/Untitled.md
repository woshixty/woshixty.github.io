## Go语言基础-包（Packet）

Go语言源码复用建立在包的基础之上

Go语言的包与文件夹一一对应，所有与包有关的操作必须依赖于工作目录（GOPATH）

### 一、工作目录（GOPATH）

GOPATH是GO语言使用的一个环境变量，使用绝对路径提供项目的工作目录。

#### 1、使用命令行查看GOPATH

```shell
go env
```

执行上面的指令可以得到以下输出（部分）

```shell
GO111MODULE="on"
GOARCH="arm64"
GOBIN="/opt/homebrew/Cellar/go/1.18.1/bin"
GOCACHE="/Users/xietingyu/Library/Caches/go-build"
GOENV="/Users/xietingyu/Library/Application Support/go/env"
......
GOPATH="/Users/xietingyu/go"
......
```

- GOARCH：目标处理器架构
- GOBIN：编译器和链接器安装位置

就不一一解释了，由上命令行输出可见，GOPATH路径为：`/Users/xietingyu/go`



#### 2、使用GOPATH的工程结构

代码保存在`$GOPATH/src`下

工程经过编译后的二进制文件在`$GOPATH/bin`下

生成的中间缓存文件在`$GOPATH/pkg`下



#### 3、设置和使用GOPATH

##### （1）设置当前目录为GOPATH

```shell
export GOPATH=`pwd`
```

##### （2）建立GOPATH的源码路径

```shell
mkdir -p src/hello
```

- -p：表示递归创建·文件夹

##### （3）添加main.go文件

先进入`src/hello`文件夹

```shell
cd src/hello
```

使用vim创建文件

```shell
vim hello.go
```

写入以下代码

```go
package main

import "fmt"

func main() {
	fmt.Println("hello")
}
```

##### （4）编译运行

```shell
go install hello
```

在`bin`目录下会有名为hello的可执行文件

```shell
./hello
```

