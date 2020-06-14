## git push origin gadget 报错。

error: src refspec master does not match any

1. 这种错误一般是因为push的时候暂存区没有文件，确认下add的README.md存不存在
2. 可能是git add filename 的文件名不存在。导致提交commit的时候报错。error: src refspec master does not match any
3. 也可能提交一个空文件。

原因是使用了

`git checkout -b origin/master` 这样切换远程分支是存在问题的，这样是在本地新建了一个名为origin/master的分支，并没有切换到远程分支。因此这个commit的内容是本地分支的一次commit，远程的分支并没有进行改动，因此提交远程分支的过程中会出现提交文件为空文件，也就是说代码没有改动的错误。

`git chekout origin/master` 这样也有上面的问题

`git checkout -b master origin/master`这样是正确的解决方法。

git clone只能clone远程库的master分支，无法clone所有分支，解决办法如下：
1. 找一个干净目录，假设是git_work
2. cd git_work
3. git clone http://myrepo.xxx.com/project/.git ,这样在git_work目录下得到一个project子目录
4. cd project
5. git branch -a，列出所有分支名称如下：
remotes/origin/dev
remotes/origin/release
6. git checkout -b dev origin/dev，作用是checkout远程的dev分支，在本地起名为dev分支，并切换到本地的dev分支
7. git checkout -b release origin/release，作用参见上一步解释
8. git checkout dev，切换回dev分支，并开始开发。

* https://blog.csdn.net/liuhaomatou/article/details/54412071

