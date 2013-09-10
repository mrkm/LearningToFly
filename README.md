LearningToFly
=============

# 3.2 Git のブランチ機能 - ブランチとマージの基本 #

ref.: [3.2 Git のブランチ機能 - ブランチとマージの基本][1]

## ブランチとマージの基本 ##
実際の作業に使うであろう流れを例にとって、ブランチとマージの処理を見てみましょう。次の手順で進めます。

1. ウェブサイトに関する作業を行っている
1. 新たな作業用にブランチを作成する
1. そのブランチで作業を行う

ここで、重大な問題が発生したので至急対応してほしいという連絡を受けました。その後の流れは次のようになります。

1. 実運用環境用のブランチに戻る
1. 修正を適用するためのブランチを作成する
1. テストをした後で修正用ブランチをマージし、実運用環境用のブランチにプッシュする
1. 元の作業用ブランチに戻り、作業を続ける

### ブランチの基本 ###
まず、すでに数回のコミットを済ませた状態のプロジェクトで作業をしているものと仮定します。

    master
    |
    C2 -> C1 -> C0

短くて単純なコミットの歴史

ここで、あなたの勤務先で使っている何らかの問題追跡システムに登録されている問題番号 53 への対応を始めることにしました。念のために言っておくと、Git は何かの問題追跡システムと連動しているわけではありません。しかし、今回の作業はこの問題番号 53 に対応するものであるため、作業用に新しいブランチを作成します。ブランチの作成と新しいブランチへの切り替えを同時に行うには、git checkout コマンドに -b スイッチをつけて実行します。
    $ git checkout -b iss53
    Switched to a new branch "iss53"

これは、次のコマンドのショートカットです。

    $ git branch iss53
    $ git checkout iss53

結果を示します。

                 master   
                 |
    C0 <- C1 <- C2
                 |
                 iss53

ウェブサイト上で何らかの作業をしてコミットします。そうすると iss53 ブランチが先に進みます。このブランチをチェックアウトしているからです (つまり、HEAD が iss53 ブランチを指しているということです。)。

                 master   
                 |
    C0 <- C1 <- C2 <- C3
                       |
                       iss53

作業した結果、iss53 ブランチが移動した

ここで、ウェブサイトに別の問題が発生したという連絡を受けました。そっちのほうを優先して対応する必要があるとのことです。Git を使っていれば、ここで iss53 に関する変更をリリースしてしまう必要はありません。また、これまでの作業をいったん元に戻してから改めて優先度の高い作業にとりかかるなどという大変な作業も不要です。ただ単に、master ブランチに戻るだけでよいのです。

しかしその前に注意すべき点があります。作業ディレクトリやステージングエリアに未コミットの変更が残っている場合、それがもしチェックアウト先のブランチと衝突する内容ならブランチの切り替えはできません。ブランチを切り替える際には、クリーンな状態にしておくのが一番です。これを回避する方法もあります (stash およびコミットの amend という処理です) が、また後ほど説明します。今回はすべての変更をコミットし終えているので、master ブランチに戻ることができます。


    $ git checkout master
    Switched to branch "master"

作業ディレクトリは問題番号 53 の対応を始める前とまったく同じ状態に戻りました。これで、緊急の問題対応に集中できます。ここで覚えておくべき重要な点は、Git が作業ディレクトリの状態をリセットし、チェックアウトしたブランチが指すコミットの時と同じ状態にするということです。そのブランチにおける直近のコミットと同じ状態にするため、ファイルの追加・削除・変更を自動的に行います。

***commit後にmasterブランチに戻ったら、確かにC2に戻ってた。iss53へ戻るとC3になった。***

次に、緊急の問題対応を行います。緊急作業用に hotfix ブランチを作成し、作業をそこで進めるようにしましょう。


    $ git checkout -b hotfix
    Switched to a new branch "hotfix"
    $ vim index.html
    $ git commit -a -m 'fixed the broken email address'
    [hotfix]: created 3a0874c: "fixed the broken email address"
     1 failes changed, 0 insertions(+), 1 deletions(-)

結果を示します。

                master hotfix
                 |     |
    C0 <- C1 <- C2 <- C4
                   <- C3
                       |
                       iss53

master ブランチから新たに作成した hotfix ブランチ

テストをすませて修正がうまくいったことを確認したら、master ブランチにそれをマージしてリリースします。ここで使うのが git merge コマンドです。

    $ git checkout master
    $ git merge hotfix
    Updating f42c576..3a0874c
    Fast forward
     README |    1 -
     1 files changed, 0 insertions(+), 1 deletions(-)

