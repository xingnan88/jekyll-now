---
layout: post
title: pre-commit
---


# 安装

```shell
brew install pre-commit
```

# .pre-commit-config.yaml

在你的Flutter项目根目录下创建`.pre-commit-config.yaml`文件，并添加以下内容:

```yaml
repos: 
- repo: local 
- hooks: 
	- id: block-android-build-script name: Block android_build.sh script 
	- entry: sh -c 'if git diff --cached --name-only | grep -- "scripts/android_build.sh"; then exit 1; fi' 
	- language: system 
	- pass_filenames: false 
	- stages: [commit]
```

# 安装Git钩子

```shell
pre-commit install
```

# 每次提交自动检查

	禁止提交scripts/android_build.sh