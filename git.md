## Git操作

### Git初始化配置

​		一般在新的系统上，我们都需要先配置下自己的git工作环境。配置工作只需要一次，以后升级git时，还会沿用现在的配置。当然，如果需要，我们可以用相同的命令来修改已有的配置。

​		Git提供了一个叫做`git config`的命令来配置或读取相应的工作环境变量，而正是由这些环境变量，决定了Git在各个环节的具体工作方式和行为。这些变量可以存放在以下三个不同的地方：

- /etc/gitconfig文件：系统中对所有用户都普遍适用的配置。若使用`git congit`时，用`--system`选项，读写的就是这个文件。
- ~/.gitconfig文件：用户目录下的配置文件只适用于该用户。若使用`git config`时，用`--global`选项，读写的就是这个文件。
- .git/config文件：当前项目的Git目录中的配置文件（也就是工作目录中的`.git/config`文件）这是的配置仅仅针对当前项目有效。

例如：

```bash
# git config --global user.name "zhouwen"
# git config --global user.email "18200233689@163.com"
```

​		要检查已有的配置信息，可以使用`git config --list`命令

### Git底层概念（底层命令）

#### 初始化新仓库

​		命令：`git init`

​		解析：要对现有的某个项目开始使用 **git** 管理，只需要到此项目所在的目录，执行：git init

​		作用：初始化后，在当前目录下会出现一个名为 `.git` 的目录，所有`Git`需要的数据和资源都存放在这个目录中。不过目前，仅仅是按照既有的结构框架初始化好了里面所有的文件和目录，但我们还没有开始跟踪管理项目中的任何一个文件。

#### .git目录

![](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210924091514337.png)

​		hooks：目录包含客户端或服务端的钩子脚本

​		info：包含一个全局性排除文件，git不需要管理的文件可以放在里面。

​		logs：保存日志信息

​		**objects**：目录存储所有数据内容

​		**refs**：目录存储指向数据（分支）的提交对象的指针

​		config：文件包含项目特有的配置选项

​		description：用来显示对仓库的描述信息

​		**HEAD**：文件指示目前被检出的分支

​		**index**：文件保存暂存区信息

#### Git Objects的类型区分

- Git Objects包含4种类型：Blob , Tree , Commit , Tag
- Blob 负责存储文件内容
- Tree：目录索引
- Commit：提交元信息（作者，提交都，说明，父提交等）并关联一个目录索引视图（root-tree）
- Tag：记录重要历史时刻的提交节点和相关信息（一般用于标记发行版）



#### git对象

​		Git 的核心部分是一个简单的键值对数据库。你可以向该数据库插入任意类型的内容，它会返回一个键值，通过该键值可以在任意时刻再次检索该内容

- 向数据库写入内容，并返回对应键值

  - 命令：

    ```bash
    # echo "test content" | git hash-object -w --stdin
    # -w 选项指示hash-object命令存储数据对象；若不指定此选项，则该命令仅返回对应的键值
    # --stdin选项则指示该命令从标准输入读取内容；若不指定此选项，则须在命令尾部给出待定存储文件的路径
    ```

    ```bash
    # git hash-object -w 文件路径
    	# 存文件
    # git hash-object 文件路径
    	# 返回对应文件的键值
    ```

  - 返回：

    该命令输出一个长度为40个字符的校验和。这是一个SHA-1哈希值

- 查看Git是如何存储数据的

  - 命令：

    ```bash
    admin@DESKTOP-8L26URU MINGW64 ~/Desktop/testgit/.git/objects/72 (GIT_DIR!)
    $ pwd
    /c/Users/admin/Desktop/testgit/.git/objects/72
    
    admin@DESKTOP-8L26URU MINGW64 ~/Desktop/testgit/.git/objects/72 (GIT_DIR!)
    $ ls
    943a16fb2c8f38f9dde202b7a70ccc19c52f34
    
    # 这就是开始时，Git存储内容的方式：一个文件对应一条内容。校验和的前两个字符用于命名子目录，余下的38个字符则做文件名。
    ```

- 根据键值拉取数据

  - 命令：

    ```bash
    # git cat-file -p 943a16fb2c8f38f9dde202b7a70ccc19c52f34
    # -p 选项可以指示该使命令自动判断内容的类型，并为我们显示格式友好的内容
    ```

  - 返回：

    ```bash
    admin@DESKTOP-8L26URU MINGW64 ~/Desktop/testgit/.git/objects/72 (GIT_DIR!)
    $ git cat-file -p 72943a16fb2c8f38f9dde202b7a70ccc19c52f34
    aaa
    ```

