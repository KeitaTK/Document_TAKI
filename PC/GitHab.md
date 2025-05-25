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

## まとめ

- VSCodeで`git init`→`remote add`→`push`  
- Raspberry Piで`clone`→`pull`  
- SSHキー連携を使うとパスフレーズ入力が不要に  
- 各環境でGitの基本コマンドを覚えておくと効率化可能
