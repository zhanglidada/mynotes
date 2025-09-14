[[git学习补充]]
##一、git的常见用法：
###1.git解决冲突问题
当我们在git的本地分支进行修改后，如果遇到远程分支已经有很多修改提交后，可以先`git stash`将本地的缓存暂时存入栈中，再`git pull`把主分支的最新的代码拉下来；之后`git stash pop`将自己的stash中暂时缓存的代码和pull下来的代码在本地进行合并，并在代码中修改冲突的部分。

###2.`git commit`用法：
`git commit`主要是将暂存区里的改动给提交到本地的版本库。每次使用`git commit` 命令我们都会在本地版本库生成一个40位的哈希值，这个哈希值也叫`commit-id`，`commit-id`在版本回退的时候是非常有用的，它相当于一个快照,可以在未来的任何时候通过与`git reset`的组合命令回到这里.

1).`git commit -m “message”`:
-m 参数表示可以直接输入后面的“message”，如果不加 -m参数，那么是不能直接输入message的。

2). `git commit --amend`:
即追加提交，它可以在不增加一个新的`commit-id`的情况下将新修改的代码追加到前一次的`commit-id`中，用于解决对推送给服务器并且没有merge之前的版本代码的修改操作。

###3.`git checkout`命令:
(**一般用于操作文件或者操作分支**) `git checkout`命令用于切换分支或恢复工作树文件。git checkout是git最常用的命令之一，同时也是一个很危险的命令，因为这条命令会重写工作区。

1). 操作文件：
`git checkout filename`<font color=#4yt46>放弃单个文件的修改，可以是放弃删除，增加文件或者放弃修改文件。</font>

`git checkout . `放弃当前目录下的所有文件的修改

2). 操作分支：
`git checkout master` 将分支切换到master

`git checkout -b 目的分支` 如果分支存在则只切换分支，若不存在则创建并切换到目的分支。

###4.`git status`：
`git status`命令用于显示工作目录和暂存区的状态。<font color=#236689>使用此命令能看到那些修改被暂存到了, 哪些没有, 哪些文件没有被Git tracked到。</font>git status不显示已经commit到项目历史中去的信息。看项目历史的信息要使用`git log`.
补充：`git status`取消显示untracked files在屏幕上的红字
使用`git status -uno`，等价于`git status --untracked-files=no`

###5.`git rebase` 和`git merge`用法区别：
二者都可以用于合并两个分支的提交。<font color=#256489>git merge是将两个分支做一个三方合并(如果不是直接上游分支)，这样一来，查看提交历史记录，可能会显得非常凌乱。git rebase则会将当前分支相对于基低分支的所有提交生成一系列补丁，然后放到基底分支的顶端，从而使得提交记录变称一条直线，非常整洁。</font>

###6.git本地修改文件和远程合并解决冲突问题
在master分支上，本地进行修改后，如果利用`git log`查看master分支上的历史版本，可以用`git checkout hash`值进行切换版本; 但是记住，由于你对本地master进行了修改，所以此时工作区是dirty的，需要先`git stash`进行暂存，然后git checkout 切换到以前的分支，之后切换回master分支后需要git stash pop恢复自己进行的修改。

###7.`git diff`用法：

1)`git diff`用法：
**当暂存区中没有文件时，git diff比较的是，工作区中的文件与上次提交到版本库中的文件。**
**当暂存区中有文件时，git diff则比较的是，当前工作区中的文件与暂存区中的文件。**

2)`git diff HEAD --file`用法：
比较的是工作区中的文件与版本库中文件的差异。HEAD指向的是版本库中的当前版本，而file指的是当前工作区中的文件。
**补充：`git diff HEAD^`是和上一个提交进行比较**

###8.tig命令：
让git命令可视化

###9.`git reset`用法：
**用于撤回之前的提交。或者撤回之前对于文件的add**(使用的是默认的soft方式)
1).`git reset (–mixed) HEAD~1` :
回退一个版本,且会将暂存区的内容和本地已提交的内容全部恢复到未暂存的状态,不影响原来本地文件(未提交的也不受影响)

2). `git reset –soft HEAD~1`：
回退一个版本,不清空暂存区,将已提交的内容恢复到暂存区,不影响原来本地的文件(未提交的也不受影响)

3). `git reset --hard HEAD~1`：
回退一个版本,清空暂存区,将已提交的内容的版本恢复到本地,本地的文件也将被恢复后的版本替换

