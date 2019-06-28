# Git 简单实用命令

##### `mkdir test` 
创建一个名为 test 的文件夹

##### `cd test` 
进入 test 文件夹

##### `pwd` 
这个命令是显示当前所在的目录

##### `git init` 
创建 git 版本仓库

##### `git status` 
查看当前仓库的状态（有没有改变、未提交之类的）

##### `git diff` 
查看文件的变动，如 `git diff readme.txt` 就是查看 readme.txt 这个文件的改动

##### `git log` 
显示从最近到最远的提交日志（如果想看 log 的简便信息，可以使用 `git log --pretty=oneline` 这个命令）

##### `git reset --hard HEAD^` 
回退到上一个版本（在 Git 中，用 HEAD 表示当前版本，也就是最新的提交 1094adb...（注意我的提交 id 和你的肯定不一样），上一个版本就是 HEAD^，上上一个版本就是 HEAD^^，当然往上 100 个版本写 100个 ^ 比较容易数不过来，所以写成 HEAD~100）
如果想回到摸一个特定的版本，我们可以直接从 log 中看到版本的哈希值（没必要写全，取前面几位就行，一般 8 位），如 `git reset --hard e475afc9`

##### `git reflog` 
用来记录你每次的命令（比如当你有 5 个提交记录，这个时候 git log 就会有5条记录，当你使用 `git reset` 回到第一个版本的时候，这个时候你使用 git log 查看就只有第一次提交记录的 log 了，这个时候你再去想通过 hash 值回到第五个版本找不到这个 hash 了，这个时候就可以通过这个命令查看你的变动记录）
`git checkout -- readme.txt` 意思就是，把 readme.txt 文件在工作区的修改全部撤销，这里有两种情况：
一种是 readme.txt 自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
一种是 readme.txt 已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

##### `git checkout HEAD -- readme.txt` 
把暂存区的修改撤销掉（unstage），重新放回工作区（如果我们的修改已经提交到了暂存区，我们想回到之前未修改的工作区可以两步走，第一步使用 `git checkout HEAD -- readme.txt` 回到工作区。 第二步使用 `git checkout -- readme.txt` 回到修改前）

##### `git rm test.txt` 
从版本库中删除 test.txt 文件，删除完毕之后需要 commit 一下

##### `git remote add origin xxxxxx`
为你的项目添加远程仓库，关联了以后我们使用 `git push -u origin master` 命令将我们的本地内容推送到远程仓库中（把本地库的内容推送到远程，用 git push 命令，实际上是把当前分支master推送到远程。
由于远程库是空的，我们第一次推送 master 分支时，加上了-u 参数，Git 不但会把本地的 master 分支内容推送的远程新的 master 分支，还会把本地的 master 分支和远程的 master 分支关联起来，在以后的推送或者拉取时就可以简化命令， 就是说以后推送就是用 `git push origin master` 就可以了）

##### `git checkout -b test_dev`
创建并切换到 test_dev 分支，相当于 `git branch test_dev` 和 `git checkout test_dev` 这两个命令

##### `git merge test_dev` 
当前分支和 test_dev 分支合并

##### `git branch -d test_dev`
删除 test_dev 这个分支

##### `git log --graph --pretty=oneline --abbrev-commit`
查看分支合并图，分支合并冲突以后解决冲突，然后再 add 一下 commit 一下就行

##### `rm -rf .git`
删除git

##### `git stash` 
将当前的工作现场存储起来，一般我们不想提交，但是又得切到其他分支改 bug ，这个时候就可以用这个命令，当 bug 修复完毕，从其他分支切换回来的时候，使用 `git stash list` 查看你的存储情况，然后使用 `git stash pop` 回到你最初的工作现场

##### `git branch -D <name>`
丢弃一个没有被合并过的分支

##### `git remote`
查看远程库信息 git remote -v 查看更详细的信息

##### `git push origin chw_dev`
将分支推送到远程仓库

##### `git tag v1.0`
打一个 tag 如果想给以前的版本打 tag ，那就 git log 找到对应的提交记录， 使用个`git tag v1.0 xxxxxxx` 就可以了
 
##### `git tag`
查看所有的 tag， `git show v1.0` 查看 v1.0 这次 tag 的提交记录
 
