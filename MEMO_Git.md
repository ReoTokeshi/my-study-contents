
ルートディレクトリ `C:\Program Files\Git`<br>
　⇒GitBash起動時のカレント<br>
ホームディレクトリ `C:\Users\81808`

デフォルトブランチ名 `main`<br>
　⇒git configのinit.fefaultBranchで確認＆設定

core.editorの値を`"code --wait"`に設定で、コミット時にVSCode起動。

| コマンド | 内容 |
| --- | --- |
| cd ~ | ホームディレクトリへ移動 |
| git config --list | .gitconfig内の各設定値の確認 |
| git config --global _<設定項目>_ _<設定値>_ | 項目の設定 |
|  |  |
| git init | カレントでローカルリポジトリ作成 |
| git commit | コミット。VSCodeでコメント編集 |
| git status | ワークツリー内の状態確認 |
| git diff | 差分確認（ワークツリー⇔ステージングエリア） |
| git diff --cached | 差分確認（ステージングエリア⇔Gitディレクトリ） |
|  |  |

実行結果が途中までしか表示されず一番下に「:」が表示されたら、キーボード↑↓でスクロール可能。表示やめてコマンドラインに戻るときはQキーを押す。