###10.`git add`用法：
`git add 文件`命令将<font color=red>文件内容</font>添加到索引中(也就是将修改添加到暂存区)。只有`git add 文件`将修改的文件添加到索引后，才可以继续进行commit操作。
1）`git add [path]`：这里path可以是文件，也可以是目录；git不仅能判断出[path]中，修改（不包括已删除）的文件，还能判断需要新添加的文件，并把它们的信息添加到索引库中。
2）`git add . 和 git add all`：两者都可以将所有未跟踪或者修改的文件添加到暂存区。
**区别：**`git add all`无论在哪个目录下执行均会将工作区中所有未跟踪或者修改的文件添加到暂存区。`git add .`只会将命令的当前执行目录及其子目录下的所有文件添加到暂存区中。

###11.`git branch`用法：
git branch一般用于分支的操作，比如创建分支，查看分支等等。

1). `git branch`：不带参数，表示列出本地已经存在的分支，并且在当前分支的前面用"*"标记

2). `git branch -r`： 查看远程版本库分支列表

3). `git branch -a`：查看所有分支列表，包括本地和远程

4). `git branch dev`：创建名为dev的分支，创建分支时需要是最新的环境，创建分支但依然停留在当前分支

5). `git branch -d dev`：删除dev分支，如果在分支中有一些未merge的提交，那么会删除分支失败，此时可以使用`git branch -D dev`：强制删除dev分支，

6). `git branch -vv`：可以查看本地分支对应的远程分支

7). `git branch -m oldName newName`：给分支重命名
**补充：从master分支创建新分支的时候都是默认复制master分支的文件到新创建的分支**

8). 删除远程分支：`git push origin --delete <remoteBranchName>`

###12.`git stash`的缓存操作：
1). `git stash save "save message"  `
写入缓存区，会把所有未提交的修改（包括暂存的和非暂存的）都保存起来，用于后续恢复当前工作目录。“save message”用于执行存储时，添加备注，方便查找，只有git stash 也要可以的，
但查找时不方便识别。注意，stash是本地的，不会通过`git push`命令上传到`git server`上。

2). `git stash list`:  用于列举当前缓存区中所有的stash。

3). `git stash apply`：通过名字指定使用哪个stash，默认使用最近的stash（即`stash@{0}`）

4). `git stash pop`：缓存出栈，并将对应内容应用到当前的工作目录下,默认为stash头,即`stash@{0}`，如果要应用并删除其他stash，命令：`git stash pop stash@{$num}` ，比如应用并删除第二个：`git stash pop stash@{1}`

5). `git stash drop stash@{$num}` : 丢弃stash@{$num}这个存储，从列表中删除这个存储

6). `git stash clear` ：  删除所有缓存的stash

7). `git stash show`：显示做了哪些改动，默认show第一个存储,如果要显示其他存贮，后面加stash@{$num}，比如第二个 git stash show stash@{1}

8). `git stash show -p` : 显示第一个存储的改动，如果想显示其他存存储，命令：git stash show  stash@{$num}  -p ，比如第二个：git stash show  stash@{1}  -p

**补充**：
<font color=#345678>新增的文件，直接执行stash是不会被存储的，因为没有在git 版本控制中的文件，是不能被git stash 存起来。
git add 只是把文件加到git 版本控制里，并不等于就被stash起来了，git add和git stash 没有必然的关系，但是执行git stash 能正确存储的前提是文件必须在git 版本控制中才行。</font>


###13.`git push`用法：
在使用git commit命令将修改从暂存区提交到本地版本库后，剩下最后一步就是将本地版本库的分支推送到远程服务器上对应的分支。git push的一般形式为：
`git push <远程主机名> <本地分支名>:<远程分支名> `，(**注意，这里冒号之间不要有空格**)例如：`git push origin master:refs/for/master`，即是将本地的master分支推送到远程主机origin上
的对应master分支，这里origin 是远程主机名。<font color=#356754>注意：第一个master是本地分支名，第二个master是远程分支名。</font>

1). `git push origin localbranch:remotebranch`：
将本地的loaclbranch推送到远程主机origin上面的remotebranch上，如果remotebranch不存在，则创建这个远程分支并进行本地分支的推送。

1). `git push origin master`
如果远程分支被省略，如上，则表示将本地分支`master`推送到与之存在追踪关系的远程分支**（通常两者同名）**，如果该远程分支不存在，则会被新建

2). `git push origin ：refs/for/master`
如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支，等同于`git push origin --delete master`

3). `git push origin`
如果当前分支与远程分支存在追踪关系，则本地分支和远程分支都可以省略，将当前分支推送到origin主机的对应分支

4). `git push`
如果当前分支只有一个远程分支，那么主机名都可以省略，形如`git push`，我们可以使用`git branch -r`，查看远程的分支名。

