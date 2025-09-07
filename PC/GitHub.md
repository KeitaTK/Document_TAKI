---
marp: true
theme: default
paginate: true
class: lead
---

# VSCode → GitHub → Raspberry Pi  
Gitによる一連のアップロード／クローン・プルフロー

---

## アジェンダ

- 環境準備  
- VSCodeでのリポジトリ作成・Push  
- Raspberry PiでのClone  
- Raspberry PiでのPull  
- まとめ

---

## 環境準備

**1. GitHubアカウント**  
- リポジトリを作成できる状態に  

**2. VSCode側**  
- Git拡張機能を有効化  
- SSHキーの登録（推奨）  

**3. Raspberry Pi側**  
- Gitインストール
  
```bash
sudo apt update
sudo apt install git
```

---

## VSCodeでのリポジトリ作成・Push

1. **プロジェクトフォルダを開く**  
2. **Initialize Repository**  
3. ターミナルでリモート登録  


```bash
3. ターミナルでリモート登録  
git init
git add .
git commit -m "Initial commit"
git remote add origin git@github.com:ユーザー名/リポジトリ名.git
```

---

4. **Push**  

```bash
git push -u origin main
```

## Raspberry PiでのClone

1. VSCode側で作成したGitHubリポジトリのSSH/HTTPS URLをコピー  
2. Raspberry Piのターミナルで実行  

```bash
git clone git@github.com:ユーザー名/リポジトリ名.git
cd リポジトリ名
```

```bash
git pull origin main
```

---

## コンフリクト（競合）が発生した場合の対処法

### 例：GitHubのmainとローカルのmainで内容が異なり、push/pull時にエラーが出る場合

1. **エラー内容を確認**
	- 例: `error: failed to push some refs to ...` など
2. **リモートの変更をローカルに取り込む（マージ）**

```bash
git pull origin main
```

3. **コンフリクト箇所を修正**
	- ファイル内に`<<<<<<<`, `=======`, `>>>>>>>`の記号が現れるので、どちらの内容を残すか編集
	- 編集後、保存

4. **修正内容をコミット**

```bash
git add .
git commit -m "コンフリクト解消"
```

5. **再度push**

```bash
git push origin main
```

---

## まとめ

- VSCodeで`git init`→`remote add`→`push`  
- Raspberry Piで`clone`→`pull`  
- **コンフリクト時は`git pull`でマージし、競合箇所を修正してからpush**  
- SSHキー連携を使うとパスフレーズ入力が不要に  
- 各環境でGitの基本コマンドを覚えておくと効率化可能
- テスト
- 

