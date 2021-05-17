## 檢視提交的歷史記錄
在產生數筆提交（commit）或者克隆（clone）一個已有歷史記錄的版本庫之後，你或許會想要檢視之前發生過什麼事； 最基本也最具威力的工具就是 `git log` 命令。

在專案目錄內執行 `git log`，你應該會看到類似以下訊息：

```
$ git log
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 16:40:33 2008 -0700

    removed unnecessary test

commit a11bef06a3f659402fe7563abf99ad00de2209e6
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 10:31:28 2008 -0700

    first commit
```

預設情況（未加任何選項）`git log` 以反向的時間順序列出版本庫的提交歷史記錄——也就是說最新的提交會先被列出來； 如你所見，它也會列出每筆提交的 SHA-1 校驗碼、作者名字及電子郵件、寫入日期以及提交訊息。

`git log` 命令有大量且多樣的選項，能精確地找出你想搜尋的結果； 在這裡，我們會展示一些最受歡迎的選項。

| 選項              | 說明                                                                                      |
| :---------------- | :---------------------------------------------------------------------------------------- |
| `-p`              | 顯示每筆提交的補綴。                                                                      |
| `--stat`          | 顯示每筆提交中更動檔案的統計及摘要資訊。                                                  |
| `--shortstat`     | 只顯示 --stat 提供的的訊息中關於更動、插入、刪除的文字。                                  |
| `--name-only`     | 在提交訊息後方顯示更動的檔案列表。                                                        |
| `--name-status`   | 在檔案列表顯示「新增」、「更動」、「刪除」等資訊。                                        |
| `--abbrev-commit` | 只顯示 SHA-1 校驗碼的前幾位數，而不是顯示全部 40 位數。                                   |
| `--relative-date` | 以相對時間格式顯示日期（例如：「2 weeks ago」），而不是使用完整的日期格式。               |
| `--graph`         | 在輸出的日誌旁邊顯示分支及合併歷史的 ASCII 圖形。                                         |
| `--pretty`        | 以其它格式顯示提交。選項包括 `oneline`、`short`、`full`、`fuller` 及可自訂格式的 format。 |

### 限制日誌的輸出
除了輸出格式的選項以外，`git log` 還有一些有用的輸出限制選項，例如:用 `-2` 選項只顯示最後二筆提交。

或者像 `--since` 和 `--until` 這些限制時間的選項也很有用； 例如，以下命令列出最近兩週以來的提交：

```
$ git log --since=2.weeks
```

另一個實用的選項是 `-S`，用來尋找所修改的內容中被加入或移除某字串的提交； 擧例，如果你想要找出最後一個有加入或移除某個特定函數參照的提交，你可以使用：

```
$ git log -Sfunction_name
```

最後一個實用的 `git log` 過濾選項是路徑， 如果你指定一個目錄或檔名，你可以列出只對這些檔案有修改記錄的提交； 這個選項永遠放在最後一個，並且通常使用二個連接號（`--`）將路徑與其它選項隔開。

| 選項                | 說明                                       |
| :------------------ | :----------------------------------------- |
| `-(n)`              | 只顯示最後 n 筆提交。                      |
| `--since, --after`  | 列出特定日期後的提交。                     |
| `--until, --before` | 列出特定日期前的提交。                     |
| `--author`          | 列出作者名字符合指定字串的提交。           |
| `--committer`       | 列出提交者名字符合指定字串的提交。         |
| `--grep`            | 列出提交訊息中符合指定字串的提交。         |
| `-S`                | 列出修改檔案中有加入或移除指定字串的提交。 |


例如：如果你想檢視 Git 原始碼的測試檔案中（註：它們都放在資料夾 `t/`），由 Junio Hamano 在 2008 年 10 月份所提交，但不包含「合併提交」的提交。可執行以下的命令：

```
$ git log --pretty="%h - %s" --author=gitster --since="2008-10-01" \
   --before="2008-11-01" --no-merges -- t/
5610e3b - Fix testcase failure when extended attributes are in use
acd3b9e - Enhance hold_lock_file_for_{update,append}() API
f563754 - demonstrate breakage of detached checkout with symbolic link HEAD
d1a43f2 - reset --hard/read-tree --reset -u: remove unmerged new paths
51a94af - Fix "checkout --track -b newbranch" on detached HEAD
b0ad11e - pull: allow "git pull origin $something:$current_branch" into an unborn branch
```

## 復原

在任何一個過程中，你都可能想要復原某些內容， 在這裡我們會回顧一些基本的工具用來復原你做過的修改內容； 小心！因為復原操作不是永遠都可逆的， 這是少數在使用 Git 時，執行錯誤的動作會遺失資料的情況。

一個常見的復原操作發生在當你太早提交（`commit`），接著才發現忘了加入某些檔案，或者寫錯了提交訊息； 如果你想要重新提交，你可以在提交命令上使用 `--amend` 選項：

```
$ git commit --amend
```

這個命令會再次把預存區（staging area）拿來提交。

例如：如果你提交後才意識到你想要把某些忘記預存（stage）的修改也一併加入到上一個提交中，你可以這樣做：

```
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```

最終只會得到一個提交——第二次的提交取代了第一次提交的結果。

### 將已預存的檔案移出預存區
接下來的兩節會展示如何操作預存區和工作目錄中已修改的檔案；
 例如：假設你已經修改了二個檔案，並且想要分別提交它們，但是你卻意外地使用 `git add *` 把它們二個都預存了， 要如何將其中一個「移出預存區（unstage）」呢？`git status` 命令提示你：

```
$ git add *
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
    modified:   CONTRIBUTING.md
```

在「Changes to be committed」文字正下方，說明了使用 `git reset HEAD <file>...` 將檔案移出預存區； 因此，讓我們依循該建議將 `CONTRIBUTING.md` 檔案移出預存區：

```
$ git reset HEAD CONTRIBUTING.md
Unstaged changes after reset:
M	CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

關於 `git reset` 命令，後面章節會在深入介紹更多細節。

### 復原被修改的檔案
當你不想要保留 `CONTRIBUTING.md `檔案的修改時該怎麼辦？ 你如何才能輕易地復原它——將它還原到上次提交時的樣子（或最初克隆時、或當初放到工作目錄時的版本）？ 很幸運的，`git status` 也告訴你該如何做； 在上一個範例的輸出中，有修改而未預存的內容長得像這樣：

```
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

它相當明確地提示你如何捨棄工作目錄所做的修改， 讓我們跟著提示做：

```
$ git checkout -- CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
```

你可以看到那些修改已經被還原了。

#### 重要提醒
你必需明瞭一件很重要的事：`git checkout -- <file>` 是一個危險的命令， 你對那個檔案所做的任何修改都會消失——Git 只是複製了另一個檔案來覆蓋它； 除非你很肯定地知道你不想要那個檔案了，否則千萬不要使用這個命令。

如果你仍然想保留那個檔案所做的修改，但是某個當下需要先復原檔案，我們將會在 使用 Git 分支章節 中介紹「收藏（stashing）」和「分支（branching）」，一般而言它們是比較好的做法。