5).`git push 的其他命令`
a. `git push -u origin master`：如果当前分支与多个主机存在追踪关系，则可以使用 -u 参数指定一个默认主机，这样后面就可以不加任何参数使用git push。
对于不带任何参数的git push，默认只推送当前分支，这叫做simple方式，还有一种matching方式，会推送所有有对应的远程分支的本地分支，如果想更改设置，可以使用git config命令。
（`git config --global push.default matching` OR `git config --global push.default simple`，可以使用git config -l 查看配置。）

b. `git push --all origin`当遇到这种情况就是不管是否存在对应的远程分支，将本地的所有分支都推送到远程主机，这时需要 -all 选项

c. `git push --force origin`，在我们执行`git push`的时候需要本地先`git pull`更新到跟服务器版本一致，如果本地版本库比远程服务器上的低，那么一般会提示你git pull更新，但是如果一定要提交，那么可以使用这个命令。

d. `git push origin --tags` 我们在git push 的时候不会推送分支，如果一定要推送标签的话那么可以使用这个命令

6). 关于res/for：
refs/for 的意义在于我们提交代码到服务器之后是需要经过code review 之后才能进行merge的，而refs/heads 不需要。

###14.`git fetch`以及`git pull`用法：
![1](/assets/1_hoi71olcp.jpg)
`git fetch`是将远程主机的最新内容拉到本地，用户在检查了以后决定是否合并到工作本机分支中。而`git pull`则是将远程主机的最新内容拉下来后直接合并，即：
`git pull = git fetch + git merge`，但是这样可能会产生冲突，需要手动解决。
1). `git fetch`用法：
a. `git fetch <远程主机名>` ：这个命令将某个远程主机的更新全部取回本地

b. `git fetch <远程主机名> <分支名>` 只取回特定分支，注意之间有空格。如：`git fetch origin master`，即取回远程主机origin的master分支。

c. 取回更新后，会返回一个FETCH_HEAD ，指的是某个branch在服务器上的最新状态，我们可以在本地通过它查看刚取回的更新信息：`git log -p FETCH_HEAD`

2). `git pull`用法：
a. `git pull <远程主机名> <远程分支名>:<本地分支名>`：将远程主机的某个分支的更新取回，并与本地指定的分支合并。

b. `git pull <远程主机名> <远程分支名>`：如果远程分支是与当前分支合并，则冒号后面的部分可以省略。

###15.git取消对于文件的追踪：
1). 取消对所有文件的跟踪：
`git rm -r --cached` 　　//不删除本地文件

`git rm -r --f`  　　//删除本地文件

2). 取消对于某个文件的追踪：
`git rm --cached readme1.txt`：删除readme1.txt的跟踪，并保留在本地。

`git rm --f readme1.txt`：删除readme1.txt的跟踪，并且删除本地文件。

###16.`git remote用法：`
####16.1 查看关联的远程仓库信息
`git remote`：查看关联的远程仓库名称
`git remote -v`：查看关联的远程仓库的详细信息

####16.2 git操作远程仓库的关联
**远程仓库的名称一般默认为origin**，也可以设置别的名称；我们通过`git clone`下载项目到本地时，项目文件夹中的`.git`目录就是版本库目录。`.git`目录中的config文件中有远程仓库的关联配置。
1）添加远程仓库关联：`git remote add origin <远程仓库url>`

2）删除远程仓库关联：`git remote rm <name>`

3）修改远程仓库关联：
比如，之前我们关联的远程仓库使用的协议是 http ，现在需要将关联的远程仓库协议改为ssh协议；修改关联的远程仓库的方法，主要有三种：

第一种：使用`git remote set-url`命令，更新远程仓库的url
`git remote set-url origin <new_url>`

第二种：先删除之前关联的远程仓库，再来添加新的远程仓库关联
```
[[删除旧的远程仓库关联]]
git remote remove <name>
[[添加新的远程仓库关联]]
git remote add <name> <url>
```
远程仓库的名称推荐使用默认的名称`origin`

第三种：直接修改项目目录下的`.git`目录中的`config`配置文件

####17 git cherry-pick
在我们需要另一个分支的全部代码改动时，可以用`git merge`,但是另一种情况是，你只需要部分代码变动（某几个提交），这时可以采用 `git cherry-pick`

将指定的提交应用于其他分支：
```
git cherry-pick <commitHash>
```
上面的命令会将指定的提交`commitHash`，应用于当前分支。这会在当前分支产生一个新的提交，当然它们的哈希值会不一样。

举例来说，代码仓库有master和feature两个分支。
```
    a - b - c - d   Master
         \
           e - f - g Feature
```
现在将提交f应用到master分支。
```
# 切换到 master 分支
$ git checkout master

# Cherry pick 操作
$ git cherry-pick f
```
上面的操作完成以后，代码库就变成了下面的样子。
```
    a - b - c - d - f   Master
         \
           e - f - g   Feature
```
从上面可以看到，master分支的末尾增加了一个提交f。

