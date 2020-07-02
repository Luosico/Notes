# Git

## 本地操作
	git init  		初始化为为git仓库，默认生成主分支master，会生成一个.git的隐藏目录
	
	git status  	查看仓库状态
	
	git add file	提交文件到暂存区,file是要提交的文件
		add -A		提交所有变化
		add -u		提交被修改(modified)和被删除(deleted)文件，不包括新文件(new)
		add .		提交新文件(new)和被修改(modified)文件，不包括被删除(deleted)文件
	
	git commit -m 'info' 	将暂存区所有文件提交到仓库（当前分支），-m 表示提交的提示信息，每次提交必须附加这个
	
	git log			查看产生的所有commit记录
	
	git branch		查看当前分支情况
		branch name 新建一个名字为name的分支，分支内容和master一样
		branch -d branch_name  删除分支branch_name,前提是该分支代码已经合并到mater
		branch -D branch_name  强制删除分支branch_name
	
	git checkout  	
				 	切换branch或tag:
					branch_name  	  	切换到名为branch_name的分支
					branch origin/branch	把远程的branch分支pull到本地
					-b branch_name  	新建一个branch_name分支，再自动切换到这个分支
					-b branch  origin/branch	把远程的branch分支pull到本地,并切换到该分支
				 	tag_name		  	切换到名为tag_name的标签
					撤销:
					file_name			让这个文件回到最近一次git commit或git add时的状态
				 
	git merge/rebase branch_nam 	必须在mater分支操作，将分支branch_name合并到master分支上，可能会有冲突导致合并失败
	
	git tag 		查看历史tag记录，tag 是 Git 版本库的一个快照，指向某个 commit 的指针
		tag tag_name  在当前状态下新建名为tag_name的标签tag
	git reset	重置位置
				--hard	重置位置的同时，直接将 working Tree工作目录、 index 暂存区及 repository 都重置成目标Reset节点的內容,
						所以效果看起来等同于清空暂存区和工作区
				--soft	重置位置的同时，保留working Tree工作目录和index暂存区的内容，只让repository中的内容和 reset 目标节点保持一致
				--mixed(默认)	重置位置的同时，只保留Working Tree工作目录的內容，但会将Index暂存区和Repository中的內容更改和reset目标节点一致

## 在线操作
push：把本地仓库推到远程仓库
pull：把远程仓库拉倒本地仓库

	git clone git@github.com:Luosico/gitTest.git    从远程仓库下载一个仓库到本地，本地变成一个文件夹，不需要再init初始化为git仓库了
	
	git push <远程主机名> <本地分支名>:<远程分支名>	远程分支和本地分支保持一致，例如本地删除某个文件，提交后远程分支也会删除该文件
	git push origin master		将本地masert分支推到远程master分支，push前最好先pull，以免产生冲突，例如远程仓库有新文件而本地仓库没有
	git push -u origin master -f 	不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来


​	
​	git pull origin master 		将远程最新代码更新到本地，本地之前新增的不会被删除
​	
	git remote add origin git@github.com:Luosico/gitTest.git	本地添加一个远程仓库，origin为自定义远程仓库名字，可以任意取
	
	git remote -v				查看当前项目有哪些远程仓库
	
	git diff					比较工作目录和暂存区之间的差异，也就是修改后还没有暂存以来变化的内容
				HEAD ——commit版本         Index——staged版本
		diff --cached/staged	查看已经暂存起来的文件(staged)和上次提交时的快照之间(HEAD)的差异
		diff HEAD				显示工作区和HEAD（仓库）的差异
		diff HEAD -- ./lib 		显示当前工作目录下的lib目录与上次提交之间的差别(或者更准确的 说是在当前分支
		diff branch_name		显示当前工作区和分支branch_name的差别
		diff HEAD^ HEAD 		比较上次提交commit和上上次提交
		diff branch1..branch2	显示两个分支的差异
		diff branch1...branch2	显示它们共有父分支和branch2分支之间的差异
		diff --stat				统计一下有哪些文件被改动，有多少行被改动
	
	git stash					把当前分支所有没有 commit 的代码先暂存起来,这个时候你再执行 git status 你会发现当前分支很干净，
								几乎看不到任何改动， 你的代码改动也看不见了， 但其实是暂存起来了
		stash list				查看暂存区记录
		stash apply				还原出之前暂时保存的代码
		stash drop				删除暂存区的这次stash记录，每次只删除一条
		stash pop				拥有apply一样的功能，还会自动将这次的stash记录删除
		stash clear				清空所有暂存区的记录