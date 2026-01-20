# Hướng Dẫn GitLab CLI (glab)

Đúng rồi! GitLab CLI (`glab`) là công cụ dòng lệnh để quản lý GitLab từ terminal, giúp bạn làm việc nhanh hơn mà không cần mở web browser.

## 📦 Cài Đặt

### macOS
```bash
brew install glab
```

### Linux
```bash
# Debian/Ubuntu
sudo apt install glab

# Hoặc dùng snap
sudo snap install glab
```

### Kiểm tra version
```bash
glab version
```

## 🔐 Xác Thực & Thiết Lập

### Đăng nhập lần đầu
```bash
# Đăng nhập với GitLab.com
glab auth login

# Đăng nhập với GitLab tự host
glab auth login --hostname gitlab.example.com
```

### Kiểm tra trạng thái đăng nhập
```bash
glab auth status
```

### Tạo Personal Access Token thủ công
```bash
# Tạo token tại: GitLab > Preferences > Access Tokens
# Sau đó:
glab auth login --token YOUR_TOKEN
```

## 🔄 Merge Request (MR) - Lệnh Phổ Biến Nhất

### Tạo MR
```bash
# Tạo MR cơ bản
glab mr create

# Tạo MR với đầy đủ thông tin
glab mr create \
  --source-branch feat/ai-chat \
  --target-branch sandbox \
  --title "Add AI Chat" \
  --description "Implement AI chat feature" \
  --label "feature,enhancement" \
  --assignee @username \
  --milestone "v2.0"

# Tạo MR draft
glab mr create --draft

# Tạo và xóa source branch sau khi merge
glab mr create --remove-source-branch

# Tạo MR và mở web để review
glab mr create --web
```

### Xem danh sách MR
```bash
# List tất cả MR
glab mr list

# MR của mình
glab mr list --author=@me

# MR được assigned cho mình
glab mr list --assignee=@me

# MR theo trạng thái
glab mr list --state=opened
glab mr list --state=merged
glab mr list --state=closed
```

### Xem chi tiết MR
```bash
# Xem MR số 123
glab mr view 123

# Xem và mở trên web
glab mr view 123 --web
```

### Approve & Merge MR
```bash
# Approve MR
glab mr approve 123

# Merge MR
glab mr merge 123

# Merge và xóa source branch
glab mr merge 123 --remove-source-branch

# Merge khi pipeline pass
glab mr merge 123 --when-pipeline-succeeds
```

### Checkout MR để test
```bash
# Checkout MR số 123 về local
glab mr checkout 123
```

### Comment vào MR
```bash
glab mr note 123 -m "LGTM! 👍"
```

### Đóng/Reopen MR
```bash
glab mr close 123
glab mr reopen 123
```

## 📝 Issues

### Tạo issue
```bash
# Tạo issue cơ bản
glab issue create

# Tạo issue với đầy đủ info
glab issue create \
  --title "Fix login bug" \
  --description "Users cannot login with email" \
  --label "bug,priority::high" \
  --assignee @username
```

### Xem issues
```bash
# List tất cả issues
glab issue list

# Issues của mình
glab issue list --author=@me

# Issues assigned cho mình
glab issue list --assignee=@me

# Xem chi tiết issue
glab issue view 456
```

### Đóng issue
```bash
glab issue close 456
```

## 🏢 Repository Management

### Clone repo
```bash
# Clone repo
glab repo clone group/project-name

# Clone và đặt tên khác
glab repo clone group/project-name my-local-name
```

### Xem thông tin repo
```bash
glab repo view

# Xem repo khác
glab repo view group/another-project
```

### Fork repo
```bash
glab repo fork
```

## 🚀 CI/CD Pipeline

### Xem pipelines
```bash
# List pipelines
glab ci list

# Xem chi tiết pipeline
glab ci view

# Xem pipeline jobs
glab ci trace
```

### Trigger pipeline
```bash
# Run pipeline cho branch hiện tại
glab ci run

# Run pipeline cho branch khác
glab ci run --branch develop
```

### Xem logs của job
```bash
glab ci trace
```

## 🤖 Automation Cho Nhiều Repos

### Script 1: Tạo MR cho nhiều repos cùng lúc
```bash
#!/bin/bash
# File: bulk-create-mr.sh

REPOS=(
  "group/repo1"
  "group/repo2"
  "group/repo3"
)

SOURCE_BRANCH="feat/update-dependencies"
TARGET_BRANCH="main"
TITLE="Update dependencies to latest version"
DESCRIPTION="Automated dependency update"

for repo in "${REPOS[@]}"; do
  echo "Creating MR for $repo..."
  glab mr create \
    --repo "$repo" \
    --source-branch "$SOURCE_BRANCH" \
    --target-branch "$TARGET_BRANCH" \
    --title "$TITLE" \
    --description "$DESCRIPTION" \
    --label "dependencies,automated" \
    || echo "Failed to create MR for $repo"
done
```

