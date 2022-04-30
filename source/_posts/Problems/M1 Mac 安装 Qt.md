M1 Macbook（Monterey 12.2.1） 安装Qt遇到的问题记录

### 一

首先执行：

```shell
brew install qt
```

没有意外报错了，错误信息如下：

```shell
Running `brew update --preinstall`...
fatal: Could not resolve HEAD to a revision
==> Auto-updated Homebrew!
Updated 1 tap (homebrew/cask).
==> New Casks
keysafe                    mixed-in-key-live          wpsoffice-cn
==> Updated Casks
Updated 40 casks.

Warning: No available formula with the name "qt".
==> Searching for similarly named formulae...
Error: No similarly named formulae found.
==> Searching for a previously deleted formula (in the last month)...
Error: No previously deleted formula found.
==> Searching taps on GitHub...
Error: No formulae found in taps.
```

### 二

那我就更新一下brew吧，执行：

```shell
brew update --preinstall
```

又是报错：

```shell
fatal: Could not resolve HEAD to a revision
```

### 三

应该是brew出问题了，再次用这个命令更新一下：

```shell
brew update --verbose
```

孩她妈报错：

```shell
Checking if we need to fetch /opt/homebrew...
Checking if we need to fetch /opt/homebrew/Library/Taps/homebrew/homebrew-cask...
Fetching /opt/homebrew...
Checking if we need to fetch /opt/homebrew/Library/Taps/homebrew/homebrew-core...
Fetching /opt/homebrew/Library/Taps/homebrew/homebrew-core...
Updating /opt/homebrew...
Branch 'master' set up to track remote branch 'master' from 'origin'.
Switched to and reset branch 'master'
Your branch is up to date with 'origin/master'.
Switched to and reset branch 'stable'
Current branch stable is up to date.

Updating /opt/homebrew/Library/Taps/homebrew/homebrew-core...
fatal: Could not resolve HEAD to a revision

Already up-to-date.
```

### 四

但至少信息详细了点，注意看`Updating /opt/homebrew/Library/Taps/homebrew/homebrew-core...`，于是我们，进入这个目录，执行下面的命令

```shell
cd /opt/homebrew/Library/Taps/homebrew/homebrew-core
```

```shell
git fetch --prune origin
```

```shell
git pull --rebase origin master
```

命令都执行完后，在更新brew就易如反掌了，最后顺利安装Qt

```shell
brew update
```

```shell
brew install qt
```