***masterに戻ってhotfixをmerge。git-flowなら、developに戻ってfeatureをmergeのようなイメージか。***

このマージ処理で "Fast forward" というフレーズが登場したのにお気づきでしょうか。マージ先のブランチが指すコミットがマージ元のコミットの直接の親であるため、Git がポインタを前に進めたのです。言い換えると、あるコミットに対してコミット履歴上で直接到達できる別のコミットをマージしようとした場合、Git は単にポインタを前に進めるだけで済ませます。マージ対象が分岐しているわけではないからです。この処理のことを "fast forward" と言います。

変更した内容が、これで master ブランチの指すスナップショットに反映されました。これで変更をリリースできます。

                       master, hotfix
                       |
    C0 <- C1 <- C2 <- C4
                   <- C3
                       |
                       iss53

マージした結果、master ブランチの指す先が hotfix ブランチと同じ場所になった

超重要な修正作業が終わったので、横やりが入る前にしていた作業に戻ることができます。しかしその前に、まずは hotfix ブランチを削除しておきましょう。master ブランチが同じ場所を指しているので、もはやこのブランチは不要だからです。削除するには git branch で -d オプションを指定します。

    $ git branch -d hotfix
    Deleted branch hotfix (3a0874c).

では、先ほどまで問題番号 53 の対応をしていたブランチに戻り、作業を続けましょう。

    $ git checkout iss53
    Switched to branch "iss53"
    $ vim index.html
    $ git commit -a -m 'finished the new footer [issue 53]'
    [iss53]: created ad82d7a: "finished the new footer [issue 53]"
     1 files changed, 1 insertions(+), 0 deletions(-)

.

                       master
                       |
    C0 <- C1 <- C2 <- C4
                   <- C3 <- C5
                             |
                             iss53

iss53 ブランチは独立して進めることができる

ここで、hotfix ブランチ上で行った作業は iss53 ブランチには含まれていないことに注意しましょう。もしそれを取得する必要があるのなら、方法はふたつあります。ひとつは git merge master で master ブランチの内容を iss53 ブランチにマージすること。そしてもうひとつはそのまま作業を続け、いつか iss53 ブランチの内容を master に適用することになった時点で統合することです。

### マージの基本 ###

問題番号 53 の対応を終え、master ブランチにマージする準備ができたとしましょう。iss53 ブランチのマージは、先ほど hotfix ブランチをマージしたときとまったく同じような手順でできます。つまり、マージ先のブランチに切り替えてから git merge コマンドを実行するだけです。

    $ git checkout master
    $ git merge iss53
    Merge made by recursive.
     README |    1 +
     1 files changed, 1 insertions(+), 0 deletions(-)

先ほどの hotfix のマージとはちょっとちがう感じですね。今回の場合、開発の歴史が過去のとある時点で分岐しています。マージ先のコミットがマージ元のコミットの直系の先祖ではないため、Git 側でちょっとした処理が必要だったのです。ここでは、各ブランチが指すふたつのスナップショットとそれらの共通の先祖との間で三方向のマージを行いました。図 3-16 に、今回のマージで使用した三つのスナップショットを示します。

                       master
                       |
    C0 <- C1 <- C2 <- C4              C2: Common Ancestor
                   <- C3 <- C5        C4: Snapshot to Merge Into
                             |        C5: Snapshot to Merge In
                             iss53

Git が共通の先祖を自動的に見つけ、ブランチのマージに使用する

単にブランチのポインタを先に進めるのではなく、Git はこの三方向のマージ結果から新たなスナップショットを作成し、それを指す新しいコミットを自動作成します。これはマージコミットと呼ばれ、複数の親を持つ特別なコミットとなります。

マージの基点として使用する共通の先祖を Git が自動的に判別するというのが特筆すべき点です。CVS や Subversion (バージョン 1.5 より前のもの) は、マージの基点となるポイントを自分で見つける必要があります。これにより、他のシステムに比べて Git のマージが非常に簡単なものとなっているのです。

                                   master
                                   |
    C0 <- C1 <- C2 <- C4 <------- C6
                    \           /
                     - C3 <- C5
                             |
                             iss53

マージ作業の結果から、Git が自動的に新しいコミットオブジェクトを作成する