#### 对一个文件进行简单的版本控制

- 创建一个新文件并将其内容存入数据库

  - 命令：

    ```bash
    # echo 'version 1' > test.txt
    # git hash-object -w test.txt
    ```

- 利用 cat-file -t 命令，可以让git 告诉我们其内部存储的任何对象类型。

- 问题：

  	1. 记住文件的每一个版本所对应的SHA-1值并不现实 
  	2. 在git中，文件名并没有被保存，仅仅保存了文件的内容
  	3. 解决方案：树对象

- 注：当前的操作都是对本地数据库进行操作，不涉及暂存区

#### 树对象

​		树对象（tree object），它能解决文件名保存的问题，也允许我们将多个文件组织到一起。Git以一种类似于UNIX文件系统的方式存储内容。所有内容均以树对象和数据对象(git对象)的形式存储，其中树对象对应了UNIX中的目录项，数据对象（git对象）则大致上对应文件内容。一个树对象包含了一条或多条记录（每条记录含有一个指向Git对象或或者子树对象的SHA-1指针，以及相应的模式，类型，文件名信息）。一个树对象也可以包含另一个树对象。

- 构建树对象

  ​	我们可以通过update-index;write-tree;read-treee等命令来构建树对象并塞入到暂存区。

  ​	假设我们做了一系列操作之后得到一个树对象

  - 操作

    1.利用update-index命令为test.txt 文件的首个版本创建一个暂存区。并通过write-tree命令生成树对象。

    ​	命令：

    ​	**git update-index --add --cacheinfo** 100644 72943a16fb2c8f38f9dde202b7a70ccc19c52f34 own_name.txt  ：往暂存区添加一条记录（让git对象对应上文件名），保存到.git/index

    ​		文件模式为	100644，表明这是一个普通文件

    ​								100755，表示一个可执行文件

    ​								120000，表示一个符号链接

    ​		--add选项：因为此前该文件并不在暂存区，首次需要 --add

    ​		--cacheinfo 选项：因为将要添加的文件位于Git数据库中，而不是位于当前目录下，所以需要--cacheinfo

    

    ​	**上面的命令执行，并不会生成树对象，只是在暂存区里面添加了内容。**

    ```bash
    admin@DESKTOP-8L26URU MINGW64 ~/Desktop/testgit (master)
    $ find .git/objects/ -type f
    .git/objects/3b/18e512dba79e4c8300dd08aeb37f8e728b8dad
    ```

    

    ​	**git write-tree：**生成树对象，**保存当前暂存区里面的git对象。  生成树对象存到.git/objects**

    ```bash
    admin@DESKTOP-8L26URU MINGW64 ~/Desktop/testgit (master)
    $ find .git/objects/ -type f
    .git/objects/3b/18e512dba79e4c8300dd08aeb37f8e728b8dad
    .git/objects/c3/b8bb102afeca86037d5b5dd89ceeb0090eae9d
    ```

    

    

    ​	**查看暂存区里面的对象：git ls-files -s**

    3. 新增new.txt 将net.txt和test.txt文件的第二个版本塞入暂存区。并通过write-tree命令生成树对象。

    4. 查看暂存区里面的对象：git ls-files -s

- 查看树对象

  - 命令：

  ```bash
  # git cat-file -p master^{tree} (或者是树对象的hash)
  # master^{tree} 语法表示 master 分支上最新的提交所指向的树对象
  ```

#### 提交对象

​		我们可以通过调用commit-tree命令创建一个提交对象，为此需要指定一个树对象的SHA-1值，以及该提交的父提交对象（如果有的话，第一次将暂存区做快照就没有父对象）

- 创建提交对象

  ```bash
  # echo 'first commit' | git commit-tree 树对象的HASH值
  ```

### Git本地操作（高层命令）

#### 初始化新仓库

- git init 初始化仓库

  解析：要对现有的某个项目开始使用git管理，只需要到此项目所在的目录，执行：git init

  作用：初始化后，在当前目录下会出现一个名为.git的目录，所有Git需要的数据和资源都存放在这个目录中。不过目前，仅仅是按照既有的结构框架初始化好了里面所有的文件和目录，但我们还没有开始跟踪管理项目中的任何一个文件。

#### 记录每次更新到仓库

​		工作目录下面的所有文件都不外乎这两种状态：**已跟踪**  或  **未跟踪**

