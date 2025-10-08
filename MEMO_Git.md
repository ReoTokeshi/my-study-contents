
ルートディレクトリ `C:\Program Files\Git`<br>
　⇒GitBash起動時のカレント<br>
ホームディレクトリ `C:\Users\81808`

デフォルトブランチ名 `main`<br>
　⇒git configのinit.fefaultBranchで確認＆設定

core.editorの値を`"code --wait"`に設定で、コミット時にVSCode起動。

| コマンド | 内容 |
| --- | --- |
| cd ~ | ホームディレクトリへ移動 |
| pwd | カレントディレクトリの絶対パス表示 |
| ls -a | 内容すべて表示 |
| git config --list | .gitconfig内の各設定値の確認 |
| git config --global _<設定項目>_ _<設定値>_ | 項目の設定 |
| git remote -v | 対応するリモートリポジトリの確認。<br>デフォルトではoriginと名前がついてる。 |
|  |  |
| git init | カレントでローカルリポジトリ作成 |
| git add _<ファイル名>_ | ステージングエリアへ登録 |
| git commit | コミット。VSCodeでコメント編集 |
| git status | ワークツリー内の状態確認 |
| git diff | 差分確認（ワークツリー⇔ステージングエリア） |
| git diff --cached | 差分確認（ステージングエリア⇔Gitディレクトリ） |
| git log (-p) | コミット履歴表示。-p付で差分も表示。 |
|  |  |
| git restore _<ファイル名>_ | ワークツリーの変更をコミット済のものに戻す<br><u>（ステージングエリアはそのまま）</u> |
| git restore --staged _<ファイル名>_ | ステージングエリアにaddした変更をコミット済のものに戻す<br><u>（ワークツリーはそのまま）</u> |
| git restore --staged --worktree _<ファイル名>_ | ワークツリー・ステージングエリア両方同時に戻す |
| git rm _<ファイル名>_ | ステージングエリアに削除を反映する<br>（そのあとコミット） |
| git rm -r _<ディレクトリ名>_ | フォルダごと削除 |
|  |  |

実行結果が途中までしか表示されず一番下に「:」が表示されたら、キーボード↑↓でスクロール可能。表示やめてコマンドラインに戻るときはQキーを押す。

ファイルの改行コード変換させたくない場合はconfigの`core.autocrlf`をfalseにする。

さまざまなプログラム言語・ツールのための.gitignoreファイルのテンプレート<br>
https://github.com/github/gitignore

フォークとクローンの違い

- フォーク・・自分のGitHub上（＝リモートリポジトリ）にコピーされる。  
　　方法：GitHubのWeb上からForkボタンを操作。
- クローン・・自分のPC（＝ローカルリポジトリ）にコピーされる。  
　　方法：コマンド`git clone git@github.com:ReoTokeshi/ichiyasaGitSample.git`

他人のリポジトリを修正して提案したい（プルリク送りたい）ときは、  
<u>**フォーク → クローン → 修正 → プッシュ → プルリクエスト**</u>

