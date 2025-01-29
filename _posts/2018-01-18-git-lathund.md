---
layout: post
title: Lathund git-kommandon
categories: programmering
tags: git
---

# Git Lazy dog

Här är några av de git-kommandon som jag inte kommer ihåg, men återkommer till tillräckligt ofta för att skriva ner dem här... 

## Commit changes to other branch than current

	git stash
	git checkout {other-branch}
	git stash pop
	git commit
	
## Merge develop branch into master
	git checkout master
	git pull origin master
	git merge develop
	git push origin master

## Move commit on master to develop
	git checkout develop
	git merge master
	git checkout master
	git reset –hard HEAD~1

## Cherry-pick
	git cherry-pick {commit id}
	
## Resolve merge conflicts
	git mergetool
	
## Force a stash pop
	git stash show -p | git apply && git stash drop

## Merge two repositories
[https://saintgimp.org/2013/01/22/merging-two-git-repositories-into-one-repository-without-losing-file-history/](https://saintgimp.org/2013/01/22/merging-two-git-repositories-into-one-repository-without-losing-file-history/)

## Import issues from one repo to another
[https://github.com/IQAndreas/github-issues-import](https://github.com/IQAndreas/github-issues-import)
	
## How to work with github and multiple accounts
[https://code.tutsplus.com/tutorials/quick-tip-how-to-work-with-github-and-multiple-accounts--net-22574](https://code.tutsplus.com/tutorials/quick-tip-how-to-work-with-github-and-multiple-accounts--net-22574)
	
## Create new repo from existing project

* Add .gitignore and if needed .gitattributes (see below)

		git init
		git add .
		git commit
		
* Go to [github.com](https://github.com/) and log in to your account
* Click the new repository button in the top-right
* Click the "Create repository" button
		
		git remote add origin https://github.com/{username}/{new_repo}
		git push -u origin master

## Remove remote branches that do not exist in remote
	git remote prune origin

## Remove local branches that do not exist in remote
**Note that new not yet pushed branches will be removed by this**

	git branch -vv | Select-String -Pattern ": gone]" | % { $_.ToString().Trim().Split(" ")[0]} | % {git branch -d $_}

## Contents of .gitignore
	.DS_Store
	.Trashes
	*.vbw
	*.csi
	*.exp
	*.lib
	*.lvw
	*.dca
	*.scc
	*.tmp
	Thumbs.db
	<output file name.exe or dll or whatever>
