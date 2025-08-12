#自己的小練習&語法

# 從零開始（一次性設定）

1. 安裝與身分設定

```bash
git --version                 # 檢查是否已安裝
git config --global user.name "你的名字"
git config --global user.email "你的信箱"
git config --global init.defaultBranch main   # 新 repo 預設分支叫 main
git config --global core.editor "code --wait" # 用 VS Code 編輯 commit 訊息（可選）
```

2. 生成 SSH 金鑰並連接 GitHub（推薦）

```bash
# 生成金鑰（一路 Enter，或自訂檔名與密碼）
ssh-keygen -t ed25519 -C "你的信箱"
# 顯示公鑰，複製貼到 GitHub → Settings → SSH and GPG keys
cat ~/.ssh/id_ed25519.pub

# 測試連線
ssh -T git@github.com
```

> 若你想先用 HTTPS 也行，之後再改 SSH。

3. 建立新倉庫（兩種起手式）

* 本機已有專案要上傳：

```bash
cd /path/to/project
git init
echo ".venv/\n__pycache__/\n*.log\n.DS_Store" > .gitignore   # 例：建立忽略檔
git add .
git commit -m "init"
# 到 GitHub 建立空 repo（不要勾選初始化 README）
git remote add origin git@github.com:<你的帳號>/<repo>.git   # 或用 HTTPS
git push -u origin main
```

* 從 GitHub 下載既有專案：

```bash
git clone git@github.com:<owner>/<repo>.git   # 或 HTTPS
cd <repo>
```

# 日常工作流（90% 情況就用這些）

0. 同步遠端最新（開始前好習慣）

```bash
git switch main
git pull --ff-only origin main   # 只做 fast-forward，避免意外 merge
```

1. 開分支開發

```bash
git switch -c feat/login-page    # 新功能分支（命名建議：feat/xxx、fix/xxx）
# ... 修改檔案 ...
git status                       # 看有哪些變更
git add -A                       # 全部加入暫存（或 git add <檔名>）
git commit -m "feat: add login page layout"
```

2. 推到遠端、發 PR

```bash
git push -u origin feat/login-page
# 接著到 GitHub 網頁按 "Compare & pull request"
```

3. Code review 後合併

* 在 GitHub 上 Merge PR（建議保留「Squash and merge」讓歷史乾淨）。
* 合併後，清理本地分支：

```bash
git switch main
git pull --ff-only
git branch -d feat/login-page      # 刪本地
git push origin --delete feat/login-page   # 刪遠端（可選）
```

4. 修正同一分支上的最後一次提交（小改字、補漏檔）

```bash
git add <漏掉的檔>
git commit --amend
git push --force-with-lease        # 只有你自己在該分支時才安全這樣做
```

5. 解簡單衝突（當 rebase/merge 時出現衝突）

```bash
# Git 會標記 <<<<<<< ======= >>>>>>>，手動編輯解決後：
git add <解完的檔案>
git rebase --continue   # 若你是在 rebase 流程
# 或 git commit          # 若你是在 merge 流程
```

# 進階常用指令（用到再查，這裡給最常見的）

查詢與比較

```bash
git log --oneline --graph --decorate --all   # 俯瞰分支歷史
git diff                     # 工作區 vs 暫存區差異
git diff --staged            # 暫存區 vs 最近一次提交
git blame <檔名>             # 逐行看是誰在何時改的
```

變更與回滾

```bash
git restore <檔名>           # 丟掉工作區對檔案的更動
git restore --staged <檔名>  # 從暫存區移除（不動工作區）
git revert <commit>          # 產生反向提交，安全「回滾」某次變更（推得上遠端）
git reset --soft <commit>    # 回到某提交，但保留檔案在暫存
git reset --mixed <commit>   # 預設；保留檔案在工作區
git reset --hard <commit>    # 小心！工作區一起回覆（會丟本地更動）
```

分支管理

```bash
git branch                   # 列出分支
git switch <branch>          # 切換分支（新語法，取代部分 checkout）
git merge <branch>           # 把 <branch> 合進目前分支（產生 merge commit）
git rebase main              # 把目前分支「平移」到 main 之後（歷史更線性）
```

暫存工作與挑單

```bash
git stash push -m "WIP camera calib"   # 暫存未完成的修改
git stash list
git stash pop                          # 還原並移除該筆 stash

git cherry-pick <commit>               # 把某一個提交套到當前分支
```

遠端操作

```bash
git remote -v
git remote add origin <url>
git fetch origin            # 取回遠端更新但不合併
git pull --ff-only          # 只做快轉更新，避免意外 merge
git push -u origin <branch>
```

標籤（版本號）

```bash
git tag v1.0.0
git push origin v1.0.0
```

忽略檔案（專案與全域）

