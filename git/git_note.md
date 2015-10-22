#Git Note


##Introduction
---
Before start this section, we strongly recommend you to use this Quick Tool to get to know

* What is Git?
* Why we use Git?
* How to use Git?


	1. If you are contributing to a project for more than a few days, fork the repository to your own account.
	That makes you have the permission to push your code anytime without peer review. It’s also free to push your branches to the server.
	2. Add both org repository (slingstone/xxx) and your own repository to your git clone.
	
	$ git remote add wangkan git@git.corp.yahoo.com:wangkan/feeding.git
	$ git remote -v
	gerrit       ssh://wangkan@gerrit.pnt.corp.yahoo.com:29418/slingstone/feeding (fetch)
	gerrit       ssh://wangkan@gerrit.pnt.corp.yahoo.com:29418/slingstone/feeding (push)
	origin       git@git.corp.yahoo.com:slingstone/feeding.git (fetch)
	origin       git@git.corp.yahoo.com:slingstone/feeding.git (push)
	wangkan          git@git.corp.yahoo.com:wangkan/feeding.git (fetch)
	wangkan          git@git.corp.yahoo.com:wangkan/feeding.git (push)
	3. Keep in mind that git is distributed source control, so “git commit” only writes to your local repo. Anytime you want to save your progress on remote server, push to your own remote repo.
		$ git push wangkan S65548:S65548
	If you set your own branch tracking remote branch in your own repo, the push command could be simpler.
	
	$ git branch --set-upstream-to=wangkan/S65548 S65548
	Branch S65548 set up to track remote branch S65548 from wangkan.
	$ git branch --list -vv S65548
	* S65548 293cc5f [wangkan/S65548] update exploration topology to new batch structure
	$ git push

Basic Git Command
Revert Commit
	
	## reset head
	$ git reset HEAD^
	## reset to commit with keeping all the changes
	$ git reset --soft $(commit_id)
	## reset to commit without keeping all the changes
	$ git reset --hard $(commit_id)


Gerrit
In slingstone, we are using Gerrit for code review system, which will help us to ensure our product’s quality. If you are not familiar with Gerrit, it is better for you to read Gerrit User Guideline at first and follow to steps to setup your local env for gerrit.
	1. Set your username and email correctly. Gerrit requires your email to submit codereview for your account. Setup gerrit commit hooks before committing any changes to your git clone, that creates changeset-id for every changeset.
	
	$ scp -p -P 29418 gerrit.pnt.corp.yahoo.com:hooks/cmmit-msg .git/hooks/
	That creates Change-Id for every commits, including the commits pushed to your own repo. Without Change-Id, you may be rejected when you trying to merge to org repo throw gerrit.
	
	$ git log
	commit 293cc5f5a8e9cf72407b79b7d03a2a788314fd5e
	Author: wangkan <wangkan@yahoo-inc.com>
	Date:   Fri Nov 8 18:05:44 2013 +0800
	update exploration topology to new batch structure
	Change-Id: I26c943d511da52c942ffdd5b2ccde5ea1d7f23da
	commit fda05ca9ae484957f3555ff29904456a03168589
	Merge: 91a20c3 3321017
	Author: wangkan <wangkan@yahoo-inc.com>
	Date:   Fri Nov 8 13:50:50 2013 +0800
	merge ci changes and schema changes
	Change-Id: I3c7d3d1a28d81049208481f177ef7dfbe7fb72e0
	When you submit code review, gerrit creates review request for every changeset. Every commits that in the history of your commit, and not already in org repo, will be reviewed separately and pushed consequently.
	2. If you are rejected by gerrit(invalid email, no Change-Id, etc.), check the history, and ensure all commits are valid. You can use commit --amend to change the log message, author information for the last commit.
	If there is some commits before the last one rejected by gerrit, try rebase the changes. Rebase gives you a chance to reply the changes while rewrite the commit or logs. If you still do not have Change-Id after rebase, run --amend. That’s because the commit hook cannot deal with rebase commits.
