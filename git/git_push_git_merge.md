### git push 和 git merge的区别

作者：波罗学

pull 根据不同的配置，可等于 fetch +  merge 或  fetch +  rebase。具体了解可继续读下去。

要理解它们的区别，首先我们需要明白的git的架构，它是分布式的版本管理系统。我画了张图，不仅仅涉及到git fetch和git pull，对整体理解也会很有帮助。如下：

![img](https://pic2.zhimg.com/50/v2-af3bf6fee935820d481853e452ed2d55_hd.jpg)

上图展示了git的整体架构，以及和各部分相关的主要命令。先说明下其中涉及的各部分。

**工作区(working directory)，**简言之就是你工作的区域。对于git而言，就是的本地工作目录。工作区的内容会包含提交到暂存区和版本库(当前提交点)的内容，同时也包含自己的修改内容。

**暂存区(stage area, 又称为索引区index)，**是git中一个非常重要的概念。是我们把修改提交版本库前的一个过渡阶段。查看GIT自带帮助手册的时候，通常以index来表示暂存区。在工作目录下有一个.git的目录，里面有个index文件，存储着关于暂存区的内容。git add命令将工作区内容添加到暂存区。

**本地仓库(local repository)，**版本控制系统的仓库，存在于本地。当执行git commit命令后，会将暂存区内容提交到仓库之中。在工作区下面有.git的目录，这个目录下的内容不属于工作区，里面便是仓库的数据信息，暂存区相关内容也在其中。这里也可以使用merge或rebase将**远程仓库副本**合并到本地仓库。图中的只有merge，注意这里也可以使用rebase。

**远程版本库(remote repository)，**与本地仓库概念基本一致，不同之处在于一个存在远程，可用于远程协作，一个却是存在于本地。通过push/pull可实现本地与远程的交互；

**远程仓库副本，**可以理解为存在于本地的远程仓库缓存。如需更新，可通过git fetch/pull命令获取远程仓库内容。使用fech获取时，并未合并到本地仓库，此时可使用git merge实现远程仓库副本与本地仓库的合并。git pull 根据配置的不同，可为git fetch + git merge 或 git fetch + git rebase。rebase和merge的区别可以自己去网上找些资料了解下。



### 参考链接

[知乎-git pull 和 git fetch的区别](https://www.zhihu.com/question/38305012)