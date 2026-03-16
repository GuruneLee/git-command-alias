## alias script
```
[alias]
	# ─── Quick commit & push ───
	lg = "!f() { git add -A && git commit -m \"$@\" && git push; }; f"

	# ─── Log ───
	l  = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%C(bold blue)<%an>%Creset' --abbrev-commit
	la = "!git l --all"

	# ─── Basic shortcuts ───
	a   = add
	aa  = add *
	al  = "!git add $(git status -s | fzf -m | awk '{print $2}')"
	cm  = commit -m
	s   = status
	ss  = status -s
	co  = checkout
	sh  = show
	br  = branch
	pl  = pull
	ps  = push
	rb  = rebase
	df  = diff
	st  = stash
	sp  = stash pop
	ru  = remote -v update
	b0  = "!git branch | awk '/^\\*/{print $2}'"

	# ─── Interactive (fzf) ───
	col = "!git co $(git branch | grep -v '^\\*' | sed 's/^[+* ]*//' | fzf --preview \"git l {1}\" | awk '{print $1}')"
	shl = "!git show $(git log | grep commit | fzf --preview \"git show {2}\" | awk '{print $2}')"

	# ─── Branch tools (git bb) ───
	blist        = "!git branch | grep -v '^\\*' | sed 's/^[+* ]*//' | tr -d ' '"
	blist-merged = "!git branch --merged | grep -v '^\\*\\|\\<master$\\|\\<main$' | sed 's/^[+* ]*//' | tr -d ' '"
	cleanbranch  = "!git branch -d $(git blist-merged)"

	bselect = "! f() { \
		_height=$(stty size | awk '{print $1}'); \
		git br | sed 's/^[+* ]*//' | egrep -v '^$' | fzf --preview \"git l {1} | head -n $_height\"; \
	}; f"

	brl = "! f() { \
		git checkout --track $(git branch -avv | grep -v '^\\*' | fzf | awk '{print $1}'); \
	}; f"

	bclean = "! # Search and delete merged branches;\n\
		git branch -d $(git blist-merged); \
		for branch in $(git blist) ; do \
			echo \"\nSearch :\\033[32m $branch \\033[0m\"; \
			if [ $(git l | grep $branch -c) -gt 0 ]; then \
				git l | egrep \"Merge.*$branch\" -C 2; \
				read -p \"Delete $branch? [y|n] \" -r; \
				REPLY=${REPLY:-\"n\"}; \
				if [ $REPLY = \"y\" ]; then \
					git branch -D $branch; \
					echo \"\\033[32m$branch \\033[0mhas been\\033[31m deleted\\033[0m.\n\"; \
				fi; \
			fi; \
		done \n\
	"

	bb = "! f() { \n\
		if [ $# = 0 ]; then \
			git checkout $(git branch -vv | grep -v '^\\*' | fzf | awk '{print $1}'); \
		elif [ $1 = 'help' ]; then \
			echo 'git bb           : Select and checkout branch'; \
			echo 'git bb c, clean  : Clean merged branches (+ worktrees)'; \
			echo 'git bb d, D      : Delete selected branches (D: force)'; \
			echo 'git bb l, list   : List branches excluding current'; \
			echo 'git bb m, merged : List merged branches'; \
		elif [ $1 = 'clean' -o $1 = 'c' ]; then \
			for _b in $(git branch --merged | grep -v '^\\*\\|\\<master$\\|\\<main$' | sed 's/^[+* ]*//' | tr -d ' '); do \
				_wtpath=$(git worktree list | grep \"\\[$_b\\]\" | awk '{print $1}'); \
				if [ -n \"$_wtpath\" ]; then \
					git worktree remove \"$_wtpath\" --force; \
					echo \"\\033[33mWorktree for '$_b' removed.\\033[0m\"; \
				fi; \
			done; \
			git bclean; \
		elif [ $1 = 'd' -o $1 = 'D' ]; then \
			git branch -$1 $(git bb list | fzf -m); \
		elif [ $1 = 'list' -o $1 = 'l' ]; then \
			git blist; \
		elif [ $1 = 'merged' -o $1 = 'm' ]; then \
			git blist-merged; \
		else \
			git bb help; \
		fi; \
	}; f"

	# ─── Tag ───
	mvtag = "!f() { \
		if [ $# -lt 2 ]; then \
			echo \"Usage: git mvtag <old> <new> [new message]\"; return 1; \
		fi; \
		old=$1; new=$2; shift 2; \
		if [ $# -eq 0 ]; then \
			msg=$(git for-each-ref refs/tags/$old --format=\"%(contents)\"); \
			git tag -a \"$new\" -m \"$msg\" \"$old\" && git tag -d \"$old\"; \
		else \
			git tag -a \"$new\" -m \"$*\" \"$old\" && git tag -d \"$old\"; \
		fi; \
	}; f"

	# ─── Worktree for multi-agent sessions (git wt) ───
	wtl = worktree list

	wt = "! f() { \n\
		if [ $# = 0 ]; then \
			git wt help; \
		elif [ $1 = 'help' -o $1 = 'h' ]; then \
			echo 'git wt a,  add <branch> [base]  : Add worktree'; \
			echo 'git wt co, checkout             : Move worktree branch to current directory'; \
			echo 'git wt m,  merge                : Merge worktree branch into current'; \
			echo 'git wt rb, rebase [base]        : Rebase worktree branch onto base'; \
			echo 'git wt d,  remove [branch]      : Remove worktree'; \
			echo 'git wt o,  open                 : Open worktree in IntelliJ'; \
			echo 'git wt l,  list                 : List all worktrees'; \
		elif [ $1 = 'a' -o $1 = 'add' ]; then \
			shift; \
			if [ $# = 0 ]; then \
				echo 'Usage: git wt add <branch> [base-ref]'; \
				return 1; \
			fi; \
			_branch=$1; \
			_base=${2:-HEAD}; \
			_wtdir=\"../worktrees/$_branch\"; \
			if git show-ref --verify --quiet \"refs/heads/$_branch\"; then \
				echo \"Branch '$_branch' exists. Checking out in worktree...\"; \
				git worktree add \"$_wtdir\" \"$_branch\"; \
			else \
				echo \"Branch '$_branch' not found. Creating from $_base...\"; \
				git worktree add -b \"$_branch\" \"$_wtdir\" \"$_base\"; \
			fi; \
			echo \"Worktree ready at: $(cd \"$_wtdir\" && pwd)\"; \
		elif [ $1 = 'co' -o $1 = 'checkout' ]; then \
			_selected=$(git worktree list --porcelain | grep '^branch' | sed 's|branch refs/heads/||' | grep -v \"^$(git branch --show-current)$\" | \
				fzf --header='Checkout worktree branch here' \
					--preview=\"echo '--- diff stat ---'; git diff --stat $(git branch --show-current)...{}; echo ''; echo '--- log ---'; git log --oneline $(git branch --show-current)..{} | head -20\" \
					--preview-window=right:50%); \
			if [ -z \"$_selected\" ]; then \
				echo 'Cancelled.'; return 0; \
			fi; \
			_wtpath=$(git worktree list | grep \"\\[$_selected\\]\" | awk '{print $1}'); \
			if [ -n \"$_wtpath\" ]; then \
				git worktree remove \"$_wtpath\" --force; \
				echo \"Worktree removed.\"; \
			fi; \
			git checkout \"$_selected\"; \
		elif [ $1 = 'm' -o $1 = 'merge' ]; then \
			_current=$(git branch --show-current); \
			_selected=$(git worktree list --porcelain | grep '^branch' | sed 's|branch refs/heads/||' | grep -v \"^$_current$\" | \
				fzf --header=\"Merge into '$_current'\" \
					--preview=\"echo '--- diff stat ---'; git diff --stat $_current...{}; echo ''; echo '--- log ---'; git log --oneline $_current..{} | head -20\" \
					--preview-window=right:50%); \
			if [ -z \"$_selected\" ]; then \
				echo 'Cancelled.'; return 0; \
			fi; \
			echo \"Merging '$_selected' into '$_current'...\"; \
			if git merge \"$_selected\" --no-edit; then \
				echo \"\\033[32mMerge successful.\\033[0m Cleaning up worktree...\"; \
				_wtpath=$(git worktree list | grep \"\\[$_selected\\]\" | awk '{print $1}'); \
				if [ -n \"$_wtpath\" ]; then \
					git worktree remove \"$_wtpath\"; \
					echo \"Worktree removed.\"; \
				fi; \
				read -p \"Delete branch '$_selected'? [y|n] \" -r; \
				REPLY=${REPLY:-\"n\"}; \
				if [ \"$REPLY\" = \"y\" ]; then \
					git branch -d \"$_selected\"; \
					echo \"\\033[31mBranch '$_selected' deleted.\\033[0m\"; \
				fi; \
			else \
				echo \"\\033[31mMerge failed.\\033[0m Resolve conflicts first.\"; \
			fi; \
		elif [ $1 = 'rb' -o $1 = 'rebase' ]; then \
			shift; \
			_base=${1:-$(git remote show origin | grep 'HEAD branch' | awk '{print $NF}')}; \
			_selected=$(git worktree list --porcelain | grep '^branch' | sed 's|branch refs/heads/||' | grep -v \"^$(git branch --show-current)$\" | \
				fzf --header=\"Rebase onto '$_base'\" \
					--preview=\"git log --oneline $_base..{} | head -20\" \
					--preview-window=right:50%); \
			if [ -z \"$_selected\" ]; then \
				echo 'Cancelled.'; return 0; \
			fi; \
			_wtpath=$(git worktree list | grep \"\\[$_selected\\]\" | awk '{print $1}'); \
			echo \"Rebasing '$_selected' onto '$_base'...\"; \
			git -C \"$_wtpath\" rebase \"$_base\"; \
		elif [ $1 = 'd' -o $1 = 'remove' ]; then \
			shift; \
			if [ $# = 0 ]; then \
				_selected=$(git worktree list --porcelain | grep '^branch' | sed 's|branch refs/heads/||' | grep -v \"^$(git branch --show-current)$\" | \
					fzf --header='Select worktree to remove' \
						--preview=\"echo '--- diff stat ---'; git diff --stat $(git branch --show-current)...{}; echo ''; echo '--- log ---'; git log --oneline $(git branch --show-current)..{} | head -20\" \
						--preview-window=right:50%); \
				if [ -z \"$_selected\" ]; then \
					echo 'Cancelled.'; return 0; \
				fi; \
			else \
				_selected=$1; \
			fi; \
			_wtpath=$(git worktree list | grep \"\\[$_selected\\]\" | awk '{print $1}'); \
			if [ -n \"$_wtpath\" ]; then \
				git worktree remove \"$_wtpath\" --force; \
				echo \"Worktree for '$_selected' removed.\"; \
			else \
				echo \"No worktree found for '$_selected'.\"; \
			fi; \
		elif [ $1 = 'o' -o $1 = 'open' ]; then \
			_selected=$(git worktree list | fzf --header='Open in IntelliJ' | awk '{print $1}'); \
			if [ -z \"$_selected\" ]; then \
				echo 'Cancelled.'; return 0; \
			fi; \
			idea \"$_selected\"; \
		elif [ $1 = 'l' -o $1 = 'list' ]; then \
			git worktree list; \
		else \
			echo \"Unknown: $1\"; git wt help; \
		fi; \
	}; f"
```
