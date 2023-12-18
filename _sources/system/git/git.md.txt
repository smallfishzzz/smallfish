# git tips

## set ssh 、http/https proxy for git


### ssh protocol

**edit the file ~/.ssh/config**

```bash
Host github.com
  User git
  ProxyCommand nc -x localhost:12123 %h %p
```

### http/https protocol

Enter the following command on the terminal.

```bash
> global effective
> git config --global http.proxy http://localhost:12123  %h %p
> git config --global https.proxy https://localhost:12123  %h %p
> locally effective
> git config http.proxy http://localhost:12123  %h %p
> git config https.proxy https://localhost:12123  %h %p
```

> and you can also edit  ~/.gitconfig (global)  or  .git/config (git source directory)



## reference connection

### github

> [Use Proxy for Git/GitHub](https://gist.github.com/coin8086/7228b177221f6db913933021ac33bb92)

