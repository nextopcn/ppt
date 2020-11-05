# git 实践

1. 集中式：有几个主要branch, 例如master, dev, release, 所有开发人员在这几个分支上开发
2. 分布式：每个人有自己的分支, 开发完之后以PR的形式并入主要分支, 每个开发人员仅在自己的分支上开发, 要经常拉取主要分支的代码

## 公司内的一些git使用原则

1. 集中式使用git
2. 在开发中的项目，所有人员都在master上提交，例如balrog
3. 已经发布的项目, 禁止在master分支提交内容， 会按每个release日期创建branch, 例如gondor, release_20191218分支，当期要发布的内容提交到此分支，并合并到develop分支，当期不发布的内容，例如下次release 发布的内容，提交到develop分支。合并原则永远是release分支合并到develop分支，不能反过来。在当期release测试完部署后，根据develop分支创建下次release分支，当期release分支代码不允许有其他改动

## git 常用操作

1. reset & revert
2. merge & rebase
3. stash & unstash

## 集中式git使用实例

1. 在develop分支误提交某个文件，本应该提交到release分支
2. 在develop分支开发功能，要紧急解决一个在release分支的bug
3. release分支合并回develop分支时，解决冲突
4. revert某一commit（包含解决冲突）
5. 想在服务器部署某一个commit的代码
6. 项目间同步代码

## 集中式git的一些技巧

1. 使提交log更合理. 开发功能时，开启git commit --amend, 把当次commit累加到上次，待完整开发完一个功能，再push到远端
2. 集中式禁止使用rebase， 在分布式时会讲到原因
3. 使用git merge -no-ff, 创建一个合并记录的commit, 表明此次合并过程
4. 完全废弃本地提交， 使用git reset --hard origin/{branch}
5. 合理使用git reset --soft --hard --mixed

## 分布式git的一些技巧

1. 使用rebase拉取上游代码（如果有冲突如何解决）
2. 自己分支的多个提交合并成一个(整理要提PR的git log)
3. master分支使用git merge -no-ff合并来记录合并过程
4. 避免在master分支使用force push