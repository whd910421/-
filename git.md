用到的Linux命令行 
ls -la
cd $path
..
touch $fileName
mkdir $floderName
rm -rf 递归删除
vi $fileName 


Git
分布式管理系统

SVN集中式管理系统

安装：Win 下载Git 软件
Linux sudo apt-get install git
Mac Xcode 自带

git config --global user.name "Your Name"
git config --global user.email "email@example.com"

提交的时候会显示

创建 版本库 git init 生成.git 文件夹

git status 查看git当前状态 （红色 绿色）

git add 添加文件 

git commit 提交修改 -m -a

git diff $fileName 查看修改 

git reset --hard $commitId 回退到之前的某个版本 这个commitid通过git log 来获得

git checkout -- file 在没有add之前，在工作区 就算是revert

git reset HEAD file 在add之后，文件已经加载到了缓存区，可从暂存区出来 如果是在AS中进行操作，它会把两个步骤合二为一

git remote add origin $remoteGit 添加远程仓库 先有本地库 再有远程库
git pull git push 会让建立连接，建立完连接可以直接pull或者push

git clone $remoteGit 从远程库clone 会自动使master分支相联系

git branch -ra 查看本地和远程分支

git branch $branchName 创建分支

git checkout $branchName 切换到这个分支

git checkout -b $branchName 创建并且换到 如果后面有 <remote>/<branch> 则会直接与其关联

git merge $branchName 合并某分支到当前分支

git branch -d $branchName 删除分支

出现冲突了的时候对出现冲突的分支文件进行修改，再提交

.gitignore alias 