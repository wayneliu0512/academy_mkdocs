## 與遠端協同工作

### 顯示你的遠端
使用 `git remote` 命令可以檢視你已經設定好的遠端版本庫， 它會列出每個遠端版本庫的「簡稱」。 如果你克隆（clone）了一個遠端版本庫，你至少看得到「origin」——它是 Git 給定的預設簡稱，用來代表被克隆的來源。 

```
$ git remote
origin
```

你也可以指定 -v 選項來顯示 Git 用來讀寫遠端簡稱時所用的網址。

```
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
```

如果遠端版本庫不止一個，這個命令會將它們全部列出來。 例如，一個版本庫內連結了許多其它協作者的遠端版本庫，可能看起來就像這樣：

```
$ cd grit
$ git remote -v
bakkdoor  https://github.com/bakkdoor/grit (fetch)
bakkdoor  https://github.com/bakkdoor/grit (push)
cho45     https://github.com/cho45/grit (fetch)
cho45     https://github.com/cho45/grit (push)
defunkt   https://github.com/defunkt/grit (fetch)
defunkt   https://github.com/defunkt/grit (push)
koke      git://github.com/koke/grit.git (fetch)
koke      git://github.com/koke/grit.git (push)
origin    git@github.com:mojombo/grit.git (fetch)
origin    git@github.com:mojombo/grit.git (push)
```

### 新增遠端版本庫
我們在之前的章節中已經提過並給了一些範例來說明 `clone` 命令如何隱式地為你加入 `origin` 遠端； 而在這裡將說明如何「明確地」新增一個遠端。 選一個你可以輕鬆引用的簡稱，用來代表要新增的遠端 Git 版本庫，然後執行 `git remote add <簡稱> <url>` 來新增它：

```
$ git remote
origin
$ git remote add pb https://github.com/paulboone/ticgit
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
pb	https://github.com/paulboone/ticgit (fetch)
pb	https://github.com/paulboone/ticgit (push)
```

### 從你的遠端獲取或拉取
若要從你的遠端專案中取得資料，你可以執行：

```
$ git fetch [remote-name]
```

例如，如果你想從 Paul 的版本庫中取得所有資訊，而這些資訊並不存在於你的版本庫中，你可以執行 `git fetch pb`：

```
$ git fetch pb
remote: Counting objects: 43, done.
remote: Compressing objects: 100% (36/36), done.
remote: Total 43 (delta 10), reused 31 (delta 5)
Unpacking objects: 100% (43/43), done.
From https://github.com/paulboone/ticgit
 * [new branch]      master     -> pb/master
 * [new branch]      ticgit     -> pb/ticgit
```

這個命令會連到遠端專案，然後從遠端專案中將你還沒有的資料全部拉下來； 執行完成後，你應該會有那個遠端版本庫中所有分支的參照（reference）（譯註：再次強調，遠端的分支在本地端的分身——遠端追蹤分支），可以隨時用來合併或檢視。

很重要的一點是：`git fetch` 命令只會下載資料到你的版本庫——它並不會自動合併你的任何工作內容。

如果你的目前分支被設定為「追蹤」遠端上的分支（閱讀下一節以及 使用 Git 分支 以了解更多資訊），你便可使用 `git pull` 命令來自動「獲取」並「合併」那個遠端分支到你目前的分支裡去；由於 `git clone` 命令會「自動地」將本地分支 master 設定為「追蹤」遠端上的 master（無論預設分支叫什麼名稱。譯註：只要是預設分支都會自動設定追踨行為，而 master 常常是預設分支）

### 推送到你的遠端
當你想分享你的專案成果時，你必需將它推送到上游。 推送的命令很簡單：`git push [remote-name] [branch-name]`。 如果你想要將 master 分支推送到 origin 伺服器上時（再次說明，克隆時通常會自動地幫你設定好 master 和 origin 這二個名稱），那麼你可以執行這個命今將所有你完成的提交（commit）推送回伺服器上。

```
$ git push origin master
```

只有在你對克隆來源的伺服器有寫入權限，並且在這個當下還沒有其它人推送過，這個命令才會成功； 如果你和其它人同時做了克隆，然後他們先推送到上游，接著換你推送到上游，毫無疑問地你的推送會被拒絕； 你必需先獲取他們的工作內容，將其整併到你之前的工作內容，如此你才會被允許推送。

### 檢視遠端
如果你想要對一個特定遠端檢視更多資訊，你可以使用 `git remote show [remote-name]` 命令。 如果你在執行這個命令中使用特定的簡稱，例如 origin，你會得到下面這個類似的訊息：

```
$ git remote show origin
* remote origin
  Fetch URL: https://github.com/schacon/ticgit
  Push  URL: https://github.com/schacon/ticgit
  HEAD branch: master
  Remote branches:
    master                               tracked
    dev-branch                           tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```

它同時列出了遠端版本庫的網址和「追蹤分支（tracking branch）」資訊。 這個命令很有用地告訴你：目前分支是 master（譯注：HEAD 意味著目前的）。

下面是另一個例子:

```
$ git remote show origin
* remote origin
  URL: https://github.com/my-org/complex-project
  Fetch URL: https://github.com/my-org/complex-project
  Push  URL: https://github.com/my-org/complex-project
  HEAD branch: master
  Remote branches:
    master                           tracked
    dev-branch                       tracked
    markdown-strip                   tracked
    issue-43                         new (next fetch will store in remotes/origin)
    issue-45                         new (next fetch will store in remotes/origin)
    refs/remotes/origin/issue-11     stale (use 'git remote prune' to remove)
  Local branches configured for 'git pull':
    dev-branch merges with remote dev-branch
    master     merges with remote master
  Local refs configured for 'git push':
    dev-branch                     pushes to dev-branch                     (up to date)
    markdown-strip                 pushes to markdown-strip                 (up to date)
    master                         pushes to master                         (up to date)
```

這個命令顯示了當你在特定的分支上執行 git push 時，它將自動推送到哪一個遠端分支； 它也顯示了：哪些遠端分支是在你的本地端還沒有的（譯註：new 屬性）、哪些你曾獲取過的遠端分支已經在遠端上被移除了（譯註：stale 屬性）、哪些本地分支是有能力在執行 git pull 後自動和它們的遠端追蹤分支合併。

### 移除或重新命名遠端
你可以執行 `git remote rename` 來重新命名遠端的簡稱。 例如：如果你想要將 `pb` 重新命名為 `paul`，你可以這樣使用 `git remote rename`：

```
$ git remote rename pb paul
$ git remote
origin
paul
```

如果你因為某些原因想要移除一個遠端——你搬動了伺服器、或者不再使用某個特定的鏡像、或者某個貢獻者不再貢獻了——你可以執行 git remote rm：

```
$ git remote rm paul
$ git remote
origin
```