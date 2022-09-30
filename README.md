## alias script
```
[alias]
        l = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%C(bold blue)<%an>%Creset' --abbrev-commit
        la = "! git l --all"
        ru = remote -v update
        s = status
        ss = status -s
        ps = push
        pl = pull

        co = checkout
        col = "! git co $(git branch | grep -v '^\\*' | fzf --preview \"git l {1}\" | awk '{print $1}')"

        cm = commit -m
        a = add
        aa = add *

        shl = "!git show $(git log | grep commit | fzf --preview \"git show {2}\" | awk '{print $2}')"
        #dfl = "!git diff $(git log | grep commit | fzf -m --preview \"git show {2}\" | awk '{print $2}')"
```
