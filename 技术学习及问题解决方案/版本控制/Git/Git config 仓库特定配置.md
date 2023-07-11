> git管理的项目文件在根目录下回存在 `.git` 隐藏文件夹，其下存在 `config` 配置文件。

> 一般git存在全局配置，若想对某个仓库进行特定配置，需修改其 `config文件`

## config文件case:

```bash
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
	ignorecase = true
	precomposeunicode = true
[submodule]
	active = .
[remote "origin"]
	url = git@github.com:Gavid/Gavid.github.io.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
[user]
	name = Gavid
	email = 460528401@qq.com
```

## config 配置项解读

.git 文件夹下的 `config` 文件是 Git 仓库的配置文件，它包含了仓库特定的配置信息。下面是一些常见的 `.git/config` 配置项及其含义的解读：

1. `[core]` 部分：这个部分包含了与 Git 核心功能相关的配置选项，如：
   - `repositoryformatversion`：仓库的格式版本。
   - `filemode`：是否跟踪文件权限变化。
   - `bare`：是否为裸仓库。
   - `ignorecase`：是否忽略文件名大小写。
2. `[remote "<remote-name>"]` 部分：这个部分定义了远程仓库的配置，其中 `<remote-name>` 是远程仓库的名称。这部分包含了与远程仓库相关的配置项，如：
   - `url`：远程仓库的 URL。
   - `fetch`：默认的拉取配置。
3. `[branch "<branch-name>"]` 部分：这个部分定义了分支的配置，其中 `<branch-name>` 是分支的名称。这部分包含了与分支相关的配置项，如：
   - `remote`：关联的远程仓库。
   - `merge`：默认的合并分支。
4. `[user]` 部分：这个部分包含了与用户身份验证相关的配置项，如：
   - `name`：用户名。
   - `email`：用户邮箱地址。
5. `[alias]` 部分：这个部分定义了 Git 命令的别名。你可以在这里设置自定义的 Git 命令缩写。

除了上述常见的配置项之外，你可能会看到其他特定于仓库的配置项或扩展配置项，这取决于仓库的使用和相关的 Git 扩展。

