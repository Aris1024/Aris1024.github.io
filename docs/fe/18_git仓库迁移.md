# git仓库迁移

背景:gerrit仓库迁移至gitlab仓库

gerrit也是git仓库

```sh
originGitUrl=你要迁移的仓库git地址
originGitDir=代码克隆下来存放的文件夹路径
targetGitUrl=要迁移的目标仓库git地址
# 拉取远程所有分支
git clone --mirror ${originGitUrl}
cd ${originGitDir}
git config --bool core.bare false
# 切换remote_url
git remote set-url origin ${targetGitUrl}
# 推送所有分支
git push --mirror origin
```

