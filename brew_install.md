Mac安装brew并更改源
https://www.jianshu.com/p/22820319f71b

1.问题描述
  在Mac上安装brew时，如果使用官方推荐的方式，会耗费很长时间，并且也不一定能成功
2. 解决方案
  将安装源换成国内源。
3. brew安装具体过程
1）将brew的install文件下载本地
$ cd
$ curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install >> brew_install


注意：命令前面的 $ 符号不要复制过去，$ 符号是Mac和Linux系统中普通用户的标志，root用户标志为 #，下同
2）修改install文件的镜像源
$ vim brew_install 

  以上命令执行之后会在终端窗口中带开brew_install文件，进入的vim编辑器界面。不会使用vim命令也可以用其他文本编辑器打开。然后将brew_install文件里面的两行代码替换掉，待替换的代码为：
BREW_REPO = "https://github.com/Homebrew/brew".freeze
CORE_TAP_REPO = "https://github.com/Homebrew/homebrew-core".freeze

  替换为：
BREW_REPO = "git://mirrors.ustc.edu.cn/brew.git".freeze
CORE_TAP_REPO = "git://mirrors.ustc.edu.cn/homebrew-core.git".freeze

或者将要替换掉的两行代码注释掉，注释符号为 # ，效果如下：
# BREW_REPO = "https://github.com/Homebrew/brew".freeze
# CORE_TAP_REPO = "https://github.com/Homebrew/homebrew-core".freeze
BREW_REPO = "git://mirrors.ustc.edu.cn/brew.git".freeze
CORE_TAP_REPO = "git://mirrors.ustc.edu.cn/homebrew-core.git".freeze

此时1、2两行代码前面加上了 # 号，说明代码被注释掉了，后面两行是我们添加的代码
  修改完成之后保存好修改后的brew_install文件，并退出。
3）安装brew
 emsp;代码如下：
$ /usr/local/bin/ruby ~/brew_install

4) 修改PATH变量
  代码如下：
$ vim /etc/profile

  在打开的profile文件中加入下面这一行
export PATH=/usr/local/bin:$PATH

注意：更改/etc/profile文件可能需呀root权限，可以尝试用 sudo vim /etc/profile 命令（需要输入密码），若还是不行就需要切换到root用户，然后在进行编辑，切换到root用户的命令为 su - ，然后会要求输入密码，该密码为root用户的密码，若还不明白自行百度。
  执行下面一条命令，使得刚才的更改立即生效：
$ source /etc/profile

5) 验证
  执行命令：
$ brew doctor

6) 安装wget
  执行命令：
$ brew install wget 

4. 修改brew的源为国内源
  依次执行如下命令：
$ cd "$(brew --repo)"
$ git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
$ cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
$ git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
$ brew update

5. 参考资源
1）使用国内源安装brew并更改源

2) mac brew 权限问题解决记录