Alias in git config
open ~/.gitconfig
	
	...
	[alias]
	b = branch
	br = branch
	ci = commit
	co = checkout
	d = diff
	di = diff
	s = status
	st = status
	p = pull
	rec = commit --amend
	recommit = commit --amend
	l = log
	lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --
	…

We can alias anything here.Furthermore, for our project, we can also let "git pg" be a alias of "git push gerrit HEAD:refs/for/master"
Tools
Git Completion
	install https://github.com/git/git/blob/master/contrib/completion/git-completion.bash
	source git-completion.bash in your bashrc
	In shell, [tab] can do some completion as possible as it can, like this:
	yafei@mustfruit-lm(dev) ~/git.migrate/searchplatform_serving $ git re[tab][tab]
	rebase         recommit       relink         repack         request-pull   revert         
	rec            reflog         remote         replace        reset
Vim/Emacs Syntax Plugin
These syntax files are enabled when we edit commit/rebase messages.
	https://github.com/git/git/tree/master/contrib/vim
	https://github.com/git/git/tree/master/contrib/emacs
Git Branch in PS
This tool will display status of git repo of current working directory.
	install https://raw.github.com/git/git/master/contrib/completion/git-prompt.sh
	source git-prompt.sh in your bashrc
	Then your PS1 variable that includes __git_ps1 '%s' should work fine.
	For example, in my ~/.bashrc:
	PS1='\[\033[01;32m\]\u@\h\[\033[01;34m\]$(__git_ps1 "(%s)")($YROOT_NAME) \w\$\[\033[00m\] '
	And I am in a branch named “dev”
	yafei@mustfruit-lm(dev) ~/git.migrate/searchplatform_serving $
Git Plugin for Eclipse
	Plugin URL: Git Eclipse Plugin
	1. install Egit
	2. Import projects from git
		○ you can change the branch in the Git Repository 
		○ you can apply different operations in the Git Repository (This is very similar to svn plugin)
Workflow
In this section, we will introduce you how we use git in our daily development. we hope you have been familiar with some basic git command before start this section.
There’s many different workflows under git, each have pros/cons. Get an agreement in your team, and keep use it. I (wangkan) like this A successful Git branching model
In our environment, the “development” branch could be your own master branch in your repo.
Setup Build Env
	• Pick a dev box or create one on openhouse.
	• Create a yroot:  yroot --create    <root name> [<image name>]
	• Restore build env: yinst restore -file build.state  (P.S.the state file is located in ${project}/buildinfo/buildyblocks/)
	• Build package: make devbuild
     the package is located in target/yinst/slingstone_searchplatform_serving-1.* , you can install it in your dev box to check your changes work right or not
Basic Workflow
Before start introduce this basic workflow, we want you to know that create standalone branches for every story, do not merge them to master until the story is ready for launch. It’s encouraged in git to frequently create/merge branches. And it’s easy and quick to do that.
As a contributor, you need to check out the code, change the code and submit the code. This is the most common process of our development cycle. As we mention above, in slingstone we use Git for source version control, and use Gerrit for code review.
	1. Check out the repository to your local file system.
	
	$ git clone $(git_repository_path)
	2. Create a branch locally
	
	## list branch locally
	$ git br
	 approriateness
	* master
	 unittest
	## create branch
	$ git co -b $(branch_name)
	## switch branch
	$ git co approriateness
	Switched to branch 'approriateness'
	3. Change the code
	
	## show status
	$ git st
	On branch master
	Your branch is up-to-date with 'origin/master'.
	Changes not staged for commit:
	 (use "git add <file>..." to update what will be committed)
	 (use "git checkout -- <file>..." to discard changes in working directory)
	modified:   config/qrs/endpoint-config-v10.xml
	modified:   config/queryprofiles/video_ht.xml
	4. Submit the code
	
	## update what will be committed
	$ git add $(file_to_commit)
	## discard changes in working directory
	$ git checkout -- <file>...
	## the commit msg session will appear to ask you to input the commit msg
	$ git ci 
	For example, you might get information like below, please input the commit msg, and then save the msg by :wq, if you don’t save the commit msg, the commit won’t be successfully.
	# Please enter the commit message for your changes. Lines starting
	# with '#' will be ignored, and an empty message aborts the commit.
	# On branch master
	# Your branch is up-to-date with 'origin/master'.
	#
	# Changes to be committed:
	#   modified:   config/qrs/endpoint-config-v10.xml
	#   modified:   config/queryprofiles/video_ht.xml
	#
	## you can also input the commit msg directly
	$ git ci -m “$(commit_msg)”
	5. Review the code