​				已跟踪的文件是指本来就被纳入版本控制管理的文件，在上次快照中有它们的记录，工作一段时间后，它们的状态可能是已提交，已修改或者已暂存

​				所有其它文件都属于未跟踪文件。它们即没有上次更新时的快照，也不在当前的暂存区域。

- git add ./  将修改添加到暂存区
- git commit -m 注释  将暂存区提交到版本库

#### 检查当前文件状态

- git status

  查看当前git版本控制的整个状态

- git diff

  查看当前工作空间哪些文件被修改过，这儿的diff是和暂存区比较

- git diff --staged

  查看当前暂存区里面的文件内容和当前工作空间文件内容的差异

#### 基本操作

- 跟踪新文件（暂存）

  命令：git add 文件名

  作用：跟踪一个文件

  ​		再次运行git status 命令，会看到文件已被跟踪，并处于暂存状态

- 暂存已修改文件

- 提交更新

  命令：git commit -m

- 跳过使用暂存区域

  命令：git commit -a -m，-a 参数的使用，必须在使用过git add ./命令过后

- 移除文件

  ​		要从git中移除某个文件，就必须要从已跟踪文件清单中注册删除（确切地说，是在暂存区载注册删除），然后提交。可以用git rm命令完成此项工作，并连带从目录中删除指定的文件，这样以后就不会出现在未跟踪文件清单中了。

  ​		1.从工作目录中手工删除文件

  ​		git status

- 查看历史记录

  - git log

    ​	在提交了若干更新，又或者克隆了某个项目之后，你也许想回顾下提交历史。完成这个任务最简单而又有效的工具是git log命令

    ```bash
    # git log
    commit bf0d408a2fef5e1fa8f201d48ffe56ac1bbde2bc
    Author: zhouwen <313755368@qq.com>
    Date:   Fri Sep 24 10:11:13 2021 +0800
    
        第一次用git提交暂存区里的内容到git版本控制器里面
    ```

    ​	默认不用任何参数的话，`git log`会按照提交时间列出所有的更新，最近的更新排在最上面。这个命令会列出每个提交的SHA-1检验和，作者的名字和电子邮件地址，提交时间以及提交说明。

  - git log 参数

    git log --pretty=oneline

    git log --oneline

#### 高层命令总结

##### 查看版本：

​		git --version

##### 初始化配置：

​		git config --global user.name "zhouwen"

​		git config --global user.email 18200233689@163.com

​		git config --list

##### 初始化仓库

​		git init

​				

### Git分支操作（杀手功能）

​		几乎所有的版本控制系统都以某种形式支持分支。使用分支意味着你可以把你的工作从开发主线上分离开来，以免影响开发主线。在很多版本控制系统中，这是一个略微低效的过程----常常需要完全创建一个源代码目录的副本。对于大项目来说，这样的过程会耗费很多时间。

​		而git的分支模型极其的高效轻量。是git的必杀技特性，也正因为这一特性，使得git从众多版本控制系统中脱颖而出。

#### 创建分支

命令：git branch

作用：

​		为你创建了一个可以移动的新的指针。比如，创建一个testing分支：git branch testing。这会在当前所在的提交对象上创建一个指针

注意：

​		a. **git branch 分支名** 创建一个新分支，并不会自动切换到新分支中去。

​		b. git branch 不只是可以创建与删除分支。如果不加任何参数运行它，**会得到当前所有分支的一个列表**

​		c. **git branch -d** name

​				删除分支

​		d. **git branch name commitHash**

​				新建一个分支并且使分支指向对应的提交对象

​		e. git branch --merged

​				查看哪些分支已经合并到当前分支

​				在这个列表中分支名字前没有*号的分支通常可以使用

​				git branch -d 删除

​		f. git branch --no-merged

​				查看所有包含未合并工作的分支

​				尝试使用git branch -d 命令删除在这个列表中的分支时会失败。如果真的想要删除分支并丢掉那些工作，可以使用-D选项强制删除。

​		g. git branch -v

​			查看每一个分支的最后一次提交

#### 查看当前分支所指提交对象

- 命令：

  ```bash
  # git log --online-decorate
  # 提供这一功能的参数是  --decorate
  ```

#### 切换分支

- 命令：

  ```bash
  # git checkout 分支名
  ```

#### 创建分支并切换到新建的分支上

- 命令：

  ```bash
  # git checkout -b test
  ```

  

#### 配置别名

