# 关注的性能指标

- CPU 使用情况
- 内存 使用情况
- I/O 使用情况
- goroutine 使用情况: 阻塞
- 锁 使用情况: 死锁



# 工具

## `pprof`性能分析工具包

示例: `https://github.com/luode0320/go-pprof.git`

```go
package main

import (
	"log"
	"net/http"
	_ "net/http/pprof"
	"os"
	"runtime"
	"time"

	"github.com/wolfogre/go-pprof-practice/animal"
)

func main() {
	log.SetFlags(log.Lshortfile | log.LstdFlags)
	log.SetOutput(os.Stdout)

	runtime.SetMutexProfileFraction(1) // 开启锁跟踪
	runtime.SetBlockProfileRate(1)     // 开启阻塞跟踪

	// 通过web采集: http://localhost:6060/debug/pprof/
	go func() {
		if err := http.ListenAndServe(":6060", nil); err != nil {
			log.Fatal(err)
		}
		os.Exit(0)
	}()

	for {
		for _, v := range animal.AllAnimals {
			v.Live()
		}
		time.Sleep(time.Second)
	}
}
```

![image-20240807012932756](image-20240807012932756.png)



## go自带性能分析工具

cmd命令行执行

```cmd
go tool pprof -h
```

![image-20240807012805686](image-20240807012805686.png)





# 使用

## 进入命令行交互

```cmd
go tool pprof http://localhost:6060/debug/pprof/allocs // 所有情况
go tool pprof http://localhost:6060/debug/pprof/block // 阻塞情况
go tool pprof http://localhost:6060/debug/pprof/goroutine // goroutine情况
go tool pprof http://localhost:6060/debug/pprof/heap // 堆情况
go tool pprof http://localhost:6060/debug/pprof/mutex // 锁情况
go tool pprof http://localhost:6060/debug/pprof/profile // cpu情况
go tool pprof http://localhost:6060/debug/pprof/threadcreate // 线程创建情况
```

![image-20240807013031526](image-20240807013031526.png)

## `help`: 帮助指令![image-20240807013159075](image-20240807013159075.png)

## `top`: 查看前10条, 内存倒序![image-20240807013255841](image-20240807013255841.png)

- **flat**: 函数自身允许资源消耗
- **flat%**: 消耗百分比
- **Sum%**: 资源消耗总和
- **Cum**: 当前函数加上它所有调用栈允许总消耗
- **Cum%**: 消耗百分比
- **堆栈**



## `list`: 看指定源代码内存占用

![image-20240807013608774](image-20240807013608774.png)

## `web`: 下载svg

`Graphviz` 是一个用于绘制图形和网络图的工具，`pprof` 使用 `Graphviz` 的 `dot` 工具来生成可视化的火焰图和其他类型的图表。



#### 1. 安装 `Graphviz`

确保您已经安装了 `Graphviz`。以下是在不同操作系统上安装 `Graphviz` 的方法：

- **Ubuntu/Debian**

```bash
sudo apt-get update
sudo apt-get install graphviz
```

- **CentOS/RHEL**

```bash
sudo yum install graphviz
```

- **macOS** (使用 Homebrew)

```bash
brew install graphviz
```

- **Windows**
    - 访问 [Graphviz 官方网站](https://graphviz.org/download/) 下载 Windows 安装包。
    - 安装后，确保将 `Graphviz` 的安装目录添加到系统的 `PATH` 环境变量中。

#### 2. 添加 `Graphviz` 到 `PATH`

确保 `Graphviz` 的 `dot` 可执行文件位于系统路径中。通常，安装完成后 `dot` 会在某个目录下，例如 `/usr/bin` 或 `/usr/local/bin`。



**Linux/macOS**: 如果 `dot` 不在默认路径中，您可以通过以下命令找到它：

```bash
which dot
```

如果找不到，您需要手动将 `dot` 的路径添加到 `PATH` 中。例如，如果 `dot` 的路径是 `/usr/local/bin/dot`，您可以编辑 `~/.bashrc` 或 `~/.zshrc` 文件（取决于您使用的 shell），并添加以下行：

```bash
export PATH=/usr/local/bin:$PATH
```

然后运行 `source ~/.bashrc` 或 `source ~/.zshrc` 使更改生效。



**Windows**: 如果您在 Windows 上安装了 `Graphviz`，请确保将 `Graphviz` 的安装目录添加到 `PATH` 环境变量中。通常安装过程中会有提示是否添加到 `PATH`，如果没有，您需要手动添加。



#### 测试 `Graphviz` 是否正确安装

```bash
dot -V
```

![image-20240807014851765](image-20240807014851765.png)

#### 下载svg

![image-20240807015351890](image-20240807015351890.png)

- 浏览器自动打开

![image-20240807015411396](image-20240807015411396.png)

- 框框越大, 线越粗, 问题越大