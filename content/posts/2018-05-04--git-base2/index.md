---
title: Git-基础2
subTitle: 关于 commit、rebase 使用，fork 机制
category: Git
cover: tool.png
---

## git diff

工作区：就是你在电脑里能看到的目录
暂存区：git add 是把文件到暂存区
本地仓库：工作区有一个隐藏目录 .git，这个不算工作区，而是 Git 的版本库

git diff 进行对比时不会对比新创建的文件。

`git diff branchName` 比较当前分支与 branchName 分支
`git diff branchName1 branchName2` 比较 branchName1 分支与 branchName2 分支
`git diff develop origin/develop`  比较本地 develop 分支和远程 develop 分支的差别
`git diff --staged`  比较暂存区与 HEAD
`git diff HEAD` 比较工作区与 HEAD
`git diff` 比较工作区与暂存区

## git commit

- `git commit`  提交所有添加到索引库中、或从索引库删除的文件
  输入 i 进入编辑状态，第一行添加的内容作为 summary 描述, 隔一个空行，再添加的内容作为 description 描述，按 esc 退出编辑状态，输入 shift + :，输入 wq，保存并提交。

- `git commit -m "summary 描述"`  提交所有添加到索引库中、或从索引库删除的文件
  简化版提交命令，-m 添加 summary 描述，不能添加 description 描述。

- `git commit -a`  提交所有已在索引库中发生了修改的文件，新添加的文件不会被提交
  输入 i 进入编辑状态，第一行添加的内容作为 summary 描述, 隔一个空行，再添加的内容作为 description 描述，按 esc 退出编辑状态，输入 shift + :，输入 wq，保存并提交。

- `git commit --fixup [commitId]`  提交所有添加到索引库中、或从索引库删除的文件
  提交一个新的 commit，summay 为指定 commitId 的 summay 添加了 “fixup!” 前缀。在执行 `rebase --autosquash` 命令时针对该次 commit 的操作默认为 fixup，本次提交信息会被清除，不会进入编辑提交信息的状态。

- `git commit --squash [commitId]`  提交所有添加到索引库中、或从索引库删除的文件
  提交一个新的 commit，summay 为指定 commitId 的 summay 添加了 squash!” 前缀。在执行 `rebase --autosquash` 命令时针对该次 commit 的操作默认为 squash，会进入编辑提交信息的状态。

## git rebase

类似于 git merge，将指定分支合并到当前分支。区别是 git merge 会创建一次新的提交记录，git rebase 不会，它会把当前分支里的每个提交取消掉，并且把它们临时保存为补丁(这些补丁放到".git/rebase"目录中)，然后把当前分支更新为最新的指定分支，最后再把保存的这些补丁应用到当前分支上。

“-i” 是指交互模式，就是说你可以干预 rebase 这个事务的过程，包括设置 commit message，删除 commit 等等。
“--autosquash” 是自动合并 commit 的参数。

`git rebase -i --autosquash [branchName]` 当前在特性分支上执行，合并 branchName 分支到当前分支

`git rebase -i --autosquash [branchName] [feature]` 无需在特性分支上执行

`git rebase --abort` 终止 rebase 的行动，回到 rebase 前的状态

特性分支合并了最新的主分支后，原有的 commit 是逐个应用到当前分支的，当正常执行的 commit 与已有代码发生冲突时，需执行下面三部，直到 rebase 成功。

① 解决冲突

② `git add .`

③ `git rebase --continue`

## git hooks

- `pre-commit` 在键入提交信息前运行，被用来检查即将提交的快照，当从该挂钩返回非零值时，Git 放弃此次提交，但可以用 git commit --no-verify 来忽略
- `prepare-commit-msg` 在提交信息编辑器显示之前，默认信息被创建之后运行，该挂钩对通常的提交来说不是很有用，只在自动产生默认提交信息的情况下有作用，如提交信息模板、合并、压缩和修订提交等
- `commit-msg` 接收一个参数，此参数是包含最近提交信息的临时文件的路径，如果该挂钩脚本以非零退出，Git 放弃提交，可以用来在提交通过前验证项目状态或提交信息
- `post-commit` 挂钩在整个提交过程完成后运行，作为通知之类使用的
- `pre-rebase` 在衍合前运行，脚本以非零退出可以中止衍合的过程，可以使用这个挂钩来禁止衍合已经推送的提交对象
- `post-rewrite` 被那些会替换提交记录的命令调用，如 git commit --amend 和 git rebase
- `post-checkout` git checkout 成功运行后，该挂钩会被调用，可以用来为你的项目环境设置合适的工作目录
- `post-merge` 在merge命令成功执行后，该挂钩会被调用，可以用来在 Git 无法跟踪的工作树中恢复数据
- `pre-push` 会在 git push 运行期间，更新了远程引用但尚未传送对象时被调用，可以在推送开始之前，用它验证对引用的更新操作

`cd .git/hooks` 可查看 git 所有 hooks

## git fork 机制

使用场景：如果想要修改他人 github 的项目，直接 clone 代码到本地是不能 pull 或 push 的，所以要使用 fork，先把他人代码 fork 到自己的 github 仓库，然后 clone 到本地修改，然后再提交到自己 github 仓库，这时候想要把修改的代码提交给他人的话，就可以在自己的 github 上提交 pull request，等其他人看到后就可以把代码做一个合并。当他人的代码有更新的时候，也可以将更新同步到自己的 github 仓库。

#### github 的 fork

在 github 上，fork 一个项目，如 lodash，如果远程有更新，红框内会有提示，点击 Pull request，base fork 选则本地仓库，head fork 选则 lodash 仓库，可将 lodash 的更新通过一个 pull request 合并到自己的仓库中。也可以通过交换 base fork 和 head fork 提交一个 pull request 到 lodash 仓库。

{% img https://zhulichao.github.io/2018/05/04/git-base2/fork-1.png fork 一个项目 %}

{% img https://zhulichao.github.io/2018/05/04/git-base2/fork-2.png 提交 pull request %}

{% img https://zhulichao.github.io/2018/05/04/git-base2/fork-3.png git remote -v %}

#### git 命令

在 github 创建一个仓库命名为 local-lodash 后，执行下面操作：
`git clone https://github.com/lodash/lodash.git local-lodash` 克隆远程 lodash 仓库到本地
`git remote remove origin` 删除远程仓库地址
`git remote add origin https://github.com/zhulichao/local-lodash.git` 添加自己的远程仓库地址
`git remote add upstream https://github.com/lodash/lodash.git` 添加上游远程仓库地址
`git push -u origin master` 初次提交到自己的远程仓库
`git pull upstream master` 同步上游仓库 lodash 的更新
`git push origin` 提交到自己的远程仓库
`git remote -v`  查看远程仓库地址，pull 和 push 可以是两个不同的地址

{% img https://zhulichao.github.io/2018/05/04/git-base2/fork-4.png git remote -v %}

这种方式与在 github 上执行 fork 还是有区别的，如图所示，没有上面图片中红框中的内容，在 github 上是无法通过 pull request 同步远程 lodash 更新的，当然也不能提交 pull request 到远程 lodash，只能通过上面的命令 `git pull upstream master` 同步远程 lodash 的更新。如果你恰好也是远程 lodash 的贡献者，也可以通过 `git push upstream master` 直接提交 commit 到 远程 lodash。

{% img https://zhulichao.github.io/2018/05/04/git-base2/fork-5.png github 上查看 %}
{% img https://zhulichao.github.io/2018/05/04/git-base2/fork-6.png 提交 pull request %}