Before code review, it is very common that you should apply the the changes to the latest version. so we often switch to the master branch update the branch and rebase the change in your working branch to the master and then publish the code review. you can refer to rebase code change locally for details.
## use git review to commit the change to gerrit
$ git review HEAD -r $(reviewer_id)
## use git push to commit the change to gerrit, and then gerrit url will appear, you can login the url and select the reviewer to review the code
$ git push gerrit HEAD:refs/for/master
	6. After Review
	
	## Condition 1: Review Successully, the reviewer will give this commit 2 point, and may need you to merge the patch to the trunk brunch
	## Condition 2: However sometime, the reviewer will you some comment and need you to change the code, you can change the code according to the suggestion, and then add the code to stage, and finally use the command below, which will merge the code to the same commit and won’t create another code review
	$ git ci --amend


Scenarios
Rebase the latest change locally
According to the diagram above, when we do some changes in branch C, and the trunk has been change and merge by other branch. we should update all these changes to our branch so that we can keep updated with the trunk.
Method 1
## stash the local branch change
$ git add ./ && git stash save $(stash_name)
## switch to master branch
$ git br master
## update the master branch to latest with trunk
$ git pull --rebase
## merge master branch to your dev branch, this will create another commit which will merge master branch
$ git br $(dev_branch) && git merge master
## apply the stash buffer
$ git stash apply $(stash_name)
## after apply the stash buffer, you changed code has come back, then you can check in the code
Method 2
	
	## commit the code
	$ git ci -a -m “$(commit_msg)”
	## switch to master branch
	$ git br master
	## update the master branch to latest with trunk
	$ git pull --rebase
	## merge master branch and rebase the local change to your dev branch
	$ git br $(dev_branch) && git rebase master
Create the release branch
you can use ci job to achieve this. But if you want to know the details, please refer https://git.corp.yahoo.com/slingstone/tools_git_migration/blob/master/scripts/make_tag_branch.sh 
Create remote branch
Sometime, you may want to create a git branch remotely, here I will demonstrate how can we do that.
## show the logal repositor status
$ git remote show origin
* remote origin
 Fetch URL: git@git.corp.yahoo.com:binyu/TestRepo.git
 Push  URL: git@git.corp.yahoo.com:binyu/TestRepo.git
 HEAD branch: master
 Remote branch:
   master tracked
 Local branch configured for 'git pull':
   master merges with remote master
 Local ref configured for 'git push':
   master pushes to master (up to date)
`## create a branch, and then change and submit the code (optional)`
$ git co -b bugfix
## push current brancn to remote repository
$ git push origin bugfix
Counting objects: 6, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 344 bytes | 0 bytes/s, done.
Total 4 (delta 0), reused 0 (delta 0)
To git@git.corp.yahoo.com:binyu/TestRepo.git
* [new branch]      bugfix -> bugfix
  ## check the status of remote reposistory
$ git remote show origin
* remote origin
 Fetch URL: git@git.corp.yahoo.com:binyu/TestRepo.git
 Push  URL: git@git.corp.yahoo.com:binyu/TestRepo.git
 HEAD branch: master
 Remote branches:
   bugfix tracked
   master tracked
 Local branch configured for 'git pull':
## current branch don’t know how to pull from the remote branch we create
## if you want to know how to pull the changes from remote branch you can refer here
   master merges with remote master 
 Local refs configured for 'git push':
   bugfix pushes to bugfix (up to date) ## the new change will be push to the remote branch
   master pushes to master (up to date)

