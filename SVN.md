# SVN

## git cheatcheet

When you try to initiate a new repository locally, you should

```shell
git init
```

When you try to clone a repository from the remote repository, you should

```shell
git clone [URL]
git remote add [origin] [URL]
```

For every `push`

```shell
git pull [origin] [master]
git add [files]
git commit -m "message"
git push [origin] [master]
```

To check which branch you are now at

```shell
git branch
```

To check remote branches

```shell
git branch -r
```

New branch

```shell
git branch [new branch name]
```

Check out to another branch

```shell
git checkout [branch name]
```