これで、今までの作業がマージできました。もはや iss53 ブランチは不要です。削除してしまい、問題追跡システムのチケットもクローズしておきましょう。

    $ git branch -d iss53

### マージ時のコンフリクト ###

物事は常にうまくいくとは限りません。同じファイルの同じ部分をふたつのブランチで別々に変更してそれをマージしようとすると、Git はそれをうまくマージする方法を見つけられないでしょう。問題番号 53 の変更が仮に hotfix ブランチと同じところを扱っていたとすると、このようなコンフリクトが発生します。

    $ git merge iss53
    Auto-merging index.html
    CONFLICT (content): Merge conflict in index.html
    Automatic merge failed; fix conflicts and then commit the result.

Git は新たなマージコミットを自動的には作成しませんでした。コンフリクトを解決するまで、処理は中断されます。コンフリクトが発生してマージできなかったのがどのファイルなのかを知るには git status を実行します。

    [master*]$ git status
    index.html: needs merge
    # On branch master
    # Changes not staged for commit:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #   unmerged:   index.html
    #

コンフリクトが発生してまだ解決されていないものについては unmerged として表示されます。Git は、標準的なコンフリクトマーカーをファイルに追加するので、ファイルを開いてそれを解決することにします。コンフリクトが発生したファイルの中には、このような部分が含まれています。

    <<<<<<< HEAD:index.html
    <div id="footer">contact : email.support@github.com</div>
    =======
    <div id="footer">
      please contact us at support@github.com
    </div>
    >>>>>>> iss53:index.html

これは、HEAD (merge コマンドを実行したときにチェックアウトしていたブランチなので、ここでは master となります) の内容が上の部分 (======= の上にある内容)、そして iss53 ブランチの内容が下の部分であるということです。コンフリクトを解決するには、どちらを採用するかをあなたが判断することになります。たとえば、ひとつの解決法としてブロック全体を次のように書き換えます。

    <div id="footer">
    please contact us at email.support@github.com
    </div>

このような解決を各部分に対して行い、<<<<<<< や ======= そして >>>>>>> の行をすべて除去します。そしてすべてのコンフリクトを解決したら、各ファイルに対して git add を実行して解決済みであることを通知します。ファイルをステージすると、Git はコンフリクトが解決されたと見なします。コンフリクトの解決をグラフィカルに行いたい場合は git mergetool を実行します。これは、適切なビジュアルマージツールを立ち上げてコンフリクトの解消を行います。

    $ git mergetool
    merge tool candidates: kdiff3 tkdiff xxdiff meld gvimdiff opendiff emerge vimdiff
    Merging the files: index.html

    Normal merge conflict for 'index.html':
      {local}: modified
      {remote}: modified
    Hit return to start merge resolution tool (opendiff):

デフォルトのツール (Git は opendiff を選びました。私がこのコマンドを Mac で実行したからです) 以外のマージツールを使いたい場合は、“merge tool candidates”にあるツール一覧を見ましょう。そして、使いたいツールの名前を打ち込みます。第 7 章で、環境にあわせてこのデフォルトを変更する方法を説明します。

マージツールを終了させると、マージに成功したかどうかを Git が聞いてきます。成功したと伝えると、ファイルを自動的にステージしてコンフリクトが解決したことを示します。

再び git status を実行すると、すべてのコンフリクトが解決したことを確認できます。

    $ git status
    # On branch master
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #   modified:   index.html
    #

結果に満足し、すべてのコンフリクトがステージされていることが確認できたら、git commit を実行してマージコミットを完了させます。デフォルトのコミットメッセージは、このようになります。

    Merge branch 'iss53'
    
    Conflicts:
      index.html
    #
    # It looks like you may be committing a MERGE.
    # If this is not correct, please remove the file
    # .git/MERGE_HEAD
    # and try again.
    #

このメッセージを変更して、どのようにして衝突を解決したのかを詳しく説明しておくのもよいでしょう。後から他の人がそのマージを見たときに、あなたがなぜそのようにしたのかがわかりやすくなります。

- try to merge a pull request

[1]: http://git-scm.com/book/ja/Git-%E3%81%AE%E3%83%96%E3%83%A9%E3%83%B3%E3%83%81%E6%A9%9F%E8%83%BD-%E3%83%96%E3%83%A9%E3%83%B3%E3%83%81%E3%81%A8%E3%83%9E%E3%83%BC%E3%82%B8%E3%81%AE%E5%9F%BA%E6%9C%AC "3.2 Git のブランチ機能 - ブランチとマージの基本"
