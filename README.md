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
	ru  = remote -v update # git fetch 대신 씀
	b0  = "!git branch --show-current"

	# ─── Interactive (fzf) ───
	col = "!f() { \
		_b=$(git branch --format='%(refname:short)' | grep -v \"^$(git branch --show-current)$\" | fzf --preview \"git l {1}\"); \
		[ -n \"$_b\" ] && git co \"$_b\"; \
	}; f"
	shl = "!git show $(git log | grep commit | fzf --preview \"git show {2}\" | awk '{print $2}')"

	# ─── Branch helpers ───
	blist        = "!git branch --format='%(refname:short)' | grep -v \"^$(git branch --show-current)$\""
	blist-merged = "!git branch --format='%(refname:short)' --merged | grep -v \"^$(git branch --show-current)$\" | grep -v '^master$' | grep -v '^main$'"
	cleanbranch  = "!git branch -d $(git blist-merged)"

	bselect = "! f() { \
		_height=$(stty size | awk '{print $1}'); \
		git branch --format='%(refname:short)' | fzf --preview \"git l {1} | head -n $_height\"; \
	}; f"

	brl = "! f() { \
		git checkout --track $(git branch -avv | grep -v '^\\*' | fzf | awk '{print $1}'); \
	}; f"

	# ─── Branch clean (git bb c) ───
	bclean = "! f() { \
		unset GIT_DIR; \
		_current=$(git branch --show-current); \
		if [ -z \"$_current\" ]; then \
		echo '\\033[31mDetached HEAD. Abort.\\033[0m'; return 1; \
		fi; \
		_base=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@'); \
		if [ -z \"$_base\" ]; then \
		if git show-ref --verify --quiet refs/heads/main 2>/dev/null; then \
			_base=main; \
		elif git show-ref --verify --quiet refs/heads/master 2>/dev/null; then \
			_base=master; \
		else \
			echo '\\033[31mCannot detect base branch. Set origin/HEAD or pass manually.\\033[0m'; return 1; \
		fi; \
		fi; \
		echo \"\\033[36mBase branch: $_base\\033[0m\"; \
		echo ''; \
		\
		_merged=$(git branch --merged \"$_base\" | sed 's/^[+* ]*//' | grep -vE \"^($_current|$_base)$\"); \
		_rest=$(git branch --no-merged \"$_base\" | sed 's/^[+* ]*//' | grep -vE \"^($_current|$_base)$\"); \
		\
		if [ -n \"$_merged\" ]; then \
		echo \"$_merged\" | while read -r _b; do \
			_wtpath=$(git worktree list | awk -v b=\"[$_b]\" '$NF == b {print $1}'); \
			if [ -n \"$_wtpath\" ]; then \
			if [ -z \"$(git -C \"$_wtpath\" status --porcelain 2>/dev/null)\" ]; then \
				git worktree remove \"$_wtpath\" 2>/dev/null; \
			else \
				echo \"  \\033[33m⚠ Skipped worktree (dirty): $_b → $_wtpath\\033[0m\"; \
				continue; \
			fi; \
			fi; \
			if git branch -d \"$_b\" 2>/dev/null; then \
			echo \"  \\033[32m✓ Deleted (merged): $_b\\033[0m\"; \
			else \
			_err=$(git branch -d \"$_b\" 2>&1); \
			echo \"  \\033[31m⚠ Branch delete failed: $_b — $_err\\033[0m\"; \
			fi; \
		done; \
		echo ''; \
		else \
		echo '\\033[90mNo merged branches found.\\033[0m'; \
		echo ''; \
		fi; \
		\
		if [ -z \"$_rest\" ]; then \
		echo '\\033[90mNo remaining branches.\\033[0m'; return 0; \
		fi; \
		echo \"\\033[36mRemaining branches (Tab to select, Enter to confirm, Esc to cancel):\\033[0m\"; \
		_selected=$(echo \"$_rest\" | fzf -m \
		--header='Tab: select / Enter: confirm delete / Esc: cancel' \
		--preview='echo \"\\033[33m── Recent commits ──\\033[0m\"; \
			git log --color --oneline --graph {1} -15; \
			echo \"\"; \
			echo \"\\033[33m── Changed files (vs '\"$_base\"') ──\\033[0m\"; \
			git diff '\"$_base\"'...{1} --stat 2>/dev/null || echo \"(no diff available)\"' \
		--preview-window=right:60%:wrap \
		); \
		if [ -z \"$_selected\" ]; then \
		echo 'Cancelled.'; return 0; \
		fi; \
		echo ''; \
		echo \"\\033[31m── Branches to delete ──\\033[0m\"; \
		echo \"$_selected\" | while read -r _b; do \
		_wtpath=$(git worktree list | awk -v b=\"[$_b]\" '$NF == b {print $1}'); \
		if [ -n \"$_wtpath\" ]; then \
			echo \"  $_b  \\033[33m(worktree: $_wtpath)\\033[0m\"; \
		else \
			echo \"  $_b\"; \
		fi; \
		done; \
		echo ''; \
		read -r -p 'Proceed? [y/N] ' _ans < /dev/tty; \
		_ans=${_ans:-N}; \
		if [ \"$_ans\" != 'y' ] && [ \"$_ans\" != 'Y' ]; then \
		echo 'Cancelled.'; return 0; \
		fi; \
		echo ''; \
		echo \"$_selected\" | while read -r _b; do \
		_wtpath=$(git worktree list | awk -v b=\"[$_b]\" '$NF == b {print $1}'); \
		if [ -n \"$_wtpath\" ]; then \
			if [ -z \"$(git -C \"$_wtpath\" status --porcelain 2>/dev/null)\" ]; then \
			git worktree remove \"$_wtpath\" 2>/dev/null; \
			echo \"  \\033[33m⊘ Worktree removed: $_b\\033[0m\"; \
			else \
			echo \"  \\033[31m✗ Worktree dirty, skipped: $_b → $_wtpath\\033[0m\"; \
			continue; \
			fi; \
		fi; \
		if git branch -d \"$_b\" 2>/dev/null; then \
			echo \"  \\033[32m✓ Deleted: $_b\\033[0m\"; \
		else \
			_err=$(git branch -d \"$_b\" 2>&1); \
			echo \"  \\033[33m⚠ Not fully merged: $_b — $_err\\033[0m\"; \
			read -r -p \"    Force delete $_b? [y/N] \" _force < /dev/tty; \
			_force=${_force:-N}; \
			if [ \"$_force\" = 'y' ] || [ \"$_force\" = 'Y' ]; then \
			git branch -D \"$_b\" 2>/dev/null; \
			echo \"  \\033[31m✓ Force deleted: $_b\\033[0m\"; \
			else \
			echo \"  \\033[90m  Kept: $_b\\033[0m\"; \
			fi; \
		fi; \
		done; \
		echo ''; \
		echo '\\033[32mDone.\\033[0m'; \
	}; f"

	# ─── Branch tools (git bb) ───
	bb = "! f() { \
		if [ $# = 0 ]; then \
			_b=$(git branch --format='%(refname:short)' | grep -v \"^$(git branch --show-current)$\" | fzf --preview 'git log {}'); \
			[ -n \"$_b\" ] && git checkout \"$_b\"; \
		elif [ \"$1\" = 'help' ]; then \
			echo 'git bb           : Select and checkout branch'; \
			echo 'git bb c, clean  : Clean branches (auto-delete merged + fzf select remaining)'; \
			echo 'git bb d, D      : Delete selected branches (D: force)'; \
			echo 'git bb l, list   : List branches excluding current'; \
			echo 'git bb m, merged : List merged branches'; \
		elif [ \"$1\" = 'clean' ] || [ \"$1\" = 'c' ]; then \
			git bclean; \
		elif [ \"$1\" = 'd' ] || [ \"$1\" = 'D' ]; then \
			_sel=$(git blist | fzf -m); \
			[ -n \"$_sel\" ] && git branch -\"$1\" $_sel; \
		elif [ \"$1\" = 'list' ] || [ \"$1\" = 'l' ]; then \
			git blist; \
		elif [ \"$1\" = 'merged' ] || [ \"$1\" = 'm' ]; then \
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

	# ─── Worktree ───
	wtl = worktree list
	wtman = "!echo '\n\
	━━━ git worktree 가이드 ━━━\n\
	\n\
	■ 핵심 원칙\n\
	하나의 브랜치 = 하나의 디렉토리. 같은 브랜치를 두 곳에서 체크아웃할 수 없다.\n\
	worktree를 지워도 브랜치와 커밋은 .git에 남아있다. (uncommitted만 사라짐)\n\
	\n\
	■ 새 브랜치로 worktree 생성\n\
	git worktree add -b feature/my-work ../worktrees/my-work main\n\
                    	  ~~~~~~~~~~~~~~~ ~~~~~~~~~~~~~~~~~~~~ ~~~~\n\
	                    새 브랜치명     디렉토리 경로         base\n\
	\n\
	■ remote 브랜치를 로컬로 따서 worktree 생성\n\
	git fetch origin\n\
	git worktree add ../worktrees/my-work feature/my-work\n\
	(로컬에 feature/my-work가 없으면 origin/feature/my-work를 자동 추적)\n\
	\n\
	■ 삭제\n\
	git worktree remove ../worktrees/my-work\n\
	\n\
	■ 삭제가 안 될 때 (uncommitted changes)\n\
	방법 1: 해당 디렉토리에서 커밋 또는 stash 후 다시 remove\n\
	방법 2: git worktree remove ../worktrees/my-work --force  (변경사항 버림)\n\
	\n\
	■ worktree는 지웠는데 잔여 lock이 남을 때\n\
	git worktree prune\n\
	\n\
	■ 작업 완료 후 메인으로 가져오기\n\
	방법 1: 메인 디렉토리에서 git merge feature/my-work (worktree 살아있어도 됨)\n\
	방법 2: worktree remove → 메인에서 git checkout feature/my-work\n\
	'"

	# ─── Predict conflict ───
	predict-merge = "!f() { t=$(git branch -a --format=\"%(refname:short)\" | fzf --prompt=\"merge target> \" --preview=\"git log --oneline -10 {}\" --height=40%); [ -z \"$t\" ] && exit 0; echo \"[merge] $t -> HEAD\"; r=$(git merge-tree --write-tree HEAD \"$t\" 2>&1); if [ $? -eq 0 ]; then echo \"No conflicts\"; else echo \"Conflicts:\"; echo \"$r\" | tail -n+2 | awk -F\"\\t\" \"{print \\$2}\" | sort -u; fi; }; f"
	predict-rebase = "!f() { t=$(git branch -a --format=\"%(refname:short)\" | fzf --prompt=\"rebase onto> \" --preview=\"git log --oneline -10 {}\" --height=40%); [ -z \"$t\" ] && exit 0; echo \"[rebase] HEAD onto $t\"; r=$(git merge-tree --write-tree \"$t\" HEAD 2>&1); if [ $? -eq 0 ]; then echo \"No conflicts\"; else echo \"Conflicts:\"; echo \"$r\" | tail -n+2 | awk -F\"\\t\" \"{print \\$2}\" | sort -u; fi; }; f"
[merge]
	conflictStyle = zdiff3
[rerere]
	enabled = true

```