Pull Release Branch
	
	## fetch the release branch to local
	binyu@kitchenwee-lm 16:37:40 ~/slingstone/searchplatform_serving
	$ git fetch origin release_iteration87-20140624
	remote: Counting objects: 113, done.
	remote: Compressing objects: 100% (20/20), done.
	remote: Total 30 (delta 23), reused 15 (delta 8)
	Unpacking objects: 100% (30/30), done.
	From git.corp.yahoo.com:slingstone/searchplatform_serving
	* branch            release_iteration87-20140624 -> FETCH_HEAD
	  9bd77d5..ba21114  release_iteration87-20140624 -> origin/release_iteration87-20140624
	## create and switch to release branch
	binyu@kitchenwee-lm 17:04:28 ~/slingstone/searchplatform_serving
	$ git co -b release_iteration87-20140624 origin/release_iteration87-20140624
	Switched to a new branch 'release_iteration87-20140624'
	## update the master branch to latest with trunk
	binyu@kitchenwee-lm 17:07:10 ~/slingstone/searchplatform_serving
	$ git remote show origin
	* remote origin
	 Fetch URL: git@git.corp.yahoo.com:slingstone/searchplatform_serving.git
	 Push  URL: git@git.corp.yahoo.com:slingstone/searchplatform_serving.git
	 HEAD branch: master
	 Remote branches:
	   coverage                                                       tracked
	   dwh_logger                                                   tracked
	   master                                                          tracked
	   release                                                          tracked
	   release_iteration83-20140526                                    tracked
	   release_iteration86-20140617                                    tracked
	   release_iteration87-20140624                                    tracked
	   release_iterationDummy-20140103                             tracked
	   release_iterationtest_searchplatform_serving_assembly1-20140219 tracked
	   release_iterationtest_searchplatform_serving_assembly2-20140219 tracked
	   release_iterationtest_searchplatform_serving_assembly3-20140219 tracked
	 Local branches configured for 'git pull':
	   master                       merges with remote master
	   release_iteration87-20140624 merges with remote release_iteration87-20140624
	 Local refs configured for 'git push':
	   master                       pushes to master                       (local out of date)
	   release_iteration87-20140624 pushes to release_iteration87-20140624 (up to date)
Merge local branch with remote branch
if you want to create a new branch locally, and then want to merge the branch with remote branch
	
	## show the branch status
	$ git br -a
	* bugfix
	 master
	 remotes/origin/HEAD -> origin/master
	 remotes/origin/bugfix
	 remotes/origin/master
	## switch to master branch
	$ git br --set-upstream-to=origin/bugfix bugfix
	Branch bugfix set up to track remote branch bugfix from origin.
	## update the master branch to latest with trunk
	$ git remote show origin
	* remote origin
	 Fetch URL: git@git.corp.yahoo.com:binyu/TestRepo.git
	 Push  URL: git@git.corp.yahoo.com:binyu/TestRepo.git
	 HEAD branch: master
	 Remote branches:
	   bugfix tracked
	   master tracked
	 Local branches configured for 'git pull':
	   bugfix merges with remote bugfix
	   master merges with remote master
	 Local refs configured for 'git push':
	   bugfix pushes to bugfix (up to date)
	   master pushes to master (up to date)

Git push current branch to remote branch
According to the scenario I mentioned above, it is very common that 
	
	 Local refs configured for 'git push':
	   bugfix pushes to bugfix (up to date)(doesn’t appear)
	   master pushes to master (up to date)
So what we gonna do about it. Here is the solution
	
	$ git config remote.[remoteRepository].push [localBranch]:[remoteBranch]
	oro
	$ git push -u [localBranch] [remoteRepository]:[remteBranch]

Merge a commit from another branch
Since we will freeze code in the end of each iteration, I mean at 10 a.m. Tuesday CST. code freeze means that we will pull out a new release branch from trunk, then we will get package from this new release branch. Anyone who want to check in the new code need to get permission from Mingjun, Xiaohui and Vito. So as we know, it is almost inevitable that we will alway avoid this situation, since sometimes we will need to fix some bugs in this branch. 
Here is our common way to do this.
	1. fix the bug in the trunk branch, you can follow basic workflow to achieve it.
	2. checkout the release branch, you can follow here to achieve it
	3. fetch the patch and apply this patch to the release branch

	
	## checkout the release branch
	$ git co $(release_branch)
	## apply the command abovw
	$ git fetch ssh://binyu@gerrit.pnt.corp.yahoo.com:29418/slingstone/searchplatform_serving refs/changes/40/6740/1 && git cherry-pick FETCH_HEAD


