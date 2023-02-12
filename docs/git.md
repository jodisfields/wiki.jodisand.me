# Git Cheatsheet

### Git autocomplete

```sh
# curl https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash -o ~/.git-completion.bash
# .bashrc
if [ -f ~/.git-completion.bash ]; then
  export GIT_COMPLETION_CHECKOUT_NO_GUESS=0
  export GIT_COMPLETION_SHOW_ALL_COMMANDS=1
  export GIT_COMPLETION_SHOW_ALL=1
  source ~/.git-completion.bash
fi
```

### Debug flag, verbose output of commands, output debug

```sh
export GIT_TRACE=1
export GIT_TRACE=1
export GIT_CURL_VERBOSE=1
```

### Clean working tree remove untracked files

```sh
git clean --dry-run
git clean -f -d
```

```sh
# remove all remote non-used branches
git remote prune origin
```

### Restore

```
git reset --hard
```

### Restore local branch like remote one

```sh
git reset --hard origin/master
```

### Restore local branch with saving all the work

```sh
# save work to staging
git reset --soft origin/master
# save work to working dir
git reset --mixed HEAD~2
```

### Restore removed file, restore deleted file, find removed file, show removed file

```sh
# find full path to the file
file_name="integration_test.sh.j2"
git log --diff-filter=D --name-only | grep $file_name

# find last log messages
full_path="ansible/roles/data-ingestion/templates/integration_test.sh.j2"
git log -2 --name-only -- $full_path

second_log_commit="99994ccef3dbb86c713a44815ab5ffa"

# restore file from specific commit
git checkout $second_log_commit -- $full_path
# show removed file
git show $second_log_commit:$full_path
```

### Remove last commit and put HEAD to previous one

```sh
git reset --hard HEAD~1
```

### Checkout with tracking

```sh
git checkout -t origin/develop
```

### New branch from stash

```sh
git stash branch $BRANCH_NAME stash@{3}
```

### Show removed remotely

```sh
git remote prone origin
```

### Delete local branch, remove branch, remove local branch

```sh
git branch -d release-6.9.0
git branch --delete release-6.9.0

# delete with force - for non-merged branches
git branch -D origin/release/2018.05.00.12-test
# the same as
git branch -d -f release-6.9.0
git branch --delete --force origin/release/2018.05.00.12-test
```

### Delete remote branch, remove remote, remove remote branch

```sh
git push origin --delete release/2018.05.00.12-test
```

### Remove branches, delete branches that exist locally only ( not remotely ), cleanup local repo

```sh
git gc --prune=now
git fetch --prune
```

### Delete local branches that was(were) merged to master ( and not have 'in-progress' marker )

```sh
git branch --merged | egrep -v "(^\*|master|in-progress)" | xargs git branch -d
```

### Remove commit, remove wrong commit

```sh
commit1=10141d299ac14cdadaca4dd586195309020
commit2=b6f2f57a82810948eeb4b7e7676e031a634 # should be removed and not important
commit3=be82bf6ad93c8154b68fe2199bc3e52dd69

current_branch=my_branch
current_branch_ghost=my_branch_2

git checkout $commit1
git checkout -b $current_branch_ghost
git cherry-pick $commit3
git push --force origin HEAD:$current_branch
git reset --hard origin/$current_branch
git branch -d $current_branch_ghost
```

### Squash commit replace batch of commits

