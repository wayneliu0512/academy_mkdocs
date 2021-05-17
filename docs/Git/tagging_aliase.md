## 標籤
跟大多數的版本管理系統一樣，Git 有能力對專案歷史中比較特別的時間點貼標籤，來表示其重要性。

### 列出你的標籤
想要列出 Git 中所有標籤的方法非常直覺。 只要輸入 `git tag` 如下：

```
$ git tag
v0.1
v1.3
```

你也可以使用特定的 pattern 來搜尋標籤。 舉例來說，在 Git 原始碼的版本庫中，已經包含了超過 500 個標籤。 如果你只想看到 1.8.5 系列的標籤，你可以執行以下指令：

```
$ git tag -l "v1.8.5*"
v1.8.5
v1.8.5-rc0
v1.8.5-rc1
v1.8.5-rc2
v1.8.5-rc3
v1.8.5.1
v1.8.5.2
v1.8.5.3
v1.8.5.4
v1.8.5.5
```

### 建立新的標籤
Git 主要使用兩種類型的標籤：輕量級標籤和有註解的標籤。

一個輕量級的標籤就像是一個不會移動的分支——這個標籤只會指向一個特定的提交。

然而，有註解的標籤，會在 Git 的資料庫中儲存成完整的物件。 它們將被計算校驗碼；包含貼標籤那個人的名字、電子郵件和日期；能夠紀錄一個標籤訊息；

### 有註解的標籤
建立一個有註解的標籤很簡單。 最簡單的方法是在你建立標籤時，同時指定 `-a` 的選項如下:

```
$ git tag -a v1.4 -m "my version 1.4"
$ git tag
v0.1
v1.3
v1.4
```

指令中的 `-m` 選項後面同時指定了一個標籤訊息，這個訊息會和這個標籤一起保存。 如果你沒有為標籤指定一個訊息，那麼 Git 會開啟你的編輯器以便你輸入。

當你使用 `git show` 指令時，你可以查看標籤的資訊，還有這個標籤所標記的提交資訊如下：

```
$ git show v1.4
tag v1.4
Tagger: Ben Straub <ben@straub.cc>
Date:   Sat May 3 20:19:12 2014 -0700

my version 1.4

commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number
```

### 輕量級標籤
另外一種能標記提交的標籤是輕量級標籤。 這基本上是把該提交的校驗碼存在一個檔案中，不包含其他資訊。 如果想要建立一個輕量級的標籤，不要指定 `-a`、`-s` 或 `-m` 的選項如下：

```
$ git tag v1.4-lw
$ git tag
v0.1
v1.3
v1.4
v1.4-lw
v1.5
```

此時如果對該標籤使用 `git show`，你將不會看到這個標籤的額外資訊。 這個指令就只會顯示標籤所在的提交資訊：

```
$ git show v1.4-lw
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number
```

### 對以前的提交貼標籤
你也可以對過去的提交貼標籤。 假設你的提交歷史看起來如下：

```
$ git log --pretty=oneline
15027957951b64cf874c3557a0f3547bd83b3ff6 Merge branch 'experiment'
a6b4c97498bd301d84096da251c98a07c7723e65 beginning write support
0d52aaab4479697da7686c15f77a3d64d9165190 one more thing
6d52a271eda8725415634dd79daabbc4d9b6008e Merge branch 'experiment'
0b7434d86859cc7b8c3d5e1dddfed66ff742fcbc added a commit function
4682c3261057305bdd616e23b64b0857d832627b added a todo file
166ae0c4d3f420721acbb115cc33848dfcc2121a started write support
9fceb02d0ae598e95dc970b74767f19372d61af8 updated rakefile
964f16d36dfccde844893cac5b347e7b3d44abbc commit the todo
8a5cbc430f1a9c3d00faaeffd07798508422908a updated readme
```

現在，假設你忘記在專案的「updated rakefile」提交貼 v1.2 的標籤。 你可以在後來再補貼標籤。 要在那個提交上面貼標籤，你需要在指令後面指定那個提交的校驗碼（可以省略後半段）：

```
$ git tag -a v1.2 9fceb02
```

```
$ git tag
v0.1
v1.2
v1.3
v1.4
v1.4-lw
v1.5

$ git show v1.2
tag v1.2
Tagger: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Feb 9 15:32:16 2009 -0800

version 1.2
commit 9fceb02d0ae598e95dc970b74767f19372d61af8
Author: Magnus Chacon <mchacon@gee-mail.com>
Date:   Sun Apr 27 20:43:35 2008 -0700

    updated rakefile
...
```

### 分享標籤
`git push` 指令預設不會傳送標籤到遠端伺服器。 在你建立標籤後，你必須明確的要求 Git 將標籤推送到共用的伺服器上面。 這個動作就像是在分享遠端分支一樣——你可以執行 `git push origin [tagname]`。

```
$ git push origin v1.5
Counting objects: 14, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (12/12), done.
Writing objects: 100% (14/14), 2.05 KiB | 0 bytes/s, done.
Total 14 (delta 3), reused 0 (delta 0)
To git@github.com:schacon/simplegit.git
 * [new tag]         v1.5 -> v1.5
```

如果想要一次推送很多標籤，也可以在使用 `git push` 指令的時候加上 `--tags` 選項。 這將會把你所有不在伺服器上面的標籤傳送給遠端伺服器。

```
$ git push origin --tags
Counting objects: 1, done.
Writing objects: 100% (1/1), 160 bytes | 0 bytes/s, done.
Total 1 (delta 0), reused 0 (delta 0)
To git@github.com:schacon/simplegit.git
 * [new tag]         v1.4 -> v1.4
 * [new tag]         v1.4-lw -> v1.4-lw
```

現在，當其他人從版本庫克隆或拉取時，他們就能同時拿到你所貼的標籤，

## Git Aliases
在結束「Git 基礎」這個章節以前，在此想和你分享一些使用 Git 的技巧，讓你能夠更簡易且友善的使用 Git——別名（alias）。

你可以輕易的使用 git config 來替指令設定別名。 下面有一些你可能會想要設定別名的範例：

```
$ git config --global alias.co checkout
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status
```

舉其中一個例子來說，這樣的設定意味著你可以只打 `git ci` 而不需要打 `git commit`。 隨著你深入使用 Git，你將會發現某些指令用的很頻繁，不要猶豫，馬上建立新的指令別名。

舉例來說，為了提高 `unstage` 檔案的方便性，你可以加入你自己的 `unstage` 別名：

```
$ git config --global alias.unstage 'reset HEAD --'
```

而且這個 unstage 別名會讓以下兩個指令有相同的功用：

```
$ git unstage fileA
$ git reset HEAD -- fileA
```

此外，大家通常還會新增一個 last 指令如下：

```
$ git config --global alias.last 'log -1 HEAD'
```

如此一來，你可以更簡易的看到最後的提交訊息：

```
$ git last
commit 66938dae3329c7aebe598c2246a8e6af90d04646
Author: Josh Goebel <dreamer3@example.com>
Date:   Tue Aug 26 19:48:51 2008 +0800

    test for current head

    Signed-off-by: Scott Chacon <schacon@example.com>
```