Merge Multi-Commit into One
Sometimes we will commit several times when fix bug or add new feature, and we want to merge those commit into one, because they are related to each other. Here is the basic process.
For example below, you may want to merge the 197cd77 and dc5a8a0 to 61a3f47

* 197cd77 - (HEAD, origin/master, origin/HEAD, master) fix the problem (22 hours ago) <binyu>
* dc5a8a0 - add provider udsProfileYuidFinal  (2 days ago) <binyu>
* 61a3f47 - migrate the change (2 days ago) <binyu>
	1. rebase the change with interactive

	// reabase
	
	$ git rebase --interactive HEAD~3
	pick 61a3f47 migrate the change in http://gerrit.pnt.corp.yahoo.com/#/c/13646/2
	pick dc5a8a0 add provider udsProfileYuidFinal and yahoo.yinst.slingstone_cache_client
	pick 197cd77 fix the problem of dependency
	/ Rebase 9925cbf..197cd77 onto 9925cbf
	/
	/ Commands:
	#  p, pick = use commit
	#  r, reword = use commit, but edit the commit message
	#  e, edit = use commit, but stop for amending
	#  s, squash = use commit, but meld into previous commit
	#  f, fixup = like "squash", but discard this commit's log message
	#  x, exec = run command (the rest of the line) using shell
	#
	# These lines can be re-ordered; they are executed from top to bottom.
	#
	# If you remove a line here THAT COMMIT WILL BE LOST.
	#
	# However, if you remove everything, the rebase will be aborted.
	#
	# Note that empty commits are commented out
```

	2. squash the commits to the commit you pick
	
```
	pick 61a3f47 migrate the change in http://gerrit.pnt.corp.yahoo.com/#/c/13646/2
	squash dc5a8a0 add provider udsProfileYuidFinal and yahoo.yinst.slingstone_cache_client
	squash 197cd77 fix the problem of dependency
	# Rebase 9925cbf..197cd77 onto 9925cbf
	#
	# Commands:
	#  p, pick = use commit
	#  r, reword = use commit, but edit the commit message
	#  e, edit = use commit, but stop for amending
	#  s, squash = use commit, but meld into previous commit
	#  f, fixup = like "squash", but discard this commit's log message
	#  x, exec = run command (the rest of the line) using shell
	#
	# These lines can be re-ordered; they are executed from top to bottom.
	#
	# If you remove a line here THAT COMMIT WILL BE LOST.
	#
	# However, if you remove everything, the rebase will be aborted.
	#
	# Note that empty commits are commented out
	And then save the change, and the auto generated commit msg will appear. save the msg
	# This is a combination of 3 commits.
	# The first commit's message is:
	migrate the change in http://gerrit.pnt.corp.yahoo.com/#/c/13646/2
	Change-Id: I1aab3f3605801bffb5bac2cdc08758fdfc9b7e00
	# This is the 2nd commit message:
	add provider udsProfileYuidFinal and yahoo.yinst.slingstone_cache_client
	Change-Id: Ia295f137123b8b6f98fbfa39cef8b390d0f42880
	# This is the 3rd commit message:
	fix the problem of dependency
	# Please enter the commit message for your changes. Lines starting
	# with '#' will be ignored, and an empty message aborts the commit.
	# rebase in progress; onto 9925cbf
	# You are currently editing a commit while rebasing branch 'master' on '9925cbf'.
	#
	# Changes to be committed:
	#   modified:   buildinfo/release_definition/common/common.yidf
	#   modified:   buildinfo/release_definition/common/hattrick.videosp.dev.yidf
	#   modified:   buildinfo/release_definition/common/hattrick.videosp.int.yidf
	#   modified:   buildinfo/release_definition/common/hattrick.videosp.perf.yidf
	#   new file:   config/qrs/components.xml
	#   modified:   config/qrs/endpoint-config-v10.xml
	#   modified:   config/qrs/endpoint-config-v9.xml
	#   deleted:    config/qrs/qrs-hosts.xml
	#   modified:   config/qrs/video_search_chains.xml
	#   modified:   config/queryprofiles/video_screen.xml
	#   deleted:    config/queryprofiles/video_screen_user_hist.xml
	#   modified:   pom.xml
	#   modified:   src/main/java/com/yahoo/slingstone/searchplatform/searcher/HattrickRecommendationReasonSearcher.java
	#   modified:   src/main/yinst/slingstone_serving_video.yicf
