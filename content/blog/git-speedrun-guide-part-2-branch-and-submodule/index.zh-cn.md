---
title: "Git 速通指南 | Part 2: 分支与子模块"
---

在 [Git 速通指南 | Part 1: 基础](../git-speedrun-guide-part-1-basics/) 中, 我们已经学习了如何创建 Git 储存库, 如何 `add` 和 `commit` 更改, 以及如何进行 `push` 以及 `pull` 操作. 本文将继续介绍 Git 的分支与子模块的使用.

## 0. 准备工作

请保证本地有一个与远程仓库关联的 Git 储存库. 如果没有, 请参考 [Git 速通指南 | Part 1: 基础](../git-speedrun-guide-part-1-basics/) 第 4 节创建一个远程储存库, 然后直接 `clone` 至本地.

## 1. 分支

### 1.1. 创建和切换分支

在本地 Git 储存库下打开终端, 输入以下命令创建一个新分支 `feature`:

```bash
git switch -c feature
```


> 💡 当基于当前分支创建了一个新分支时, 新分支会继承当前分支的所有 `commit` 记录; 可以认为新分支克隆了当前分支.


查看本地储存库有哪些分支:

```bash
git branch
```

查看远程储存库有哪些分支:

```bash
git branch -r
```

通过 `git branch` 你应该观察到当前我们在本地的 `feature` 分支上; 如果没有, 通过以下命令切换到 `feature` 分支:

```bash
git switch feature
```

在当前路径下创建一个新文件 `feature.txt` 并写入 "This is a feature file."; 将更改 `add` 和 `commit` 到本地储存库:

```bash
echo "This is a feature file." > feature.txt
git add feature.txt
git commit -m "Add feature file"
```

这次 `commit` 记录仅在 `feature` 分支上, 并不会影响 `main` 分支. 切换到 `main` 分支, 可以看到 `feature.txt` 文件并不存在:

```bash
git switch main
ls  # List files in current directory
```

### 1.2. `merge` 分支

由于 `feature` 分支上的更改已经 `commit`, 我们希望将 `feature` 分支上的更改合并到 `main` 分支上. 切换到 `main` 分支, 并输入以下命令:

```bash
git switch main
git merge feature  # Merge feature to main
```

此时在 `main` 分支下, 通过 `ls` 命令可以查看到已有 `feature.txt` 文件. 从 `commit` 记录来看, `main` 分支上的 `commit` 记录已经包含了 `feature` 分支上的所有 `commit` 记录.

### 1.3. 冲突处理

冲突发生在 `merge` 过程中. 例如, 当 `main` 分支和 `feature` 分支上的同一个文件的同一行都被修改时, Git 无法自动决定如何合并这两个更改, 从而产生冲突.

接下来我们尝试模拟冲突产生过程并处理冲突.

首先切换到 `main` 分支, 修改 `feature.txt` 文件, 在 "This is a feature file." 这行末尾添加 "mmm", 最终文件内容为 "This is a feature file.mmm". `add` 和 `commit` 更改.

接着切换到 `feature` 分支, 修改 `feature.txt` 文件, 在 "This is a feature file." 这行末尾添加 "fff", 最终文件内容为 "This is a feature file.fff". `add` 和 `commit` 更改.

切换回 `main` 分支, 输入以下命令合并 `feature` 分支:

```bash
git merge feature
```

终端提示如下:

```
Auto-merging feature.txt
CONFLICT (content): Merge conflict in feature.txt
Automatic merge failed; fix conflicts and then commit the result.
```

说明合并过程中发生了冲突. 此时查看 `feature.txt` 文件, 会发现文件内容如下:

```
<<<<<<< HEAD
This is a feature file. mmm
=======
This is a feature file. fff
>>>>>>> feature
```

从上面的内容可以看出, `<<<<<<< HEAD` 到 `=======` 之间的内容是 `main` 分支上的内容 (即当前分支 `HEAD`), `=======` 到 `>>>>>>> feature` 之间的内容是 `feature` 分支上的内容. 我们需要手动解决这个冲突.

解决冲突的方式非常简单, 手动选择保留哪部分内容即可. 比如把上面的内容全部删除, 只保留 `feature` 分支的更改, 最终文件内容为 "This is a feature file.fff". 

在 `main` 分支上 `add` 和 `commit` 更改后, 就完成了 `merge` `feature` 分支并解决冲突的所有过程.

> 💬 从远程 Git 储存库 `pull` 新版本时, 也可能出现冲突情况. 此时也只需手动处理冲突即可.

### 3.4. 删除本地分支

当一个分支的工作已经完成, 可以删除该分支. 删除分支前, 请确保该分支上的所有更改已经 `commit` 并合并到其他分支上. 删除本地分支 `feature`:

```bash
git branch -d feature
```

### 3.5. 创建远程分支

本地新创建的分支 `feature` 并不会自动同步到远程仓库. 如果需要将本地分支同步到远程仓库, 需要手动 `push` 该分支:

```bash
git push origin feature:feature
```

此时在远程仓库中应该可以看到一个新的分支 `feature`.

### 3.6. 删除远程分支

删除远程分支 `feature`:

```bash
git push origin --delete feature
```

## 2. 子模块

子模块是 Git 仓库中的一个仓库. 通过子模块, 我们可以将一个 Git 仓库嵌套到另一个 Git 仓库中. 子模块的使用可以帮助我们更好地管理项目的依赖关系.

### 2.1. 添加子模块

在本地 Git 储存库下打开终端, 输入以下命令添加一个子模块:

```bash
git submodule add <URL> <RelativePath>
```

其中 `<URL>` 是子模块的 Git 仓库地址, `<RelativePath>` 是子模块在本地储存库中的相对路径. 

### 2.1. 初始化子模块

直接 `clone` 包含子模块的 Git 仓库时, 子模块并不会被自动初始化. 因此需要手动初始化子模块:

```bash
git submodule update --init --recursive
```

### 2.3. 更新子模块

如果子模块的远程仓库发生了更改, 需要手动更新子模块:

```bash
git submodule update --remote --recursive
```

### 2.4. 删除子模块

删除子模块需要以下几个步骤:

1. 删除 `.gitmodules` 文件中子模块的配置.
2. 删除 `.git/config` 文件中子模块的配置.
3. 删除子模块的目录.
4. 删除 `.git/modules/<SubmodulePath>` 文件夹.