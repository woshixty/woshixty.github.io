灵感来自 [jadx-ai-mcp环境搭建AI逆向分析JAVA代码](https://mp.weixin.qq.com/s?__biz=MzI2Mzk4MjczMQ==&mid=2247484452&idx=1&sn=a32ba648aefb3141734606f502a7b6d4)

[TOC]

# Claude + JADX + MCP 逆向分析 Android APK 的完整实战（Windows 10）

> MCP 服务器 + JADX 插件，通过 MCP 与 LLM 通信，使用 Claude 等 LLM 分析 Android APK — 轻松发现漏洞、分析 APK 和进行逆向工程，以 Shizuku.apk 为例，搭建 “JADX ⇨ MCP Server ⇨ Claude Desktop” 的 AI 逆向分析链路

GitHub 项目：
https://github.com/zinja-coder/jadx-ai-mcp

## 前言

最近有WhatsApp逆向修改的需求，发现了一套比较有意思的逆向工作流：
- 用 JADX 反编译 APK
- 用 jadx-ai-mcp 插件 + MCP Server 把 JADX 的项目上下文暴露出去
- 用 Claude Desktop 作为“AI 逆向助手”，自动读代码、查类、看 Manifest、找可疑函数等等

## 准备工具链

- JADX 1.5.3（with JRE 版）
- jadx-ai-mcp 插件 + jadx-mcp-server
- Python 3.10+
- Claude Desktop

## 安装 & 选择合适的 JADX 版本
在 JADX 的 Release 页面会看到类似这样的三个压缩包（Windows）：
 - jadx-1.5.3.zip   通用版（含 GUI + 命令行），需要自己安装 Java
 - jadx-gui-1.5.3-win.zip   只有 GUI，不带 JRE，也需要自备 Java
 - jadx-gui-1.5.3-with-jre-win.zip   Windows GUI 版 + 内置 JRE，开箱即用

我选 jadx-gui-1.5.3-with-jre-win.zip，原因：懒得再弄 JDK/JRE，解压即跑

解压后结构类似：
```shell
D:\applications\jadx-gui-1.5.3-with-jre-win\
    ├─ jadx-gui-1.5.3.exe
    ├─ runtime\  (内置 JRE)
    └─ ...
```

双击 jadx-gui-1.5.3.exe 能正常打开就 OK

## 安装 jadx-ai-mcp 插件

官方文档提供了两种安装方式：

### 方式 A：命令行安装

命令如下：
```shell
jadx plugins --install "github:zinja-coder:jadx-ai-mcp"
```

!!! 注意：带 JRE 的 GUI 版只有 jadx-gui-1.5.3.exe，没有 jadx.exe 这个命令行入口，所以下面的安装无用，with-jre GUI 版老老实实手动装
```shell
.\jadx-gui-1.5.3.exe plugins --install "github:zinja-coder:jadx-ai-mcp"
```

运行之后 毫无反应，因为 GUI 版本不会解析这个 plugins 子命令

### 方式 B：手动安装 .jar 插件

在 [GitHub Release](https://github.com/zinja-coder/jadx-ai-mcp/releases) 中下载插件 jar 文件，例如：jadx-ai-mcp-0.x.x.jar

找到（或创建）JADX 的插件目录，JADX 默认插件目录在用户目录下：
```shell
C:\Users\<用户名>\.local\share\jadx\plugins\
```

目录不存在则创建
```shell
mkdir C:\Users\qyyzx\.local\share\jadx\plugins\
```

把 jadx-ai-mcp-*.jar 丢进去，重启 JADX GUI，在菜单 File → Preferences → Plugins 中就能看到这个插件，成功后，打开 APK 时会多出一个 AI MCP Server 状态按钮：

显示 Status: Running，Port: 8650 就说明插件这边 OK 了

## 运行 MCP Server：Python 版本与本地服务

插件本身还不够，它只是让 JADX 暴露一套 MCP 接口，真正处理请求的是 Python 写的 jadx-mcp-server。

在 [GitHub Release](https://github.com/zinja-coder/jadx-ai-mcp/releases) 中下载插件 jadx-mcp-server 文件，例如：jadx-mcp-server-xxxx.zip

### 1. 解压 & 安装依赖
```shell
python -m venv .venv
.\.venv\Scripts\activate
pip install httpx fastmcp
```

### 2. 启动 MCP Server
```shell
python jadx_mcp_server.py
```

看到类似监听端口的日志，表示服务端也 OK 了

## 让 Claude Desktop 认识这个 MCP Server

在 Claude Desktop 中配置 claude_desktop_config.json。

### 1. 配置文件位置

Windows 下路径：
```shell
%APPDATA%\Claude\claude_desktop_config.json
```

刚装好的 Claude 默认是 没有这个文件的，需要自己建一个

### 2. 正确配置：使用 command + args

配置如下（按你自己的路径修改）：
```json
{
  "mcpServers": {
    "jadx-mcp": {
      "command": "D:\\applications\\jadx-gui-1.5.3-with-jre-win\\jadx-mcp-server\\.venv\\Scripts\\python.exe",
      "args": [
        "D:\\applications\\jadx-gui-1.5.3-with-jre-win\\jadx-mcp-server\\jadx_mcp_server.py"
      ]
    }
  }
}
```

注意几点坑：
- 路径必须写成 双反斜杠 \\（JSON 里的 \ 要转义）
- command 指向的是 Python 可执行文件
- args 第一个参数写 jadx_mcp_server.py 的完整路径
- 不要再手动运行 python jadx_mcp_server.py，Claude 会自己启动它（否则容易端口冲突）

保存后 重启 Claude Desktop，如果一切正常，你会在 Claude 的工具列表里看到一个：jadx-mcp，执行 list_tools 也能看到完整工具清单

## 调用：Shizuku.apk + Claude AI

在 JADX 中打开 Shizuku.apk，然后在 Claude Desktop 里问：我想了解这个 APK 文件的核心逻辑

Claude 开始调用工具分析：
```
自动调用 get_main_application_classes_names
自动调用 get_class_source 去拉取对应类的代码
然后给出中文分析
```

当前版本的 Claude Desktop 对 MCP 的「Always allow」有点玄学，有时候重启后还会再问一遍