[interactive rebase](https://garrytrinder.github.io/2020/03/squashing-commits-using-interactive-rebase)

```sh
git checkout my_branch
# take a look into your local changes, for instance we are going to squeeze 4 commits
git reset --soft HEAD~4
# in case of having external changes and compress commits: git rebase --interactive HEAD~4

git commit # your files should be staged before
git push --force-with-lease origin my_branch
```

### Check hash-code of the branch, show commit hash code

```sh
git rev-parse "remotes/origin/release-6.0.0"
```

### Print current hashcode commit hash last commit hash

```sh
git rev-parse HEAD
git log -n 1 --pretty=format:'%h' > /tmp/gitHash.txt
```

### Print branch name by hashcode to branch name show named branches branchname find branch by hash

```sh
git ls-remote | grep <hashcode>
# answer will be like:          <hashcode>        <branch name>
# ada7648394793cfd781038f88993a5d533d4cdfdf        refs/heads/release-dataapi-13.0.2
```

or

```sh
git branch --all --contains ada764839
```

### Print branch hash code by name branch hash branch head hash

```sh
git rev-parse remotes/origin/release-data-api-13.3
```

### Check all branches for certain commit ( is commit in branch, is branch contains commit ), commit include in

```sh
git branch --all --contains 0ff27c79738a6ed718baae3e18c74ba87f16a314
git branch --all --merged 0ff27c79738a6ed718baae3e18c74ba87f16a314
# if branch in another branch
git branch --all --contains | grep {name-of-the-branch}
```

### Is commit included in another, commit before, commit after, if commit in branch

```sh
git merge-base --is-ancestor <commit_or_branch> <is_commit_in_branch>; if [[ 1 -eq "$?" ]]; then echo "NOT included"; else echo "included"; fi
```

### Check log by hash, message by hash

```sh
git log -1 0ff27c79738a6ed718baae3e18c74ba87f16a314
```

### Check last commits for specific branch, last commits in branch

```sh
git log -5 develop
```

### Check last commits for subfolder, check last commits for author, last commit in folder

```sh
git log -10 --author "Frank Newman" -- my-sub-folder-in-repo
```

### Log pretty print log oneline

```sh
git relog -5
```

### Check files only for last commits

```sh
git log -5 develop --name-only
```

### Check last commits by author, commits from all branches

```
git log -10 --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset%n' --all --author "Cherkashyn"
```

### List of authors, list of users, list of all users

```sh
git shortlog -sne --all
```

### List of files by author, list changed files

```bash
git whatchanged --author="Cherkashyn" --name-only
```

### Often changes by author, log files log with files

```bash
git log --author="Cherkashyn" --name-status --diff-filter=M | grep "^M" | sort | uniq -c | sort -rh
```

### Commit show files, files by commit

```sh
git diff-tree --no-commit-id --name-only -r ec3772
```

### Commit diff, show changes by commit, commit changes

```sh
git diff ec3772~ ec3772
```

### Pretty log with tree

```sh
git log --all --graph --decorate --oneline --simplify-by-decoration
```

### Git log message search commit message search commit search message

```sh
git log | grep -i jwt

git log --all --grep='jwt'
git log --name-only  --grep='XVIZ instance'
git log -g --grep='jwt'
```

### Show no merged branches

```
git branch --no-merged
```

### Show branches with commits

```sh
git show-branch
git show-branch -r
```

### Checkout branch locally and track it

```
git checkout -t remotes/origin/release
```

### Copy file from another branch

```
git checkout experiment -- deployment/connection_pool.py
git checkout origin/develop datastorage/mysql/scripts/_write_ddl.sh
# print to stdout
git show origin/develop:datastorage/mysql/scripts/_write_ddl.sh > _write_ddl.sh
```

### Git add

```sh
git add --patch
git add --interactive
```

### Git mark file unchanged skip file

```sh
git update-index --assume-unchanged path/to/file
```

### Set username, global settings

```sh
git config --global user.name "vitalii cherkashyn"
git config --global user.email vitalii.cherkashyn@wirecard.de
git config --global --list
```

or

```properties
# git config --global --edit
[user]
   name=Vitalii Cherkashyn
   email=vitalii.cherkashyn@bmw.de
```

### Default editor, set editor

```sh
git config --global core.editor "vim"
```

### Avoid to enter login/password

```
git config --global credential.helper store
```

### Revert all previous changes with "credential.helper"

```sh
git config --system --unset credential.helper
git config --global --unset credential.helper
```

### Git config mergetool

```sh
git config --global merge.tool meld
git config --global mergetool.meld.path /usr/bin/meld
```

### Show all branches merged into specified

```sh
git branch --all --merged "release" --verbose
git branch --all --no-merged "release" --verbose
git branch -vv
```

### Difference between two commits ( diff between branches )

```sh
git diff --name-status develop release-6.0.0
git cherry develop release-6.0.0
```

### Difference between branches for file ( diff between branches, compare branches )

```sh
git diff develop..master -- myfile.cs
```

github difference between two branches
https://github.com/cherkavi/management/compare/release-4.0.6...release-4.0.7

### Difference between branch and current file ( compare file with file in branch )

```sh
git diff master -- myfile.cs
```

### Difference between commited and staged

```
git diff --staged
```

### Difference between two branches, list of commits list commits, messages list of messages between two commits

```sh
git rev-list master..search-client-solr
# by author
git rev-list --author="Vitalii Cherkashyn" item-598233..item-530201
# list of files that were changed
git show --name-only --oneline `git rev-list --author="Vitalii Cherkashyn" item-598233..item-530201`
#  list of commits between two branches
git show --name-only --oneline `git rev-list d3ef784e62fdac97528a9f458b2e583ceee0ba3d..eec5683ed0fa5c16e930cd7579e32fc0af268191`
```

#### List of commits between two tags

```sh
# git tag --list
start_tag='1.0.13'
end_tag='1.1.2'
start_commit=$(git show-ref --hash $start_tag )
end_commit=$(git show-ref --hash $end_tag )
git show --name-only --oneline `git rev-list $start_commit..$end_commit`
```

#### All commits from tag till now

```sh
start_tag='1.1.2'
start_commit=$(git show-ref --hash $start_tag )
end_commit=$(git log -n 1 --pretty=format:'%H')
git show --name-only --oneline `git rev-list $start_commit..$end_commit`
```

### Difference for log changes, diff log, log diff

```
git log -1 --patch
git log -1 --patch -- path/to/controller_email.py
```

### Copying from another branch, copy file branch

```
branch_source="master"
branch_dest="feature-2121"
file_name="src/libs/service/message_encoding.py"

# check
git diff $branch_dest..$branch_source $file_name
# apply
git checkout $branch_source -- $file_name
# check
git diff $branch_source $file_name

```

### Tags

#### Create tag

```
git tag -a $newVersion -m 'deployment_jenkins_job'
```

#### Push tags only

```
git push --tags $remoteUrl
```

#### Show tags

```
# show current tags show tags for current commit
git show
git describe --tags
git describe


# fetch tags
git fetch --all --tags -prune

# list of all tags list tag list
git tag
git tag --list
git show-ref --tags

# tag checkout tag
git tags/1.0.13
```

#### Show tag hash

```
git show-ref -s 1.1.2
```

#### Remove tag delete tag delete

```sh
# remove remote
git push --delete origin 1.1.0
git push origin :refs/tags/1.1.0
git fetch --all --tags -prune

# or remove remote
git push --delete origin 1.2.1

# remove local
git tag -d 1.1.0
git push origin :refs/tags/1.1.0
```

### Conflict files, show conflicts

```sh
git diff --name-only --diff-filter=U
```

### Conflict file apply remote changes

```sh
git checkout --theirs path/to/file
```

### Git fetch

```sh
git fetch --all --prune
```

### Find by comment

```
git log --all --grep "BCM-642"
```

### Find by diff source, find through all text changes in repo

```
git grep '^test$'
```

### Current comment

```
git rev-parse HEAD
```

### Find file into log

```
git log --all -- "**db-update.py"
git log --all -- db-scripts/src/main/python/db-diff/db-update.py
```

### History of file, file changes file authors file log file history file versions

```sh
git log path/to/file
git log -p -- path/to/file
```

### Files in commit

```
git diff-tree --no-commit-id --name-only -r 6dee1f44f56cdaa673bbfc3a76213dec48ecc983
```

### Difference between current state and remote branch

```
git fetch --all
git diff HEAD..origin/develop
```

### Show changes into file only

```
git show 143243a3754c51b16c24a2edcac4bcb32cf0a37d -- db-scripts/src/main/python/db-diff/db-update.py
```

### Show changes by commit, commit changes

```
git diff {hash}~ {hash}
```

### Git cherry pick without commit, just copy changes from another branch

```
git cherry-pick -n {commit-hash}
```

### Git cherry pick with original commit message cherry pick tracking cherry pick original hash

```
git cherry-pick -x <commit hash>
```

### Git cherry pick, git cherry-pick conflict

```sh
# in case of merge conflict during cherry-pick
git cherry-pick --continue
git cherry-pick --abort
git cherry-pick --skip
# !!! don't use "git commit"
```

### Git new branch from detached head

```sh
git checkout <hash code>
git cherry-pick <hash code2>
git switch -c <new branch name>
```

### Git revert commit

```
git revert <commit>
```

### Git revert message for commit

```
git commit --amend -m "<new message>"
```

### Git show author of the commit, log commit, show commits only

```
git log --pretty=format:"%h - %an, %ar : %s" <commit SHA> -1
```

### Show author, blame, annotation, line editor, show editor

```
git blame path/to/file
git blame path/to/file | grep search_line
```

### Git into different repository, different folder, another folder, not current directory

```
git --git-dir=C:\project\horus\.git  --work-tree=C:\project\horus  branch --all
```

```sh
find . -name ".git" -maxdepth 2 | while read each_file
do
   echo $each_file
   git --git-dir=$each_file --work-tree=`dirname $each_file` status
done
```

### Show remote url

```
git remote -v
git ls-remote
git ls-remote --heads
```

### [Git chain of repositories](https://github.com/cherkavi/solutions/tree/master/git-repo-chain)

### Connect to existing repo

```sh
PATH_TO_FOLDER=/home/projects/bash-example

# remote set
git remote add local-hdd file://${PATH_TO_FOLDER}/.git
# commit all files
git add *; git commit --message 'add all files to git'

# set tracking branch
git branch --set-upstream-to=local-hdd/master master

# avoid to have "refusing to merge unrelated histories"
git fetch --all
git merge master --allow-unrelated-histories
# merge all conflicts
# in original folder move to another branch for avoiding: branch is currently checked out
git push local-hdd HEAD:master

# go to origin folder
cd $PATH_TO_FOLDER
git reset --soft origin/master
git diff
```

### Using authentication token personal access token, git remote set, git set remote

example of using github.com

```sh
# Settings -> Developer settings -> Personal access tokens
# https://github.com/settings/apps
git remote set-url origin https://$GIT_TOKEN@github.com/cherkavi/python-utilitites.git

# in case of Error: no such remote
git remote add origin https://$GIT_TOKEN@github.com/cherkavi/python-utilitites.git

# in case of asking username & password - check URL, https prefix, name of the repo....
```

remove old password-access approach

```sh
git remote set-url --delete origin https://github.com/cherkavi/python-utilitites.git
```

### Change remote url

```
git remote set-url origin git@cc-github.my-network.net:adp/data-management.git
```

### Git clone via https

```sh
# username - token
# password - empty string
git clone               https://$GIT_TOKEN@cc-github.group.net/swh/management.git
git clone        https://oauth2:$GIT_TOKEN@cc-github.group.net/swh/management.git
git clone https://$GIT_TOKEN:x-oauth-basic@cc-github.group.net/swh/management.git
```

### Git push via ssh git ssh

```sh
git commmit -am 'hello my commit message'
GIT_SSH_COMMAND="ssh -i $key"
git push
```

### Issue with removing files, issue with restoring files, can't restore file, can't remove file

```
git rm --cached -r .
git reset --hard origin/master
```

### Clone operation under the hood

if during the access ( clone, pull ) issue appear:

```
fatal: unable to access 'http://localhost:3000/vitalii/sensor-yaml.git/': The requested URL returned error: 407
```

or

```
fatal: unable to access 'http://localhost:3000/vitalii/sensor-yaml.git/': The requested URL returned error: 503
```

use next command to 'simulate' cloning

```
git clone http://localhost:3000/vitalii/sensor-yaml.git
< equals >
wget http://localhost:3000/vitalii/sensor-yaml.git/info/refs?service=git-upload-pack
```

### Clone only files without history, download code

```
git clone --depth 1 https://github.com/kubernetes/minikube
```

### Download single file from repo

```
git archive --remote=ssh://https://github.com/cherkavi/cheat-sheet HEAD jenkins.md
```

### Update remote branches, when you see not existing remote branches

```
git remote update origin --prune
```

### Worktree

> worktree it is a hard copy of existing repository but in another folder
> all worktrees are connected

```sh
# list of all existing wortrees
git worktree list

# add new worktree list
git worktree add $PATH_TO_WORKTREE $EXISTING_BRANCH

# add new worktree with checkout to new branch
git worktree add -b $BRANCH_NEW $PATH_TO_WORKTREE

# remove existing worktree, remove link from repo
git worktree remove $PATH_TO_WORKTREE
git worktree prune
```

### Git lfs

[package update](https://packagecloud.io/github/git-lfs/install)

```sh
echo 'deb http://http.debian.net/debian wheezy-backports main' > /etc/apt/sources.list.d/wheezy-backports-main.list
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
```

tool installation

```sh
sudo apt-get install git-lfs
git lfs install
git lfs pull
```

if you are using SSH access to git, you should specify http credentials ( lfs is using http access ), to avoid possible errors: "Service Unavailable...", "Smudge error...", "Error downloading object"

```bash
git config --global credential.helper store
```

file .gitconfig will have next section

```
[credential]
        helper = store
```

file ~/.git-credentials ( default from previous command ) should contains your http(s) credentials

```file:~/.git-credentials
https://username:userpass@aa-github.mygroup.net
https://username:userpass@aa-artifactory.mygroup.ne
```

#### Git lfs proxy

be aware about upper case for environment variables

```
NO_PROXY=localhost,127.0.0.1,.localdomain,.advantage.org
HTTP_PROXY=muc.proxy
HTTPS_PROXY=muc.proxy
```

#### Issue with git lfs

```
Encountered 1 file(s) that should have been pointers, but weren't:
```

```
git lfs migrate import --no-rewrite path-to-file
```

#### Git lfs add file

```sh
git lfs track "*.psd"
```

check tracking changes in file:

```sh
git add .gitattributes
```

### Create local repo in filesystem

```sh
# create bare repo file:///home/projects/bmw/temp/repo
# for avoiding: error: failed to push some refs to
mkdir /home/projects/bmw/temp/repo
cd /home/projects/bmw/temp/repo
git init --bare
# or git config --bool core.bare true

# clone to copy #1
mkdir /home/projects/bmw/temp/repo2
cd /home/projects/bmw/temp/repo2
git clone file:///home/projects/bmw/temp/repo

# clone to copy #1
mkdir /home/projects/bmw/temp/repo3
cd /home/projects/bmw/temp/repo3
git clone file:///home/projects/bmw/temp/repo
```

### Configuration for proxy server, proxy configuration

#### Set proxy, using proxy

```sh
git config --global http.proxy 139.7.95.74:8080
# proxy settings
git config --global http.proxy http://proxyuser:proxypwd@proxy.server.com:8080
git config --global https.proxy 139.7.95.74:8080
```

#### Check proxy, get proxy

```sh
git config --global --get http.proxy
```

#### Remove proxy configuration, unset proxy

```sh
git config --global --unset http.proxy
```

### Using additional command before 'fetch' 'push', custom fetch/push

```
# remote: 'receive.denyCurrentBranch' configuration variable to 'refuse'.
git config core.sshCommand 'ssh -i private_key_file'
```

### Set configuration

```sh
git config --local receive.denyCurrentBranch updateInstead
```

### Remove auto replacing CRLF for LF on Windows OS

.gitattributes

```
*.sh -crlf
```

### Http certificate ssl verification

```sh
git config --system http.sslcainfo C:\soft\git\usr\ssl\certs\ca-bundle.crt
# or
git config --system http.sslverify false
```

### Download latest release from github, release download

```
curl -s https://api.github.com/repos/bugy/script-server/releases/latest | grep browser_download_url | cut -d '"' -f 4
```

### Download last version of file from github, url to source, source download

```
wget https://raw.githubusercontent.com/cherkavi/cheat-sheet/master/git.md
```

### Linux command line changes

```
#git settings parse_git_branch() {
parse_git_branch() {
     git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
}
export PS1="\[\033[32m\]\W\[\033[33m\]\$(parse_git_branch)\[\033[00m\] $ "​
```

### Ignore tracked file, ignore changes

```
git update-index --assume-unchanged .idea/vcs.xml
```

## Hooks

### Check commit message

```
mv .git/hooks/commit-msg.sample .git/hooks/commit-msg
```

```
result=`cat $1 | grep "^check-commit"`

if [ "$result" != "" ]; then
	exit 0
else
	echo "message should start from 'check-commit'"
	exit 1
fi
```

if you want to commit hooks, then create separate folder and put all files there

```
git --git-dir $DIR_PROJECT/integration-prototype/.git config core.hooksPath $DIR_PROJECT/integration-prototype/.git_hooks
```

### Git template message template

```
git --git-dir $DIR_PROJECT/integration-prototype/.git config commit.template $DIR_PROJECT/integration-prototype/.commit.template
```

## [Git lint](https://jorisroovers.com/gitlint/)

```sh
pip install gitlint
gitlint install-hook
```

.gitlint

```properties
# See http://jorisroovers.github.io/gitlint/rules/ for a full description.
[general]
ignore=T3,T5,B1,B5,B7
[title-match-regex]
regex=^[A-Z].{0,71}[^?!.,:; ]
```

## Github REST API

export PAT=07f1798524d6f79...
export GIT_USER=tech_user
export GIT_USER2=another_user
export GIT_REPO=system_description
export GIT_URL=https://github.sbbgroup.zur

[git rest api](https://docs.github.com/en/enterprise-server@3.1/rest)
[git endpoints](https://docs.github.com/en/enterprise-server@3.1/rest/overview/endpoints-available-for-github-apps)

```sh
# read user's data
curl -H "Authorization: token ${PAT}" ${GIT_URL}/api/v3/users/${GIT_USER}
curl -u ${GIT_USER}:${PAT} ${GIT_URL}/api/v3/users/${GIT_USER2}

# list of repositories
curl -u ${GIT_USER}:${PAT} ${GIT_URL}/api/v3/users/${GIT_USER2}/repos | grep html_url

# read repository
curl -u ${GIT_USER}:${PAT} ${GIT_URL}/api/v3/repos/${GIT_USER2}/${GIT_REPO}
curl -H "Authorization: token ${PAT}" ${GIT_URL}/api/v3/repos/${GIT_USER2}/${GIT_REPO}


# read path
export FILE_PATH=doc/README
curl -u ${GIT_USER}:${PAT} ${GIT_URL}/api/v3/repos/${GIT_USER2}/${GIT_REPO}/contents/${FILE_PATH}
curl -u ${GIT_USER}:${PAT} ${GIT_URL}/api/v3/repos/${GIT_USER2}/${GIT_REPO}/contents/${FILE_PATH} | jq .download_url

# https://docs.github.com/en/enterprise-server@3.1/rest/reference/repos#contents
# read content
DOWNLOAD_URL=`curl -u ${GIT_USER}:${PAT} ${GIT_URL}/api/v3/repos/${GIT_USER2}/${GIT_REPO}/contents/${FILE_PATH} | jq .download_url | tr '"' ' '`
echo $DOWNLOAD_URL
curl -u ${GIT_USER}:${PAT} -X GET $DOWNLOAD_URL

# read content
curl -u ${GIT_USER}:${PAT} ${GIT_URL}/api/v3/repos/${GIT_USER2}/${GIT_REPO}/contents/${FILE_PATH} | jq -r ".content" | base64 --decode
```