### Script 2: Clone nhiều repos cùng lúc
```bash
#!/bin/bash
# File: bulk-clone.sh

REPOS=(
  "group/frontend"
  "group/backend"
  "group/mobile-app"
  "group/infrastructure"
)

BASE_DIR="$HOME/projects"

mkdir -p "$BASE_DIR"
cd "$BASE_DIR" || exit

for repo in "${REPOS[@]}"; do
  echo "Cloning $repo..."
  glab repo clone "$repo" || echo "Failed to clone $repo"
done
```

### Script 3: Approve tất cả MR đang chờ
```bash
#!/bin/bash
# File: bulk-approve-mrs.sh

# Lấy danh sách MR assigned cho mình
MR_LIST=$(glab mr list --assignee=@me --state=opened --per-page=100 | grep -oP '!\d+' | tr -d '!')

for mr_id in $MR_LIST; do
  echo "Approving MR #$mr_id..."
  glab mr approve "$mr_id" || echo "Failed to approve MR #$mr_id"
done
```

### Script 4: Check CI status cho nhiều repos
```bash
#!/bin/bash
# File: check-ci-status.sh

REPOS=(
  "group/repo1"
  "group/repo2"
  "group/repo3"
)

for repo in "${REPOS[@]}"; do
  echo "Checking CI status for $repo..."
  glab ci list --repo "$repo" --per-page=1
  echo "---"
done
```

### Script 5: Tạo release notes từ MRs
```bash
#!/bin/bash
# File: generate-release-notes.sh

MILESTONE="v2.0"
OUTPUT_FILE="release-notes.md"

echo "# Release Notes for $MILESTONE" > "$OUTPUT_FILE"
echo "" >> "$OUTPUT_FILE"

# Lấy tất cả MRs merged trong milestone
glab mr list --milestone="$MILESTONE" --state=merged --per-page=100 | \
  grep -oP '!\d+' | tr -d '!' | while read -r mr_id; do
    TITLE=$(glab mr view "$mr_id" --output-format json | jq -r '.title')
    echo "- $TITLE (!$mr_id)" >> "$OUTPUT_FILE"
done

echo "Release notes generated: $OUTPUT_FILE"
```

## 📋 Config File cho Automation

### Tạo file config: .glab-config.yml
```yaml
# ~/.config/glab-cli/config.yml
---
hosts:
  gitlab.com:
    token: YOUR_TOKEN_HERE
    api_protocol: https
    git_protocol: ssh

# Default settings
default:
  gitlab_host: gitlab.com
  editor: code  # hoặc vim, nano, etc.
  browser: chrome
  
# Aliases
aliases:
  co: mr checkout
  approve: mr approve
  merge: mr merge --remove-source-branch
```

### Sử dụng environment variables
```bash
# Thêm vào ~/.zshrc hoặc ~/.bashrc
export GITLAB_TOKEN="your-token-here"
export GITLAB_HOST="gitlab.com"
export GLAB_EDITOR="code"
```

## 🔧 Tips & Tricks

### 1. Xem diff của MR
```bash
glab mr diff 123
```

### 2. Interactive mode
```bash
# Tạo MR với interactive prompts
glab mr create -i
```

### 3. Output format JSON để script
```bash
glab mr list --output-format json | jq '.[] | {id, title, author}'
```

### 4. Set default repo
```bash
# Trong folder của repo
glab config set remote origin
```

### 5. Quick aliases
```bash
# Thêm vào ~/.zshrc
alias gmr='glab mr'
alias gmrc='glab mr create'
alias gmrl='glab mr list'
alias gmrv='glab mr view'
alias gci='glab ci'
```

## 🎯 Workflow Thực Tế

### Workflow 1: Feature Development
```bash
# 1. Tạo branch mới
git checkout -b feat/new-feature

# 2. Code và commit
git add .
git commit -m "Add new feature"

# 3. Push
git push -u origin feat/new-feature

# 4. Tạo MR
glab mr create --title "Add new feature" --label "feature"

# 5. Sau khi approved, merge
glab mr merge
```

### Workflow 2: Bug Fix
```bash
# 1. Checkout MR để reproduce bug
glab mr checkout 123

# 2. Fix bug
# ... code ...

# 3. Comment vào MR
glab mr note 123 -m "Bug reproduced, fixing now..."

# 4. Push fix
git push

# 5. Ready for review
glab mr ready 123  # Remove draft status
```

### Workflow 3: Code Review
```bash
# 1. Xem MRs cần review
glab mr list --assignee=@me --state=opened

# 2. Xem diff
glab mr diff 123

# 3. Checkout để test local
glab mr checkout 123

# 4. Test...

# 5. Approve hoặc request changes
glab mr approve 123
# hoặc
glab mr note 123 -m "Please fix the linting errors"
```

## 📚 Tài Liệu Thêm

```bash
# Xem help
glab help

# Xem help cho command cụ thể
glab mr help
glab issue help
glab ci help
```

## 🆚 So Sánh: glab vs git

- `git`: Quản lý version control (commit, push, pull, branch...)
- `glab`: Quản lý GitLab platform (MR, issues, CI/CD, projects...)

**Kết hợp cả hai:**
```bash
# Git workflow
git checkout -b feat/xyz
git add .
git commit -m "Add xyz"
git push -u origin feat/xyz

# Sau đó dùng glab
glab mr create
```

Chúc bạn làm việc hiệu quả với GitLab CLI! 🚀
