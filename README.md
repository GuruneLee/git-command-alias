## alias script
```
[alias]
	l = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%C(bold blue)<%an>%Creset' --abbrev-commit
	la = "! git l --all"
	ru = remote -v update

	cm = commit -m

	a = add
	aa = add *
	## add list for selection (`tab` to select)
	al = "!git add $(git status -s | fzf -m | awk '{print $2}')"

	s = status
	ss = status -s # short
	co = checkout
	col = "! git co $(git branch | grep -v '^\\*' | fzf --preview \"git l {1}\" | awk '{print $1}')"

	sh = show
	shl = "!git show $(git log | grep commit | fzf --preview \"git show {2}\" | awk '{print $2}')"
	
	br = branch	
	pl = pull
	ps = push
	rb = rebase
	df = diff

	# print current branch name
	b0 = "!git branch | awk '/^\\*/{print $2}'" 
	## clean merged branch
	cleanbranch = "!git branch -d $(git branch --merged | grep -v '^\\*\\|\\<master$')"
	# advanced
	blist = "!git branch | grep -v '^\\*'"
	blist-merged = "!git branch --merged | grep -v '^\\*\\|\\<master$'"
	bclean = "! # Search and delete merged branches;\n\
		git branch -d $(git blist-merged); \
		for branch in $(git blist) ; do \
			echo \"\nSearch :\\033[32m $branch \\033[0m\"; \
			if [ $(git lg | grep $branch -c) -gt 0 ]; then \
				git lg | egrep \"Merge.*$branch\" -C 2; \
				read -p \"Delete $branch? [y|n] \" -r; \
				REPLY=${REPLY:-"n"}; \
				if [ $REPLY = \"y\" ]; then \
					git branch -D $branch; \
					echo \"\\033[32m$branch \\033[0mhas been\\033[31m deleted\\033[0m.\n\"; \
				fi; \
			fi; \
		done \n\
	"
	bselect = "! # select branch with preview; \n\
		f() { \
			_height=$(stty size | awk '{print $1}');\
			git br | egrep -v '^\\*' | fzf --preview \"git lg {1} | head -n $_height\"; \
		}; f"

	## branch helper
	bb = "! # Branch tools. Type 'git bb help' ; \n\
        f() { \n\
            if [ $# = 0 ]; then \
                git checkout $(git branch -vv | grep -v '^\\*' | fzf | awk '{print $1}'); \
            elif [ $1 = 'help' ]; then \
                echo 'git bb           : Select and checkout branch'; \
                echo 'git bb c, clean  : Clean all merged branches'; \
                echo 'git bb d, D      : Delete seleted branches(D: force)'; \
                echo 'git bb l, list   : List branches excluding the current branch'; \
                echo 'git bb m, merged : List merged branches excluding the current and master branches'; \
            elif [ $1 = 'd' -o $1 = 'D' ]; then \
                git branch -$1 $(git bb list | fzf -m); \
            elif [ $1 = 'clean' -o $1 = 'c' ]; then \
                git bclean; \
            elif [ $1 = 'list' -o $1 = 'l' ]; then \
                git branch | grep -v '^\\*'; \
            elif [ $1 = 'merged' -o $1 = 'm' ]; then \
                git branch --merged | grep -v '^\\*\\|\\<master$'; \
            else \
                git bb help; \
            fi; \
        }; f"
```
