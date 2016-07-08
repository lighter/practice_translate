# 每天使用Git的19小技巧

原文出處：[http://www.alexkras.com/19-git-tips-for-everyday-use/](http://www.alexkras.com/19-git-tips-for-everyday-use/)

我用Git已經4年了，想與大家分享一些實用的技巧。希望能幫助到你。

如果你是一個Git新手，我推薦你可先看看這篇文章[Git Cheat Sheet](http://www.alexkras.com/getting-started-with-git/)。這篇文章撰寫的對象是有3個月以上Git經驗的人。

# 指令清單

1. 善用 log 參數，`git log --online --graph`
2. 檔案的修改差異，`git log -p filename`
3. 特定檔案特定行數的變化，`git log -L 1,1:some-file.txt`
4. 顯示尚未合併到父(parent)分支，`git log --no-merge master..`
5. 取得特定檔案在特定分支的內容，`git show some-branch:some-file.js`
6. 使用rebase的注意事項，`git pull --rebase`
7. 記住本地分支合併後的結構，`git merge --no-ff`
8. 修正你的commit，`git commit --amend`
9. 三種Stage，之間如何切換，`git reset --hard HEAD and git status -s`
10. 復原 commit ，softly，`git revert -n`
11. 使用第三方軟體比對整個專案的差異，`git difftool -d`
12. 忽略空白，`git diff -w`
13. 加入檔案局部的修改，`git add -p`
14. 老舊的分支，`git branch -a`
15. 將檔案加入stash(儲藏)，`git stash -p`
16. 好的commit訊息
17. Git 自動完成
18. 將常用的指令建立一個新的別名
19. 找出造成你程式壞掉的commit

# 1. 善用 log 參數

範例： `git log --online --graph`

查看log時，推薦你幾個我最常使用的參數：

* `--author="Alex Kras` - 顯示特定作者(author)的commit
* `--name-only` - 顯示 commit 中修改的檔案
* `--oneline` - 僅用一行顯示每次的 commit
* `--graph` - 顯示 commit 的樹狀結構
* `--reverse` - 重最舊的 commit 顯示
* `--after` - 指定顯示日期之後的 commit
* `--before` - 指定顯示日期之前的 commit

例如，每週五主管需要我提交每週的報告。所以我會執行: `git log --author="Alex Kras" --after="1 week ago" --oneline`，然後修飾一下並提交給主管。

Git log有很多參數供你使用。`man git-log`可以看到很多參數的功能。

如果你不喜歡原本的輸出格式，`--pretty`參數提供你客製化的輸出格式。

![](http://www.alexkras.com/wp-content/uploads/git-log-oneline.png)

# 2. 顯示檔案的差異

範例：`git log -p filename`

`git log -p` 或 `git log -p filename` 不僅能看到 commit 訊息、作者、日期，還有實際檔案的差異內容。

你可以使用"反斜線(slash)"搜尋你要的關鍵字，例如`/{{your-search-here}}`。(使用`n`往下尋找下一個，`N`往上尋找上一個)。

![](http://www.alexkras.com/wp-content/uploads/git-log-search.png)

# 3. 特定檔案特定行數的變化

範例：`git log -L 1,1:some-file.txt`

`git blame filename`可以顯示檔案每行修改的人

![](http://www.alexkras.com/wp-content/uploads/git-blame.png)

`git blame`是個很棒的指令，但是有時提供你的資訊並不足夠。

另一個替代方案是使用`git log`搭配`-L`旗標(flag)。這個旗標可以讓你指定行數。如同`git log -p`顯示異動的內容。

`git log -L 1,1:some-file.txt`

![](http://www.alexkras.com/wp-content/uploads/git-log-lines.png)

# 4. 顯示尚未合併到父(parent)分支

範例：`git log --no-merges master..`

與多人共同開發時，你在某個分支開發了很久的時間，當你將父(parent)分支(例如, master)合併進你的分支時。這時你會發現很難看出你開發的分支與master的差異。

`git log --no-merges master..`指令可以解決這問題。`--no-merges`旗標為不顯示merge的訊息，**master..**參數為顯示尚未合併到master的commit(在maste之後必須加上`..`)。

`git show --no-merges master..` 和 `git log -p --no-merges maser..`不同的是第一個指令可以看到檔案的異動。

# 5. 取得特定檔案在特定分支的內容

範例：`git show some-branch:some-file.js`

這是個很方便的功能，無需切換分支，就可以查看檔案在其他分支的內容。

`git show some-branch-name:some-file-name.js`，會在終端機顯示該檔案在該分支的內容。

你也可以將檔案內容導出到指定的檔案上。

`git show some-branch-name:some-file-name.js > deleteme.js`

如果你只想比對該檔案在不同分支的差異，可以使用下面這行指令：

`git diff some-branch some-filename.js`

# 6. 使用rebase的注意事項

範例：`git pull --rebase`

上面提到很多log顯示merge commit的方法，但上述的問題，有些可以用`git rebase`來解決。

我認為rebase是一個進階的功能，甚至可以獨立出一篇文專專門描述rebase。

Git book提及rebase

>	rebase並不是沒有缺點的。請不要對於已經存在於遠端的commit做rebase的動作。如果你不遵守這個規則，你會被朋友討厭，甚至被家人嘲笑。

[https://git-scm.com/book/en/v2/Git-Branching-Rebasing#The-Perils-of-Rebasing](https://git-scm.com/book/en/v2/Git-Branching-Rebasing#The-Perils-of-Rebasing)

但並不要對於rebase感到害怕，你只需要小心的使用它。

而最好的方法就是使用rebase的互動模式(interactive rebasing)，`git rebase -i {{some commit hash}}`。它會開啟一個編輯器，並提供指令的解說。

![](http://www.alexkras.com/wp-content/uploads/git-rebase-i.png)

## `git pull --rebase`是個很有用的指令。

舉例來說，當你在本地端的master分支開發時，你commit了一個小改變。同時其他人將他本週的修改推上了遠地master分支。當你試著推上你的commit時，git會要求你先做`git pull`的動作。當你下了`git pull`時，git會幫你產生一個

> Merge remtoe-tracking branch 'origin/master'

的 commit。

雖然這沒什麼大不了的，但是這讓log看起來有點混亂了。

這個例子可以使用`git pull --rebase`。

這會先強制git抓取(pull)遠地的修改，然後再重新接上(re-apply[rebase])你尚未推到遠地端的commit，並且推上你的commit。這樣一來git自動產生的`Merge remote-tracking`的log就不會出現了。

# 7. 記住本地分支合併後的結構

範例：`git merge --no-ff`

每當有一個bug或是專案出現時，我很喜歡另外開啟一個新的分支。這樣的好處是讓我清楚地瞭解每個commit的任務是什麼。如果你在github上發過pull request，`git log --oneline --graph`可以顯示出你的分支合併記錄。

如果你在本地的做分支合併，你會發現git樹狀結構是一條直線。

如果你想要強調分支的歷史紀錄，你可以加入`--no-ff`標籤，這可以凸顯你的分支歷史樹狀結構。

`git merge --no-ff some-branch-name`

![](http://www.alexkras.com/wp-content/uploads/git-merge-no-ff.png)

# 8. 修正你的commit

範例：`git commit --amend`

如果你已經commit了，但是你發現commit的文字描述有錯誤時，這時你可以使用這個指令來重新編輯妳的commit

**注意！如果你尚未把commit push到遠端**，你可以遵照以下的步驟：

1.  修正你的錯字。
2. `git add some-fixed-file.js` 加入修正的檔案。
3. 執行`git commit --amend`，你將可以修正最後一次的commit。
4. 修正完畢時推上遠端的分支。

![](http://www.alexkras.com/wp-content/uploads/git-commit-amend.gif)

如果進行開發的分支，只屬於你擁有的，你當同步到遠端分支時，一樣可以進行修改commit訊息；當你修正完commit訊息時，執行`git push -f`(-f 是強制(force)的意思)，他會覆寫歷史紀錄。但是如果該分支有別人同時進行開發，切勿這麼做！

# 9. 三種Stage，之間如何切換(Three stages in git, and how to move between them)

範例：`git reset --hard HEAD`, `git status -s`

檔案在git有三種 stage

1. 撤回檔案(Not staged for commit)
2. 暫存檔案(Staged for commit)
3. 提交修改的檔案(commited)

`git status`可以看到更多詳盡的檔案狀態資訊。`git add filename.js`指令會將檔案加入為暫存檔案(staged file)，或者使用`git add .`將所有檔案加入為暫存檔案(staged file)。

執行`git status -s`可以看到檔案目前的狀態

![](http://www.alexkras.com/wp-content/uploads/git-stages.png)

`git status`並不會顯示提交修改檔案紀錄，你可以使用`git log`查看已經提交修改的歷史紀錄。

下面提供一些方法，供你檔案 stage 的狀態。

## reset 檔案

reset有三種方式。reset允許你任意還原到你指定的歷史紀錄。

1. `git reset --hard {{some-commit-hash}}` - 還原到你指定的commit hash。**你所有的改變將都會還原，不會保留**。
2. `git reset {{some-commit-hash}}` - 還原到你指定的commit hash。**所有改變都會保留，並且不會加入暫存檔案(unstaged files)**。而你可以在使用`git add .`和`git commit`後還原回去。
3. `git reset --soft {{some-commit-hash}}` - 還原到你指定的commit hash。**所有改變都會保留，並且加入暫存檔案(staged files)**。你只需要使用`git commit`即可還原。

這樣一來你就可以切換檔案到任一不同的版本。

以下是我最常使用的情形：

1. 我想要清除所有的修改 - `git reset --hard HEAD`。
2. 我想要修改某個已經提交的紀錄 - `git reset {{some-start-point-hash}}`。
3. 我想要合併3個commit為1個commit - `git reset --soft {{some-start-point-hash}}`。

## Check out 檔案

如果你想要還原檔案可以使用下面的指令

`git checkout forget-my-changes.js`

這很像`git reset --hard`但是只有還原你指定的檔案。

你也可以切換檔案到不同的commit版本，甚至不同分支的commit

`git checkout some-branch-name file-name.js`
`git checkout {{some-commit-hash}} file-name.js`

執行完checkout後，檔案會進入到暫存檔案(staged files)。`git reset HEAD file-name.js`將檔案丟入撤回檔案(unstaged files)。再次執行`git checkout file-name.js`就會回到檔案原本的版本。

`git reset --hard HEAD file-name.js`指令是無用的。stage的切換可能令你有點搞混，但我希望透過這個章節能讓你清楚stage的切換。

# 10. 復原 commit ，softly

範例：`git revert -n`

如果你想要復原先前提交的commit，可以使用revert來達到這個目的。

通常`git revert`會自動幫你建立一個新的commit，提醒你有做了一個revert的動作。`-n`這個參數，是告訴git復原並且不要自動完成commit；你可以試著自己親手操作看看。

# 11. 使用第三方軟體比對整個專案的差異

範例：`git difftool -d`

我非常喜歡的差異比對軟體是[Meld](http://www.alexkras.com/how-to-run-meld-on-mac-os-x-yosemite-without-homebrew-macports-or-think/)。

Meld可以與git diff, merge使用。執行下面的指令來修改你預設diff和merge開啟的工具：

```
git config --global diff.tool meld
git config --global merge.tool meld
```

之後執行`git difftool some-file.js`使用Meld來看檔案的差異。

**除此之外Meld也支援資料夾的差異比對。**

`git difftool`加上`-d`參數，會比對整個資料夾的差異。

![](http://www.alexkras.com/wp-content/uploads/git-difftool-d.png)

# 12. 忽略空白

範例：`git diff -w`或`git blame -w`

如果你對程式碼作縮排或是整個排版整理，`git blame`會顯示所有你的異動。

git很聰明，他知道你異動了什麼，只需要加上一個參數`-w`就可以將空白的異動忽略不顯示。

# 13. 加入檔案局部的修改

範例：`git add -p`

有些人使用git的時候很喜歡`-p`這個參數，因為他有些很便利的功能。

`git add -p`可讓你選擇檔案中哪些異動範圍是你想要加入暫存檔案(staged file)，並且準備commit的。

![](http://www.alexkras.com/wp-content/uploads/git-add-p.png)

# 14. 老舊的分支

範例：`git branch -a`

通常遠端的分支很多，有些分支也已經合併到master了。如果你像我一樣有些潔癖，這可能會讓你有點惱怒。

加上`-a`參數，可以看到所有的方知包含遠地，加上`--merged`參數，僅會列出完整merge到master的分支。

當你在`git fetch`時，可以加上`-p`參數，抓取遠地資訊的同時，也一併清理舊有的資料。

![](http://www.alexkras.com/wp-content/uploads/git-add-p.png)

`git for-each-ref --sort=committerdate --format='%(refname:short) * %(authorname) * %(committerdate:relative)' refs/remotes/ | column -t -s '*'`指令會列出所有遠端的分支，以及每個分支的最後一個commit。

![](http://www.alexkras.com/wp-content/uploads/fancy-branch-view.png)

不幸的是，僅想要列出已合併的分支並不是一件簡單的事情。如果你只想看到合併的分支commit，可能需要寫一個script去比對找出合併的分支。

# 15. 將檔案加入stash(儲藏)

範例：`git stash --keep-index`或`git statsh -p`

`git stash`就是將你修改過的檔案存放到一個堆疊(stack)，下次當你想要使用時，執行`git stash pop`就可以重新使用堆疊裡面修改的程式碼。`git stash list`可以看到所有你儲存的修改。使用`man git-stash`看一下還有哪些參數可以使用吧。

通常執行`git stash`會一次將所有檔案放到stash(儲藏)。有時你只想將特定的檔案放入stash(儲藏)，而其他的檔案繼續在你的工作區。

是否還記得前面使用過的`-p`這個參數?如同上述，在`git stash`後加上`-p`這個參數，會出現一個互動式的介面，針對一個修改區塊一個修改的區塊問你是否要加入stash(儲藏)。

![](http://www.alexkras.com/wp-content/uploads/git-stash-p.png)

當你只想將某些檔案加入stash，還有一個小技巧是：

1. 先使用`git add`將你不想放入stash的檔案加入到staged file，例如`git add file1.js, file2.js`。
2. 接著執行`git stash --keep-index。他會將你在unstaged file的檔案加入到stash。
3. 最後執行`git reset`，將staged file的檔案放回到unstaged file。

# 16. 好的commit訊息

不久之前，我看一篇文章，是有關於良好的commit訊息，文章在此[How to Write a Git Commit Message](http://chris.beams.io/posts/git-commit/)。

有一個我非常堅持的原則是，**每一個commit都應該是一個完整的句子**

**When applied,this commit will:**{{YOUR COMMIT MESSAGE}}

例如：

- When applied this commit will **Update README file**
- When applied this commit will **Add validation for GET /user/:id API call
- When applied this commit will Revert commit 12345

# 17. Git 自動補齊

在有些作業系統預設(如：Ubuntu)有支援git自動補齊。如果你的作業系統沒有支援(如：Mac)，你可以參考下列的指南來啟動自動補齊：

[https://git-scm.com/book/en/v1/Git-Basics-Tips-and-Tricks#Auto-Completion](https://git-scm.com/book/en/v1/Git-Basics-Tips-and-Tricks#Auto-Completion)

# 18. 將常用的指令建立一個新的別名

太長的指令不易閱讀，你可以對這個長的指令創建一個短的別名指令。

學習git最好的方式就是使用command line，而學習command line最好的方式就是自己親自動手下去做。

使用一段時間後，你會發現哪些指令是你最常使用的，而你可以針對這些指令建立一個短的別名。

例如：

`git config --global alisa.l "log --oneline --graph"`

上述指令是你建立一個別名為`l`的指令，當你執行`git l`，如同執行`git log --online --graph`。

而你一樣可以在別名的指令後加上其他的參數，例如`git l --author="Alex"`。

另一個做法是Bash的別名

在.bashrc檔案加上下面這行

`alias gil="git log --oneline --graph"`，這時你可以直接下`gil`指令，效果如同`git l`。

# 19. 找出造成你程式壞掉的commit

範例：`git bisect`

`git bisect`使用2分法，幫助你在許多commit中快速找到哪一個commit造成你程式無法正常執行。

當你結束一個長假，回到公司pull專案的最新的版本，尚未pull時確認過程式可以正常運作，
但是pull之後就壞了。

在你放假期間，專案中已經有好多commit了，你很難找出到底是在哪一次的commit造成的錯誤。

![](http://www.alexkras.com/wp-content/uploads/pulling-out-hair.jpg)

這時你可能嘗試找出是哪一隻程式出錯，然後使用`git blame`找出哪一行程式出錯，以及修改的人。

如果簡單的錯誤還可以，但如果這個錯誤很難找出問題原因，這時你可以試著從commit的歷史紀錄找出造成功能無法正常運作的commit。

`git bisect`可以幫助你快速找到出問題的commit。

## `git bisect`做了什麼?

首先你要告訴`git bisect`哪一次的commit是正常的，以及哪一次的commit是壞掉的。這時會切換到好與壞的commit中間點，接著你需要確認這個中間點commit是否為正常的。

如果這個中間點commit是正常的。你需要使用`git bisect good`告訴git這是正常的。接著會繼續往左邊的commit尋找新的中間點commit。

一直重複這個動作，`git bisect`會縮小範圍，找出出問題的commit。

下面是實際執行`git bisect`的流程：

1. `git bisect start` - 告訴git開始執行2分法。
2. `git bisect good {{some-commit-hash}}` - 告訴git哪一個commit是正常的(例如在你休假前最後一個commit)。
3. `git bisect bad {{some-commit-hash}}` - 告訴git哪一個commit是壞掉的(例如目前master分支最新的commit)。`git bisect bad HEAD`(HEAD表示為最新的commit)。
4. 此時會切換到好與壞的commit中間點，你必須確認這個中間點commit是否是正常的。
5. 如果是壞掉的再次執行`git bisect bad`告訴git這個commit是壞掉的。
6. 接著會在切換到一個新的commit，再次確認後，確定這個commit是正常的，使用`git bisect good`告訴git這個commit是正常的。
7. 當你找到開始出問題的commit，`git bisect`就已經完成任務了。
8. 執行`git bisect reset`回到你一開始執行`git bisect`的commit(例如你是在master分支最新的commit執行的)
9. `git bisect log` - 顯示`git bisect`最後一次成功的紀錄。

你也可以結合`git bisect`寫一個自動化的script。你可以參考[http://git-scm.com/docs/git-bisect#_bisect_run](http://git-scm.com/docs/git-bisect#_bisect_run)。

![](http://www.alexkras.com/wp-content/uploads/git-bisect.gif)


