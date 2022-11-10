# git-training

### Two factor Authentication
- We should safeguard our git account by using two factor authentication.
- Check out [this link](https://docs.github.com/en/authentication/securing-your-account-with-two-factor-authentication-2fa/configuring-two-factor-authentication) for more details

### Using Git via SSH and HTTPS
- When 2FA is enabled, git will ask for password for each operation. We can avoid entering password by two ways
1. API Token
2. SSH Key

#### Create an API Token
- Create a new classic token by visiting [this link](https://github.com/settings/tokens)
- Enter a note and select all the scopes
- Click on generate token and ensure you copy the token and save it somewhere

#### Setup an ssh key
- Run the below commands (passphrase is optional, but it's strongly recommended)
```
ssh-keygen -t ed25519 -C "your_email@example.com"
cat ~/.ssh/id_ed25519.pub
```
- Add the public key to your profile by visiting [this link](https://github.com/settings/keys)

#### `gh` command line
- Install the `gh` command line by visiting [this link](https://github.com/cli/cli)

### Git commands
- Help section
```
git help
```
- Configuring git with your data
```
git config --global user.name "Your Name"
git config --global user.email your_email@example.com
git config --global init.defaultBranch main
git config --global core.editor vim
cat ~/.gitconfig
```

- Initializing a git repo (replace the value `YOUR_USER_NAME` with your actual user name)
```
mkdir random-git-repo
git init
touch README.md
git add README.md
git commit -m "initializing repo"
gh repo create --source=. --public
git push origin main

# Enter email and api token and then check the repo for your commit on Github
```
- Fork the repo https://github.com/keshavprasadms/git-training and clone the forked repo (replace the value `YOUR_USER_NAME` with your actual user name)
```
git clone https://github.com/YOUR_USER_NAME/git-training

# See clone to a directory
# See clone and checkout a branch
```
- Add a new file, commit and push
```
echo 'print("Hello there!")' > hello.py
python hello.py
git add hello.py
git commit -m "feat: adding hello.py"
git push origin main
```
- Every push is asking for a password? If yes, export an environment variable (replace the value `YOUR_API_TOKEN` with your actual api token)
```
export GITHUB_TOKEN=YOUR_API_TOKEN
```
- View current branch and checkout a new branch, check staged area using status command, check diff, commit and push
```
git branch
git checkout -b feature-1
echo 'print("This is feature-1!")' >> hello.py
python hello.py
git status
git diff
git add .
git commit -m "feat: adding feature-1"
git push origin feature-1
```
- Switch to main branch, merge changes of feature branch, check commit logs, set upstream branch, push and delete the branch from local and remote
```
git switch main
git merge feature-1
python hello.py
git log
git push --set-upstream origin main
git branch -vv
git branch -d feature-1
git push origin :feature-1
```
- Lets repeat adding another feature and see some more options like amending a commit message, restoring files back to original changes, restoring files from staged area and reverting commits
```
git branch
git checkout -b feature-2
echo 'print("This is feature-2!")' >> hello.py
echo "Hey there" > README.md
python hello.py
git status
git restore README.md
git status
echo "Hey there" > README.md
git diff
git add .
git status
git restore --staged README.md
git restore README.md
git commit -m "feat: adding feature-2"
git push origin feature-2
```
- Lets add a few commits and reset back to an older commit
```
for i in {1..5}; do echo $i >> README.md && git commit -am "feat adding commit $i"; done
cat README.md
git log
git reset HEAD~5
git log
cat README.md
git checkout README.md
for i in {1..5}; do echo $i >> README.md && git commit -am "feat adding commit $i"; done
cat README.md
git reset HEAD~5 --hard
cat README.md

# you can also do git reset COMMIT_ID
```
- Lets merge feature 2 also to main
```
git branch
git checkout main
git merge feature-2
python hello.py
git log
git push origin main
git branch -d feature-2
git push origin :feature-2
```
- Now feature 2 has introduced a bug in production. We need to revert it from main branch and fix the bug later.
```
python hello.py
git revert LATEST_COMMIT_ID
python hello.py
git push origin main
git checkout HEAD~1 -b feature-2-bugfix
python hello.py
sed -i 's/This is feature-2!/This is feature-2 fixed!/g' hello.py
git status
git diff
git commit -am "fix: feature-2 bug fix"
git push origin feature-2-bugfix

# You can replace HEAD~NUMBER with actual commit hash
# For macos use
# sed -i'.bak' -e 's/This is feature-2!/This is feature-2 fixed!/g' hello.py && rm hello.py.bak
```
- Lets check the difference with main branch and try to do a merge
```
git log
git log main
git merge main

# Lets resolve the conflict manually by editing the file and removing unnessary lines

git add hello.py
git merge --continue
git log
```
- At this point, we can see we are adding unnecessary merge commits. We should avoid this and instead do a rebase. 
- Lets repeat same merge operation again with an alternate command. 
- We will abort the merge and then look at rebase.
```
git reset HEAD~1 --hard
git log --oneline
git log main --oneline
git pull origin main
git merge --abort
git pull origin main --rebase

# Resolve the conflicts manually
git add .
git rebase --continue
git push origin feature-2-bugfix
```
- Now raise a PR from feature-2-bugfix to main branch
- Reveiw and Merge

### Working with forks
- Adding a remote repository as upstream, checking out the upstream branch and syncing with fork
```
git remote add upstream git@github.com:keshavprasadms/random-git-repo.git
git remote -v
git fetch upstream
git branch -a
git checkout -b remotes/upstream/main -b upstream-main
git branch -D main
git checkout -b remotes/upstream/main -b main

# Both the below commands do the same operation
# Push from one branch to another
git push origin upstream-main:main --force

# Force push
git push origin main --force
```
- After force push we lost the latest commit that was introduced by the PR. So be careful in force push.
- Lets take the missing commit to our branch and and push again
Go to the closed PR and get the commit hash
```
git show COMMIT_HASH
git checkout main
git log --oneline
git pull origin main --rebase
git checkout COMMIT_HASH
git push origin main
git branch
git checkout -b missing-commit
git push origin missing-commit:main
``` 
- Lets see how we can pick some specific commits and also look at tagging
```
git checkout -b branch1
git checkout -b branch2
for i in {1..5}; do echo $i >> README.md && git commit -am "feat adding commit $i"; done
git log --oneline
git checkout branch1
git show FIRST_NEW_COMMIT_HASH
git cherry-pick FIRST_NEW_COMMIT_HASH

# If you pick random in between commits, you might get merge conflicts which you can resolve manually or abort using --abort option
# Lets also take rest of the commits

git cherry-pick FIRST_NEW_COMMIT_HASH..LAST_NEW_COMMIT_HASH

git checkout main
git log --oneline
git tag v1.0.0
git show tags/v1.0.0
git push origin tags/v1.0.0

# Check tags on UI
git checkout COMMIT_HASH_OF_FEATURE_1
git tag v0.0.1
git push origin v0.0.1

# Check tags on UI

git tag -d v0.0.1
git push origin :tags/v0.0.1
git fetch upstream --tags
```
- Remove files, stash file, pop files, grep files and blame
```
# Remove files
rm README.md
git status
git checkout .
git rm README.md
git status
git restore --staged .
git checkout .
rm README.md

# Stash and Pop files
echo "Hey" > README.md
git checkout branch1
git stash
git checkout branch1
git checkout main
git stash pop
git stash
echo "OK" > README.md
git stash
git stash list
git stash pop --index 2
git stash show -p stash@{0}

# Grep
git grep -i "hello"
git grep -iv "hello"

# Blame
git blame hello.py
```

### Aliases
- Set aliases as per your liking to blaze through the command line
- Here are a few commonly used git alises
```
alias g='git'
alias ga='git add'
alias gb='git branch'
alias gc='git checkout'
alias gcp="git cherry-pick"
alias gca='git commit --amend'
alias gcm='git commit -S -s -m'
alias gd='git diff'
alias gfa='git fetch --all'
alias gl='git log'
alias glo='git log --graph --pretty=oneline --abbrev-commit'
alias gpl='git pull --rebase'
alias gplo='git pull --rebase origin heads/$(git rev-parse --abbrev-ref HEAD | rev | cut -d / -f1 | rev)'
alias gplu='git pull --rebase upstream heads/$(git rev-parse --abbrev-ref HEAD | rev | cut -d / -f1 | rev)'
alias gpo='git push origin $(git rev-parse --abbrev-ref HEAD)'
alias gpu='git push upstream $(git rev-parse --abbrev-ref HEAD)'
alias grb='git rebase'
alias grc='git rebase --continue'
alias grs='git rebase --skip'
alias gs='git status'
alias gst='git stash'
alias gstp='git stash pop'
```

### Sign your commits
- Signing your commits is always a good practice as it proves your identity
- Follow below steps to setup signing of commits
```
gpg --version
# >= 2.1.17
gpg --full-generate-key

# < 2.1.17
gpg --default-new-key-algo rsa4096 --gen-key

# Enter your details
gpg --list-secret-keys --keyid-format=long

# Get the key id by looking at sec field and after /
gpg --armor --export 3AA5C34371567BD2
```
- Copy your GPG key, beginning with `-----BEGIN PGP PUBLIC KEY BLOCK-----` and ending with `-----END PGP PUBLIC KEY BLOCK-----`
- Add the key [here](https://github.com/settings/keys)
- If any issues, read this [doc](https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key)

### Raising PR using hub
- Install hub and check it out
- Here is a sample
```
hub pull-request -b USERNAME_OR_ORGANISATION/REPOSITORY_NAME:BRANCH -m "title: text" -m "description: text"
```
- Example
```
hub pull-request -b keshavprasadms/git-training:main -m "Git training notes" -m "feat: adding training notes"
```

### Using git on SSH
```
echo 'ssh-add -k ~/.ssh/id_ed25519' >> ~/.bashrc
source ~/.bashrc

# Remove the GITHUB_TOKEN environment variable and use ssh going forward
```
### Further Exploration
- [Explore the `hub` command](https://hub.github.com/)
- [Explore the `gh` command](https://cli.github.com/manual/)