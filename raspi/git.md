# gitでアップロード

```bash
git pull
git add .
git commit -m "commit message"
git push
```



## 手順：パスフレーズなしのSSH鍵を作成

```bash
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/id_ed25519_nopass
```

### 1. パスフレーズを空にする
現在、SSH鍵生成のプロンプトでパスフレーズの入力を求められています。パスフレーズなしで鍵を作成するには、以下のように進めます：

- **Enter passphrase (empty for no passphrase):** と表示されたら、**Enterキーを押す**（何も入力しない）。
- 次のプロンプト **Enter same passphrase again:** でもう一度 **Enterキーを押す**。

これでパスフレーズなしのSSH鍵が作成されます。

### 2. 公開鍵を確認してGitHubに登録
鍵が生成されたら、公開鍵を表示してGitHubに登録します：

```bash
# 公開鍵を表示
cat ~/.ssh/id_ed25519_nopass.pub
```

出力された内容（`ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI...` から始まる文字列）をコピーします。

次に、GitHubのウェブサイトにアクセスし、以下の手順で公開鍵を登録します：
1. GitHubにログイン
2. 右上のプロフィールアイコンをクリック → **Settings**
3. 左側のメニューから **SSH and GPG keys** を選択
4. **New SSH key** ボタンをクリック
5. **Title** に任意の名前（例：`Raspberry Pi No Passphrase`）を入力
6. **Key** フィールドにコピーした公開鍵を貼り付ける
7. **Add SSH key** をクリック

### 3. SSH設定を更新
SSH設定ファイルを作成または編集して、新しい鍵を使用するように設定します：

```bash
# SSH設定ファイルを作成/編集
nano ~/.ssh/config
```

以下の内容を追加または修正します：
```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_nopass
```

設定ファイルの権限を適切に設定します：
```bash
chmod 600 ~/.ssh/config
```

### 4. SSH接続をテスト
新しい鍵でGitHubへの接続をテストします：

```bash
ssh -T git@github.com
```

成功メッセージが表示されれば、設定は正しいです：
```
Hi KeitaTK! You've successfully authenticated, but GitHub does not provide shell access.
```
```bash
# 1. Gitユーザー情報を設定
git config --global user.email "keita17101042@gmail.com"
git config --global user.name "KeitaTK"

# 2. リモートURLをSSHに変更
git remote set-url origin git@github.com:KeitaTK/Mavlink_raspi.git

# 3. SSHエージェントにキーを追加（必要に応じて）
ssh-add ~/.ssh/id_ed25519_nopass

# 4. 再度コミット・プッシュを試行
git add .
git commit -m "commit message"
git push
```


### 5. git pullをテスト
最後に、`git pull` を実行してパスフレーズを求められずに動作するか確認します：

```bash
git pull
```

これでパスフレーズを求められずにリポジトリの更新ができるはずです。

## 注意点

- **セキュリティリスク**: パスフレーズなしのSSH鍵は便利ですが、鍵が盗まれた場合に不正アクセスされるリスクがあります。ラズパイが安全な環境にあることを確認してください。
- **既存の鍵との混在**: 既存の鍵（パスフレーズ付き）と新しい鍵（パスフレーズなし）を混在させる場合、`~/.ssh/config` で正しい鍵が使用されるように注意してください。

この手順で問題が解決するはずです。もしエラーが発生する場合は、エラーメッセージを共有していただければ、さらにサポートいたします。