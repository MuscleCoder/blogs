# Git Q&A (1)

## Cannot update paths and switch to branch at the same time


You can get this error in the context of, e.g. a Travis build that, by default, checks code out with `git clone --depth=50 --branch=maste`. To the best of my knowledge, you can control --depth via .travis.yml but not the --branch. Since that results in only a single branch being tracked by the remote, you need to independently update the remote to track the desired remote's refs.

Before:

```
$ git branch -a
* master
remotes/origin/HEAD -> origin/master
remotes/origin/master
```

The fix:

```
$ git remote set-branches --add origin branch-1
$ git remote set-branches --add origin branch-2
$ git fetch
```
After:

```
$ git branch -a
* master
remotes/origin/HEAD -> origin/master
remotes/origin/branch-1
remotes/origin/branch-2
remotes/origin/master
```

### Reference
http://stackoverflow.com/questions/22984262/cannot-update-paths-and-switch-to-branch-at-the-same-time


## How to change Git Editor

	git config --global core.editor /usr/bin/vim

## Multiple GitHub Accounts & SSH Config

change the ssh config

```
Host me.github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/me_rsa

Host work.github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/work_rsa
```

## Multiple GitHub Accounts & SSH Config

## Git specify a branch such as 'master'.

git push --set-upstream origin master

## Git checkout remote branch

   *  git fetch
   *  git checkout -b old_rcv origin/old_rcv
   
