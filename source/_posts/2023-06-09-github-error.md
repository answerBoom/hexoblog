---
title: github error
tags:
  - github
categories:
  - github
mathjax: true
description: git commit -m "first commit"错误
abbrlink: '70556169'
swiper_index: 3
date: 2023-06-09 21:38:51
---
### $ git commit -m "first commit"错误
丢失或跳过`git add .`或`git commit`可能导致此错误：
```powershell
$ git commit -m "first commit"
On branch main
Initial commit
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .gitignore
        README.md
        _config.butterfly.yml
        _config.yml
        gulpfile.js
        package.json
        repoPic/
        scaffolds/
        source/
        themes/
nothing added to commit but untracked files present (use "git add" to track)
```
### $ git push -u origin main错误

```powershell
error: src refspec main does not match any
error: failed to push some refs to 'https://github.com
```
此错误最可能的原因是所有文件都未跟踪且未添加。 git add --all 如果你想添加所有文件或者你可以有选择地添加文件。 然后 git commit -m "Initial comment", git push origin master。
### 解决方法
要修复它，请重新初始化并遵循正确的顺序：
```powershell
git init
git add --all/git add .
$ git remote add origin https://github.com
git commit -m "Initial comment"
git push -u origin main
```

参考：[message-src-refspec-master-does-not-match-any-when-pushing-commits-in-git](https://stackoverflow.com/questions/4181861/message-src-refspec-master-does-not-match-any-when-pushing-commits-in-git)