`git cherry-pick`命令的参数，不一定是提交的哈希值，分支名也是可以的，表示转移该分支的最新提交。
```
$ git cherry-pick feature
```
上面代码表示将feature分支的最近一次提交，转移到当前分支。

##二、**补充：关于git的一些常见问题的解决**
###1.git解决non-fast-forward冲突
在git push的时候，有时候会出现这个错误。
**错误原因：**
文件冲突，本地的代码和远程Repository(存储库)中的文件个数不一致（即远程Repository中存在本地项目中不存在的文件）或本地项目不是在远程Repository代码的基础上修改的。
**解决方法：**
1). `git fetch origin debug`：获取远程分支debug的修改

2). `git merge origin debug`：合并远程分支debug

3). `git pull origin debug`：更新本地分支
###2.git关于commit过多的问题
由于git本地分支一直存在，那么在这个本地分支上进行过的commit就会一直存在于git log中，只有将本地分支删除之后git log才会没有本地分支之前所有的提交

**注意：**
1). 关于本地master分支的错误提交导致代码版本混乱问题：
先将所有的代码拷贝到本地其他文件夹下进行备份，再将master版本回退到很早之前的某个提交(`git reset --hard HEAD~n`，表示回退n个分支的内容)，再git pull更新master分支的内容。

2).`git push -f`最好只在自己的分支下提交，可以强行覆盖远程的分支。（有时可以对于某个未merge的远程分支的覆盖）

3). 在git提交的分支merge进入master分支之后，本地的分支需要提交的时候需要新的`commit -m`，因为是一个新的提交，不能够再次使用`commit --amend`的方式。

###3.git子仓的使用
git允许一个仓库作为另一个仓库的子仓，并且保持父项目和子项目相互独立。
####3.1 git初始添加仓库
1)`git submodule add <仓库地址>` ：
默认情况下，子模块会将子项目放到一个与仓库同名的目录，如果你想要放到其他地方，那么可以在命令结尾添加一个不同的路径。
`git submodule add git@github.com:pakastin/car.git`：将子项目放在当前git工作目录下的子文件夹car中。此时我们`git status`，发现，当前父目录中多了两个文件：
```
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   .gitmodules
        new file:   car
```
首先应当注意到新的`.gitmodules`文件。 该配置文件保存了项目 URL 与已经拉取的本地目录之间的映射；如果有多个子模块，该文件中就会有多条记录。**该文件也像`.gitignore`文件一样受到版本控制**,它会和该项目的其他部分一同被拉取推送。

2)之后，我们`git commit -am 'add submodule'`添加并提交，并`git push origin master`推送更改。

补充：
`git sunmodule add git@github.com:garnele007/SwiftOCR.git tools/OCR/test`表示新建文件夹`tools/OCR/test`，并将子模块下载并重命名为test。
####3.2 克隆含有子模块的项目：
有时候，我们在克隆一个git项目时，项目中本身就含有git子仓；此时我们不会将子仓中的内容克隆下来，只会克隆``.gitmodule`描述文件，此时需要进一步克隆子仓库文件。
1)
```
// 初始化本地配置文件
$ git submoudle init
// 从该项目中抓取所有数据并检出父项目中列出的合适的提交
$ git submodule update
```
2）更简单的方法：
 给`git clone`命令传递`--recurse-submodules`选项，它就会自动初始化并更新仓库中的每一个子模块， 包括可能存在的嵌套子模块。
```
$ git clone --recurse-submodules https://github.com/chaconinc/MainProject
```
3)如果已经克隆并且忘记`--recurse-submodules`：
可以运行`git submodule update --init`将git submodule init和git submodule update 合并成一步。如果还要初始化、抓取并检出任何嵌套的子模块， 使用简明的`git submodule update --init --recursive`

###4.github fork的分支使用

###4.git指定的远程分支操作
1.拉取指定远程分支
1.1直接拉取方式

```
git clone -b release_mlu100  git@github.com:Cambricon/CNStream.git
# git clone -b 远程分支名  仓库地址
```

1.2.本地已有远程仓库代码
```
[[查看远程分支]]
git branch -r
[[创建本地分支并关联]]
git checkout -b <要创建的本地分支名>  <origin/远程分支名>

[[已有本地分支，创建关联]]
git branch --set-upstream-to origin/远程分支名  本地分支名
[[拉取]]
git pull
```
2.推送本地分支到远程分支
`git push -u origin mlu100:release_mlu100`

`git push <远程主机名> <本地分支名>:<远程分支名>`
