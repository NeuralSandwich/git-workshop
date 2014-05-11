# Git Workshop

## Configuration

Preface, using requirements

### User Settings

Before being able to commit files to any of the git repositories locally git requires two pieces of information, your name and email address. This is used to identify you as the person who commited the code. These settings can be set on a per repository basis or globally (for all repositories) using the following commands.

```
git config --global user.name "Your Name"
git config --global user.email "your.name@email.server"
```

### SSH Settings

When working with remote repositories there are two methods of connecting to the remote git server, using HTTPS and using SSH. When pushing to a git server when connecting with HTTPS you will be asked for your username and password for the server, this can be tedious to repeatedly enter when simply uploading your changes, by using SSH keys you no longer need to enter your credentials. This approach uses OpenSSH, it is also possible to use alternatives but this is not covered here.

#### 1: Check for an existing SSH key

```
cd ~/.ssh
ls -a
```

If you see the file `id_rsa.pub` then you already have an SSH key and can skip to Section 3.

#### 2: Create a new SSH key

We will be using the `ssh-keygen` tool to create a new SSH key with the RSA cryptosystem, your email is also required in this step.

```
ssh-keygen -t rsa -C "your.name@email.server"
```

The first prompt will ask where the SSH key should be stored, the default location is preferable so that Git will be able to find the key without issue, simply press enter.

The you will be prompted to enter a passphrase (and confirmation) required to use the SSH key, if you are working on a trusted system just press enter twice so that no passphrase is required.

You should now have two new files in your `~/.ssh` directory: your private key `id_rsa` which must be kept safe, and public key `id_rsa.pub` which can be provided to a Git server to enable SSH authentication.

#### 3: Adding SSH key to you Git server

This step depends on who your chosen Git hosting provider is, for sites such as [GitHub](https://github.com) and [Bitbucket](httsp://bitbucket.org) and self hosted installations of [GitLab](https://gitlab.com) has a settings section which allows you to add your public key to the server quickly and easily.

### Mergetool

Solving merge conflicts can be a difficult and tedious task which can be made much easier with a visual merge tool such as [meld](http://meldmerge.org/). Meld is available for [Windows](http://sourceforge.net/projects/meld-installer/) as well as Unix based systems. Other merge tools are available and are usually a personal preference.

In order to setup Git to take advantage of an installed merge tool on Windows we need to perform the following configuration. Take care to replace the path to your merge tool with the correct path to the exectuable.

```
git config --global mergetool.meld.path "C:\Program Files (x86)\Meld\bin\meld\meld.exe"
git config --global merge.tool meld
```

### Help

#### Autocorrect

It is a common problem to type a Git command incorrectly and recieve a message such as the following.

```
git: 'stauts' is not a git command. See 'git --help'.

Did you mean this?
        status
```

If this is a common problem for you then it is possible to enable a helper setting which will make an attempt to guess which command you were attempting to use.

```
git config --global help.autocorrect 1
```

### Cross Platform Issues

#### Line Endings (Optional)

When working on cross platforms projects, or using the same hard drive in combination with different operating systems, problems arise with the way text is stored on disk. Specifically Windows and Unix like systems use different characters to define the end of a line, Windows uses CRLF and Unix like systems use LR.

The following setting tells Git to automatically deal with line endings globally.

```
git config --global core.autocrlf true
```

If you checkout a commit before setting this option, you may be asked to commit changes to files you have not changed. In this situation the easist apporach is to remove the files which were checked out with the wrong line endings then perform a hard reset on the repository, this will checkout the files with the correct line endings. Be careful not to loss any changes you have made.

### Useful Aliases (Optional)

When working on projects with many branches it can be confusing to determine the order of commits and which branches were merges and where. With the following alias it is possible show this information on the command line.

First we need to create an alias called `tree` (you can choose your own alias name) which will be the shorthand for the command. Note that this is a long command which is why having an alias is preferable.

```
git config --global alias.tree "log --graph --abbrev-commit --decorate --date=relative --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all"
```

Once this is done we can use the alias with the command `git tree`.

## Tutorials

### Stashing

The stash is a temporary place where local changes can stored whilst other operations are performed on the repository, such as pulling the lastest commits or changing branch before a commit. To stash changes is simple.

```
git stash
```

You should see the following if the stash was successful.

```
Saved working directory and index state WIP on master: <commit> <message>
```

Otherwise if you have not made any changes you will see the following, please make a change to a file in order to continue.

```
No local changes to save
```

Now that we have stashed some changes we can do whichever task for which the stash was required. It is possible to list all the current stashes.

```
git stash list
```

Which will show output in the following form.

```
stash@{0}: WIP on master: <commit> <message>
... possibly more stashes ...
```

Stashes are stored in a stack, so the latest stash will have the `stash@{0}` identifier, any older stashes will have the form `stash@{n}` where `n` is the position in the stack.

When the time comes when you want to unstash the changes there are two options, apply the changes or pop the changes which does an apply then a drop. We will start with apply.

```
git stash apply stash@{0}
```

A successful apply will show the output of a call to `git status` and your changes will be applied to the repository. If we call `git stash list` you will see that your stash is still avaialble.

The alternative is to use `git stash pop` which will apply the changes then drop the stash from the stash list. First we need to perform a hard reset to avoid a merge conflict.

```
git reset --hard HEAD
git stash pop
git stash list
```

If all goes well the stash should be applied to the working directory and the displayed list will no longer show the stash. Lets add the stash once again, then explore how to view the contents of a stash.

```
git stash
git stash list
```

To view the contents of a stash we can use `git stash show stash@{0}` to see the insertions and deletions, however it is usually more useful to see the changes in patch form.

```
git stash show -p stash@{0}
```

Finally if we decide we no longer require the changes containted within it. The stash can be dropped as follows.

```
git stash drop stash@{0}
git stash list
```