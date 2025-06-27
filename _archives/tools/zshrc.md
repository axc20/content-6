# zshrc

```shell
# export PS1="%1d $ "
# PROMPT="%1d $ "
PROMPT="%1d $ "

# visual studio code
alias code="/Applications/Visual\ Studio\ Code.app/Contents/Resources/app/bin/code"

# etrade
alias reg-docs="npm config set registry https://registry.npmjs.org/"

function jira() {
  jiraBase="https://jira.corp.etradegrp.com/secure/RapidBoard.jspa?rapidView=625&projectKey=DESLANG"

  for i in $@; do
    if [ $i = "sprint" ]; then
      open -a safari $jiraBase
    elif [ $i = "me" ]; then
      open -a safari $jiraBase"&quickFilter=22810"
    else
      open -a safari "https://jira.corp.etradegrp.com/browse/DESLANG-"$i
    fi
  done
}

function del() {
  local main_branch

  if git show-ref --quiet refs/heads/master; then
    main_branch=master
  elif git show-ref --quiet refs/heads/main; then
    main_branch=main
  fi

  if [ "$1" = "" ]; then
    local current_branch=$(git branch --show-current)

    if [[ ($current_branch = "master" || $current_branch = "main") ]]; then
      echo cannot delete master or main
    else
      echo "$0: deleting $current_branch"
      git checkout $main_branch
      git pull
      git branch -D @{-1}
    fi
  elif [ "$1" = "all" ]; then
    echo "$0: deleting all branches except master, main, release"
    git checkout $main_branch
    git branch | grep -v "master\|main\|release" | xargs git branch -D
    git pull
    git branch
  elif [ "$1" = "nuke" ]; then
    echo "$0: deleting all branches except master, main"
    git checkout $main_branch
    git branch | grep -v "master\|main" | xargs git branch -D
    git pull
    git branch
  fi
}
```