##### `git tag -d v1.0`
删除一个标签，标签打错了可以删除，因为创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除。如果要推送某个标签到远程，使用命令 `git push origin <tagname>`
 
##### ` git push origin v1.0`
推送标签到远程
 
#####  `git push origin --tags`
 一次性推送所有的本地标签到远程
 
##### `git tags -d v1.0`
删除本地标签，如果本地标签已经推送到远程，想删除，就先用这个命令把本地的标签先删除，然后使用 `git push origin :refs/tags/v1.0` 这个命令
 
##### `cat .git/config`
git 的配置

![git head 原理](https://github.com/loveway/LearnBlog/blob/master/Notes/Git/picture/git_head.png)

![git 工作区与暂存区](https://github.com/loveway/LearnBlog/blob/master/Notes/Git/picture/git_stash.png)

## 小结
* 场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令 `git checkout -- file`

* 场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令 `git reset HEAD <file>`，就回到了场景1，第二步按场景1操作。

* 场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。

* 查看分支：`git branch`
创建分支：`git branch <name>`
切换分支：`git checkout <name>`
创建+切换分支：`git checkout -b <name>`
合并某分支到当前分支：`git merge <name>`
删除分支：`git branch -d <name>`
## SSH警告
> 当你第一次使用 Git 的 clone 或者 push 命令连接 GitHub 时，会得到一个警告：
The authenticity of host 'github.com (xx.xx.xx.xx)' can't be established.
RSA key fingerprint is xx.xx.xx.xx.xx.
Are you sure you want to continue connecting (yes/no)?
这是因为 Git 使用 SSH 连接，而 SSH 连接在第一次验证 GitHub 服务器的 Key 时，需要你确认 GitHub 的 Key 的指纹信息是否真的来自 GitHub 的服务器，输入 yes 回车即可。
 Git 会输出一个警告，告诉你已经把 GitHub 的 Key 添加到本机的一个信任列表里了：
Warning: Permanently added 'github.com' (RSA) to the list of known hosts.
这个警告只会出现一次，后面的操作就不会有任何警告了。
如果你实在担心有人冒充 GitHub 服务器，输入 yes 前可以对照 GitHub 的 RSA Key 的指纹信息是否与 SSH 连接给出的一致。

开始的时候，master 分支是一条线，Git 用 master 指向最新的提交，再用 HEAD 指向 master，就能确定当前分支，以及当前分支的提交点，每次提交，master 分支都会向前移动一步，这样，随着你不断提交，master 分支的线也越来越长：

<p align='center'>
<img src='https://github.com/loveway/LearnBlog/blob/master/Notes/Git/picture/git_master.png'>
</p>

当我们创建新的分支，例如 dev 时，Git 新建了一个指针叫 dev，指向 master 相同的提交，再把 HEAD 指向 dev ，就表示当前分支在 dev 上：

<p align='center'>
<img src='https://github.com/loveway/LearnBlog/blob/master/Notes/Git/picture/git_dev.png'>
</p>

你看，Git 创建一个分支很快，因为除了增加一个 dev 指针，改改 HEAD 的指向，工作区的文件都没有任何变化！

不过，从现在开始，对工作区的修改和提交就是针对 dev 分支了，比如新提交一次后，dev 指针往前移动一步，而 master 指针不变：

<p align='center'>
<img src='https://github.com/loveway/LearnBlog/blob/master/Notes/Git/picture/git_dev2.png'>
</p>

假如我们在 dev 上的工作完成了，就可以把 dev 合并到 master 上。Git 怎么合并呢？最简单的方法，就是直接把 master 指向 dev 的当前提交，就完成了合并：

<p align='center'>
<img src='https://github.com/loveway/LearnBlog/blob/master/Notes/Git/picture/git_merge.png'>
</p>

所以 Git 合并分支也很快！就改改指针，工作区内容也不变！

合并完分支后，甚至可以删除 dev 分支。删除 dev 分支就是把 dev 指针给删掉，删掉后，我们就剩下了一条 master 分支：

<p align='center'>
<img src='https://github.com/loveway/LearnBlog/blob/master/Notes/Git/picture/git_delete_dev.png'>
</p>