```
	3. push the change
Tag the branch
Sometimes we need to tag the branch, for more details you can refer here. Here is the process
	1. tag the commit of a branch
	
	## list tag
	$ git tag --list
	## show tag info
	$ git tag -v $(tag_name)
	## tag the commit
	$ git tag -a $(tag_name) $(commit_id) -m “$(tag_msg)”
	## delete tag
	$ git tag -d $(tag_name)
	2. push the tag to trunk, git push (in slingstone’s repository, we don’t have permission to do that)
	git push origin ci-release-1.0
Collaborative for different box
It is very common that you will program locally and want to test your code. But you can not compile locally, since there might be some dependencies. So you have to restore a build env in the open house node, and apply the code change, then build the package which will be install in your test env. we will describe the details here.
	1. you can refer the setup build env for details
	2. create a patch of you local changes
	
	## add all the change you want to create a patch
	$ git add ./ 
	## create a patch
	$ git diff master > $(patch_name)
	3. apply you patch in the dev build env.
	
	## copy the patch file to your remote dev box
	$ scp $(patch_name) $(dev_box):$(path)
	## apply the patch to you dev box project
	$ cd $(project) && git apply $(path)/$(patch_name)
	4. Then you will see all the change has been apply to you remote dev box’s branch.

Collaborative for same commit
Another common usage is that A commit a change and push to gerrit, and after that B takes A’s task. How can B work on the same patch and modify the change, then push to the gerrit with same change id, which I mean with the same code review.
	1. update the local master to the latest state, please refer to Refresh
	2. go to the gerrit review page, click the checkout button, click the copy button on the right.

	3. go to master branch, paste the branch you have copy in #2. then you will get the following message:
	
	binyu@kitchenwee-lm 10:10:45 ~/slingstone/massmedia_serving
	$ git fetch ssh://binyu@gerrit.pnt.corp.yahoo.com:29418/slingstone/massmedia_serving refs/changes/55/14955/1 && git checkout FETCH_HEAD
	remote: Counting objects: 38, done
	remote: Finding sources: 100% (23/23)
	remote: Total 23 (delta 9), reused 23 (delta 9)
	Unpacking objects: 100% (23/23), done.
	From ssh://gerrit.pnt.corp.yahoo.com:29418/slingstone/massmedia_serving
	* branch            refs/changes/55/14955/1 -> FETCH_HEAD
	Note: checking out 'FETCH_HEAD'.
	You are in 'detached HEAD' state. You can look around, make experimental
	changes and commit them, and you can discard any commits you make in this
	state without impacting any branches by performing another checkout.
	If you want to create a new branch to retain commits you create, you may
	do so (now or later) by using -b with the checkout command again. Example:
	 git checkout -b new_branch_name
	HEAD is now at 6eea0f1... blend video with section throttle
	4. Then according to the msg above, you have to check out the branch, and then you will see the commit is on the top of the your new branch.
	5. Change the code, and commit the change with
	
	## commit the change to the head commit
	$ git commit -a -amend
	## push the commit to the gerrit
	$ git push gerrit HEAD:refs/for/master

Reference
	• Quick Tool
	• https://www.atlassian.com/en/git/tutorial
	• http://gitref.org/
	• http://git-scm.com/book
	• http://yo/gerrit-user
	• https://code.google.com/p/gerrit/
	• Git Community Book
Pro-Git Chinese Version | Pro-Git Mobi Version
