---
title: govendor-quickstart
tags:
  - go
categories:
  - go
date: 2018-12-25 13:16:56
---

# 安装
`go get -u -v github.com/kardianos/govendor`


# 使用
```
0. 安装
go get -u -v github.com/kardianos/govendor

1. 初始化
cd "my project in GOPATH"
govendor init

2. 添加依赖
# Add existing GOPATH files to vendor.
govendor add +external
```

# 详细教程
wiki:https://github.com/kardianos/govendor
```
Quick Start, also see the FAQ
# Setup your project.
cd "my project in GOPATH"
govendor init

# Add existing GOPATH files to vendor.
govendor add +external

# View your work.
govendor list

# Look at what is using a package
govendor list -v fmt

# Specify a specific version or revision to fetch
govendor fetch golang.org/x/net/context@a4bbce9fcae005b22ae5443f6af064d80a6f5a55
govendor fetch golang.org/x/net/context@v1   # Get latest v1.*.* tag or branch.
govendor fetch golang.org/x/net/context@=v1  # Get the tag or branch named "v1".

# Update a package to latest, given any prior version constraint
govendor fetch golang.org/x/net/context

# Format your repository only
govendor fmt +local

# Build everything in your repository only
govendor install +local

# Test your repository only
govendor test +local

Sub-commands
	init     Create the "vendor" folder and the "vendor.json" file.
	list     List and filter existing dependencies and packages.
	add      Add packages from $GOPATH.
	update   Update packages from $GOPATH.
	remove   Remove packages from the vendor folder.
	status   Lists any packages missing, out-of-date, or modified locally.
	fetch    Add new or update vendor folder packages from remote repository.
	sync     Pull packages into vendor folder from remote repository with revisions
  	             from vendor.json file.
	migrate  Move packages from a legacy tool to the vendor folder with metadata.
	get      Like "go get" but copies dependencies into a "vendor" folder.
	license  List discovered licenses for the given status or import paths.
	shell    Run a "shell" to make multiple sub-commands more efficient for large
	             projects.

	go tool commands that are wrapped:
	  `+<status>` package selection may be used with them
	fmt, build, install, clean, test, vet, generate, tool
Status
Packages can be specified by their "status".

	+local    (l) packages in your project
	+external (e) referenced packages in GOPATH but not in current project
	+vendor   (v) packages in the vendor folder
	+std      (s) packages in the standard library

	+excluded (x) external packages explicitly excluded from vendoring
	+unused   (u) packages in the vendor folder, but unused
	+missing  (m) referenced packages but not found

	+program  (p) package is a main package

	+outside  +external +missing
	+all      +all packages
Status can be referenced by their initial letters.

+std same as +s
+external same as +ext same as +e
+excluded same as +exc same as +x
Status can be logically composed:

+local,program (local AND program) local packages that are also programs
+local +vendor (local OR vendor) local packages or vendor packages
+vendor,program +std ((vendor AND program) OR std) vendor packages that are also programs or std library packages
+vendor,^program (vendor AND NOT program) vendor package that are not "main" packages.
```
