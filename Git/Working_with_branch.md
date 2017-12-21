
## 서로 다른 두 Branch간 파일 비교하기
```
git diff <branch 1>..<branch 2> -- <compare file>
```


## 다른 Branch 파일 가져오기
[git-checkout-specific-files-from_another-branch](http://nicolasgallagher.com/git-checkout-specific-files-from-another-branch/)

The git-checkout manual page describes how the git checkout command is not just useful for switching between branches.

> When **paths** or **--patch** are given, git checkout does not switch branches. It updates the named paths in the working tree from the index file or from a named <tree-ish> (most often a commit)…The <tree-ish> argument can be used to specify a specific tree-ish (i.e. commit, tag or tree) to update the index for the given paths before updating the working tree.

The syntax for using git checkout to update the working tree with files from a tree-ish is as follows:
```
git checkout [-p|--patch] [<tree-ish>] [--] <pathspec>…
```
Therefore, to update the working tree with files or directories from another branch, you can use the branch name pointer in the git checkout command.

```
git checkout <branch_name> -- <paths>
```
As an example, this is how you could update your gh-pages branch on GitHub (used to generate a static site for your project) to include the latest changes made to a file that is on the master branch.

## On branch master
```
git checkout gh-pages
git checkout master -- myplugin.js
git commit -m "Update myplugin.js from master"
```