```bash
# 專案 .gitignore（放在 repo 根目錄）
.venv/
__pycache__/
*.log
.DS_Store

# 個人全域 ignore（不會進 repo）
git config --global core.excludesfile ~/.gitignore_global
echo ".DS_Store\nThumbs.db\nnode_modules/" >> ~/.gitignore_global
```

# 緊急救援（誤操作時看這裡）

1. 找回「消失的提交」

```bash
git reflog            # 列出 HEAD 走過的所有位置
git checkout <安全點> # 或用 git switch -d，先確認內容
# 確認沒問題，再決定要 reset 回去或 cherry-pick 回來
```

2. 誤用 `--hard` 或 `--force`

* 先 `git reflog` 找到之前的 commit 或分支尖端，再 `git reset --hard <那個commit>`。

3. 線上分支被改寫

* 若他人強推（force push）導致歷史變動：

  * 先 `git fetch`，再開新分支保護現狀：`git switch -c rescue/my-view`，然後比對遠端：`git log --oneline --graph --decorate --all`，用 `cherry-pick`/`merge` 整理。

如果你想「回溯到上一個版本」，有兩種常用方式，差別在於 **要不要保留你剛做的修改紀錄**。

---

## **方法 1：回到上一個版本，但保留修改在工作區（安全、不刪檔）**

```bash
git reset --soft HEAD~1
```

* **`--soft`**：只是退回到前一個 commit 狀態，但保留所有修改在 **已暫存（staged）** 狀態，可以馬上重新 commit。
* 適合：commit 訊息打錯、剛提交但想再補改內容。

---

## **方法 2：回到上一個版本，保留修改但不在暫存區**

```bash
git reset --mixed HEAD~1
```

* **`--mixed`**（預設）：退回到前一個版本，修改保留但要重新 `git add` 才能提交。
* 適合：想重新選擇要加哪些檔案。

---

## **方法 3：回到上一個版本並丟掉修改（危險）**

```bash
git reset --hard HEAD~1
```

* **`--hard`**：直接丟掉最近一次 commit 以及所有修改，回到前一版本的純淨狀態。
* 適合：確定這次修改完全不要了。
* **⚠ 注意**：檔案會直接消失，除非還在 GitHub 上可以抓回來。

---

## **方法 4：查看歷史版本並回到特定版本**

1. 先查歷史版本 ID：

```bash
git log --oneline
```

會看到：

```
a1b2c3d 修正首頁錯誤
d4e5f6g 新增 README
```

2. 回到某個版本：

```bash
git reset --hard a1b2c3d
```

---
好，給你一份「**安全回到上一版，並同步 GitHub**」流程表（照情境選一條）。
原則：**有對外的 repo 就優先用 `git revert`（不改歷史）**；只有你自己用時，才考慮改寫歷史（`reset + force push`）。

---

# 一、尚未 push 到 GitHub（只在本地）

最簡單，改寫歷史不會害到別人。

```bash
# 回到上一版，保留變更在工作區（可再調整）
git reset --mixed HEAD~1

# 或直接丟掉剛剛那版（危險，不留變更）
git reset --hard HEAD~1
```

如需同步 GitHub（第一次 push 或已不存在遠端同一 commit 就不用管）。

---

# 二、已 push，且**多人**會拉同一分支（安全做法）

用 **revert 產生一個「反向提交」**，不改歷史。

```bash
# 1) 建備份分支（保險）
git branch backup-before-revert-$(date +%Y%m%d-%H%M)

# 2) 反轉上一個 commit
git revert HEAD

# 3) 推送
git push
```

想回到更早的某一顆：

```bash
git revert <commit_id>
# 多顆連續可用範圍：git revert <old_id>^..<new_id>
```

**如何取消剛做的 revert？**
找到那筆 revert 的 commit id，再做一次 `git revert <那筆id>`（revert the revert）。

---

# 三、已 push，但**只有你一個人用**（需要把 GitHub 也回到前一版）

可以改寫歷史；用 `--force-with-lease` 比 `--force` 安全。

```bash
# 1) 建備份分支（強烈建議）
git branch backup-before-reset-$(date +%Y%m%d-%H%M)

# 2) 回到目標版本（上一版示例）
git reset --hard HEAD~1

# 3) 強制推送（保護式）
git push --force-with-lease
```

> 若之後後悔，可切回備份分支再開 PR 或 cherry-pick 回來。

---

# 四、我該選哪個？

* **共同開發主分支**：用 **revert**。
* **個人練習或私人分支**：可用 **reset + --force-with-lease**。
* **只想暫時看舊版**：

  ```bash
  git checkout <commit_id>   # 進入分離頭狀態瀏覽
  git checkout main          # 回來
  ```

---

# 五、最小檢查清單

```bash
git status                # 乾淨？有無未存變更
git log --oneline -5      # 版本歷史是否正確
git remote -v             # 推送目標是否正確
git push / git push --force-with-lease   # 視情境推送
```