- 若某个命令特别长，如`git log --oneline --decorate --graph --all`，则可以为某个命令配置别名

- 命令

  ```bash
  # "config --global alias.longname "git log --oneline --decorate --graph --all"
  ```




### 撤销&重置

#### 重置reset

##### reset三部曲

移动HEAD

更新暂存区

更新工作目录



### 打tag

​	Git可以给历史中的革一个提交打卡标签，以示重要。比较有代表性的是人们会使用这个功能来标记发布结点（v1.0等等）

- 列出标签

  git tag

  git tag -l v1.8.5

- 创建 标签

  git 使用两种主要类型的标签：轻量标签与辅助标签

  **轻量标签**很像一个不会改变的分支，它只是一个特定提交的引用

  ​	git tag v1.4

  ​	git tag v1.4 commit-hash

- 删除标签

  git tag -d v1.4

- 检出标签

  如果你想查看某个标签所指向的文件版本，可以使用 git checkout命令

  git checkout tag-name(v1.4)

  虽然说这会使你的仓库处于“分离 头指针（detached HEAD) “状态。在”分离头指针“状态下，如果你做了某些更改然后提交它们，标签不会发生变化，但你的新提交将不属于任何分支，并且将无法访问，除非访问确切的提交哈希。因此，如果你需要进行更改，比如说你正在修改旧版本的错误，这通常需要创建一个新分支：

  git checkout -b version2

### Git操作最基本流程

- 创建工作目录，对工作目录进行修改

- git add ./ ：此命令是对下面两个命令的封装

  git hash-object -w 文件名(修改了多少个工作目录中的文件，此命令就要被执行多少次)

  git update-index 为文件的首个版本创建一个暂存区

- git commit -m "注释内容"：此命令是对下面两个命令的封装

  git write-tree  生成树对象，保存当前暂存区里面的git对象。

  git commit-tree  提交对象

### 

### 远程仓库

#### 什么是远程仓库

​		为了能在任意Git项目上团队协作，你需要知道如何管理自己的远程仓库。远程仓库是指托管在因特网或其他网络中你的项目版本库。你可以有好几个远程仓库，通常有些仓库对你只读，有些则可以读写。与他人协作涉及管理远程仓库以及根据需要推送或拉取数据。管理远程仓库包括了解如何添加远程仓库、移除无效的远程仓库、管理不同的远程分支并定义它们是否被跟踪等。

#### 远程协作基本流程

- 项目经理为远程仓库配置别名&用户信息

  ```bash
  # 添加一个新的远程Git仓库，同时指定一个你可以轻松引用的简写
  # git remote add <short_name><url>
  # SSH:git@github.com:Vigorous-Wen/Zhou.git
  # HTTPS:https://github.com/Vigorous-Wen/Zhou.git
  ```

  ```bash
  # 显示远程仓库使用的Git别名与其对应的URL
  # git remote -v
  ```

  ```bash
  # 查看某一个远程仓库的更多信息
  # git remote show [remote-name]
  ```

  ```bash
  # 重命名
  # git remote rename 
  ```

- 项目经理推送本地项目到远程仓库

  初始化一个本地仓库然后：

  ```bash
  # git push [remote-name] [branch-name]
  ```

- 成员克隆远程仓库到本地

  ```bash
  # git clone url (克隆时不需要 git init)
  # 默认克隆时为远程仓库起的别名为origin
  ```

  ​		远程仓库名字“origin”与分支名“master”一样，在Git中并没有任何特别的含义一样。同时“master”是当你运行git init时默认的起始分支名字，原因仅仅是它的广泛使用，“origin”是当你运行git clone时默认的远程仓库名字。如果你运行git clone -o booyah，那么你默认的远程仓库别名为booyah

- 项目经理邀请成员加入团队

  ​		如果你想与他人合作，并想给他们提交的权限，你需要把他们添加为"Collaborators"。如果 ben , jeff , louise 都在github上注册了，你想给他们推送的权限，你可以将他们添加到你的项目。这样做会给他们“推送”的权限，就是说他们对项目有读写的权限

- 项目经理更新成员提交的内容

  ```bash
  # git fetch [remote-name]
  # 这个命令会访问远程仓库，从中拉取所有你还没有的数据。执行完成后，你将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看
  # 必须注意，git fetch命令会将数据拉取到你的本地仓库，它并不会自动合并或修改你当前的工作。当准备好时，你必须手动将其合并入你的工作。
  ```

  