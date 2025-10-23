# カスタムNodeODMイメージ（Node.js 18対応）構築ガイド
## ローカルネットワークアクセス対応版

## 概要

WebODMの処理ノード（NodeODM）が古いNode.js 14を使用しており、最新の構文（`||=`演算子）に対応していないエラーが発生していました。このガイドでは、以下を実現します：

1. **Node.js 18対応のカスタムNodeODMイメージを構築**
2. **ポート 8014 でのアクセスを永続化**
3. **ローカルネットワーク内の他のPCからアクセス可能**

---

## 目次

1. [問題の原因](#問題の原因)
2. [ディレクトリ構成](#ディレクトリ構成)
3. [ファイル内容](#ファイル内容)
4. [構築手順](#構築手順)
5. [ネットワークアクセス設定](#ネットワークアクセス設定)
6. [動作確認](#動作確認)
7. [トラブルシューティング](#トラブルシューティング)

---

## 問題の原因

### エラーメッセージ
```
SyntaxError: Unexpected token '||='
```

### 根本原因
- 公式NodeODMイメージが**Node.js 14**を使用している
- Node.js 14では`||=`（論理演算子の代入）構文がサポートされていない
- NodeODMが依存する`@so-ric/colorspace`パッケージがNode.js 18以上を必要とする

### 解決策
- **Node.js 18対応のカスタムNodeODMイメージを構築**
- 公式ODMイメージ（opendronemap/odm:latest）をベースに使用
- **docker-compose.override.yml でカスタムイメージと永続ポート設定を指定**

---

## ディレクトリ構成

```
~/ (ホームディレクトリ)
├── WebODM/                           # WebODMメインリポジトリ
│   ├── docker-compose.yml            # メインのCompose設定
│   ├── docker-compose.override.yml   # カスタムイメージ＆ポート設定（新規作成）
│   ├── nodeodm/                      # NodeODM設定フォルダ
│   ├── webodm.sh                     # WebODM起動スクリプト
│   └── その他...
│
└── custom-nodeodm/                   # カスタムNodeODMビルド用（新規作成）
    └── Dockerfile                    # Node.js 18対応Dockerfile（新規作成）
```

### 新規作成が必要なファイル

1. `~/custom-nodeodm/Dockerfile` - Node.js 18対応のDockerfile
2. `~/WebODM/docker-compose.override.yml` - カスタムイメージとポート設定

---

## ファイル内容

### 1. Dockerfile（`~/custom-nodeodm/Dockerfile`）

```dockerfile
# 公式ODMイメージをベース（処理エンジンとPython環境を含む）
FROM opendronemap/odm:latest

# メタデータ設定
LABEL maintainer="Custom NodeODM <custom@example.com>"

# ポート公開
EXPOSE 3000

# rootユーザーで実行
USER root

# Node.js 18インストール用ツールと依存パッケージを準備
# git: NodeODMリポジトリのクローンに必要
# curl: Node.jsリポジトリの追加に必要
# gpg-agent: apt-keyの管理に必要
RUN apt-get update && apt-get install -y curl gpg-agent git

# Node.js 18をNodeSourceリポジトリからインストール
# 従来のNode.js 14（setup_14.x）から18.x（setup_18.x）に変更
RUN curl --silent --location https://deb.nodesource.com/setup_18.x | bash -

# Node.js 18と関連パッケージをインストール
# nodemon: 開発時のファイル監視ツール
# unzip, p7zip-full: アーカイブ解凍ツール
RUN apt-get install -y nodejs unzip p7zip-full && npm install -g nodemon && \
    ln -s /code/SuperBuild/install/bin/untwine /usr/bin/untwine && \
    ln -s /code/SuperBuild/install/bin/entwine /usr/bin/entwine && \
    ln -s /code/SuperBuild/install/bin/pdal /usr/bin/pdal

# NodeODM用作業ディレクトリ作成
RUN mkdir /var/www

# 作業ディレクトリ設定
WORKDIR "/var/www"

# NodeODMソースコードをGitHubからクローン
# npm install --production: 本番環境用パッケージのみをインストール
# mkdir -p tmp: 一時ディレクトリ作成
RUN git clone https://github.com/OpenDroneMap/NodeODM.git . && \
    npm install --production && mkdir -p tmp

# コンテナ起動時のデフォルトコマンド
# Node.js 18でNodeODMのindex.jsを実行
ENTRYPOINT ["/usr/bin/node", "/var/www/index.js"]
```

#### Dockerfileの重要な変更点

| 項目 | 公式版（旧） | カスタム版（新） | 理由 |
|------|-----------|--------------|------|
| Node.jsバージョン | `setup_14.x` | `setup_18.x` | `\|\|=`構文対応 |
| ベースイメージ | opendronemap/odm:latest | 同じ | ODMと依存関係を継承 |
| git依存 | なし | あり（apt-get install追加） | リポジトリクローン用 |
| ソースコピー方法 | `COPY . /var/www` | `git clone` | ローカルコピーなし |
| 起動方法 | `ENTRYPOINT` | 同じ | 正確な起動方法 |

---

### 2. docker-compose.override.yml（`~/WebODM/docker-compose.override.yml`）

```yaml
version: '3'
services:
  node-odm:
    image: custom-nodeodm:latest
  
  webapp:
    ports:
      - "0.0.0.0:8014:8000"
```

#### 説明

- `docker-compose.override.yml`はdocker-composeによって自動で検出・マージされる設定ファイル
- `services.node-odm.image`の指定により、公式の`opendronemap/nodeodm`の代わりにカスタムイメージを使用
- `services.webapp.ports`でホストポート 8014 をコンテナポート 8000 にマッピング
- WebODMが起動時にこの設定を読み込み、適切なイメージから`webodm-node-odm-1`コンテナを作成

#### ポート設定の詳細

```yaml
ports:
  - "0.0.0.0:8014:8000"
```

- **0.0.0.0**: すべてのネットワークインターフェースでリッスン（ローカルホストとローカルネットワーク両対応）
- **8014**: ホスト側（外部）ポート
- **8000**: コンテナ内部ポート

**結果**：
- ローカルホスト: `http://localhost:8014`
- ローカルネットワーク: `http://<ホストIP>:8014`

---

## 構築手順

### ステップ1: 作業ディレクトリの準備

```bash
# custom-nodeodm ディレクトリを作成
mkdir -p ~/custom-nodeodm
cd ~/custom-nodeodm
```

### ステップ2: Dockerfileの作成

上記の「Dockerfile（`~/custom-nodeodm/Dockerfile`）」セクションの内容を以下のコマンドで作成：

```bash
cat > Dockerfile << 'EOF'
# 公式ODMイメージをベース
FROM opendronemap/odm:latest

LABEL maintainer="Custom NodeODM <custom@example.com>"

EXPOSE 3000

USER root

RUN apt-get update && apt-get install -y curl gpg-agent git

RUN curl --silent --location https://deb.nodesource.com/setup_18.x | bash -

RUN apt-get install -y nodejs unzip p7zip-full && npm install -g nodemon && \
    ln -s /code/SuperBuild/install/bin/untwine /usr/bin/untwine && \
    ln -s /code/SuperBuild/install/bin/entwine /usr/bin/entwine && \
    ln -s /code/SuperBuild/install/bin/pdal /usr/bin/pdal

RUN mkdir /var/www

WORKDIR "/var/www"

RUN git clone https://github.com/OpenDroneMap/NodeODM.git . && \
    npm install --production && mkdir -p tmp

ENTRYPOINT ["/usr/bin/node", "/var/www/index.js"]
EOF
```

### ステップ3: カスタムイメージのビルド

```bash
cd ~/custom-nodeodm
docker build --no-cache -t custom-nodeodm:latest .
```

**出力例：**
```
[+] Building 50.8s (11/11) FINISHED
```

**確認：**
```bash
docker images | grep custom-nodeodm
```

`custom-nodeodm   latest`が表示されることを確認

### ステップ4: docker-compose.override.yml の作成

```bash
cd ~/WebODM
cat > docker-compose.override.yml << 'EOF'
version: '3'
services:
  node-odm:
    image: custom-nodeodm:latest
  
  webapp:
    ports:
      - "0.0.0.0:8014:8000"
EOF
```

**確認：**
```bash
cat docker-compose.override.yml
```

**期待される出力：**
```yaml
version: '3'
services:
  node-odm:
    image: custom-nodeodm:latest
  
  webapp:
    ports:
      - "0.0.0.0:8014:8000"
```

### ステップ5: WebODMサービスの停止・クリーンアップ

```bash
cd ~/WebODM
docker-compose down
docker system prune -af  # （注意：すべてのイメージが削除される）
```

### ステップ6: カスタムイメージの再ビルド（クリーンアップ後）

```bash
cd ~/custom-nodeodm
docker build --no-cache -t custom-nodeodm:latest .
```

### ステップ7: WebODMサービスの起動

```bash
cd ~/WebODM
docker-compose up
```

**または**

```bash
./webodm.sh start
```

---

## ネットワークアクセス設定

### ローカルホストからのアクセス

**設定不要** - デフォルトで動作

```
http://localhost:8014
```

### ローカルネットワーク内の他のPCからのアクセス

#### 前提条件

- WebODMが動作するPCがローカルネットワークに接続
- 両PC が同じサブネット内

#### アクセス方法

1. **WebODMが動作するPCのIPアドレスを確認**

   **Windows:**
   ```bash
   ipconfig
   ```
   
   **Linux/Mac:**
   ```bash
   ifconfig
   ```
   
   例：`192.168.1.100`

2. **他のPCから以下にアクセス**

   ```
   http://192.168.1.100:8014
   ```
   
   ブラウザのアドレスバーに上記を入力

#### ファイアウォール設定（必要な場合）

**Windows ファイアウォールでポート 8014 を許可：**

```bash
netsh advfirewall firewall add rule name="WebODM 8014" dir=in action=allow protocol=tcp localport=8014
```

**Linux（UFW使用）:**

```bash
sudo ufw allow 8014/tcp
```

#### 静的IP設定（推奨）

IPが変わらないよう、ルーターでWebODMホストに固定IPを割り当てる：

1. ルーターの管理画面にアクセス
2. WebODMが動作するPCのMACアドレスを確認
3. 固定IP（例：192.168.1.100）を割り当て
4. 以後は常に `http://192.168.1.100:8014` でアクセス可能

---

## 動作確認

### 確認1: コンテナが正常起動しているか

別のターミナルで以下を実行：

```bash
docker ps
```

**期待される出力：**
```
CONTAINER ID   IMAGE                        COMMAND                   STATUS
...
80982de8fdf0   custom-nodeodm:latest        "/usr/bin/node /var/…"   Up 35 seconds   webodm-node-odm-1
693ffe5270b4   opendronemap/webodm_webapp   "/bin/bash -c 'servi…"   Up 35 seconds   0.0.0.0:8014->8000/tcp   webapp
...
```

**確認ポイント：**
- `custom-nodeodm:latest`がIMAGE列に表示されている
- `STATUS`が「Up」で、「Exited」ではない
- `webapp` の PORTS に `0.0.0.0:8014->8000/tcp` が表示

### 確認2: Node.jsバージョン確認

```bash
docker ps | grep node-odm
# コンテナIDを取得し、以下を実行
docker exec <コンテナID> node --version
```

**期待される出力：**
```
v18.x.x
```

### 確認3: NodeODMの稼働確認

```bash
curl http://localhost:3000/info
```

**期待される出力（JSON形式）：**
```json
{
  "version": "x.x.x",
  "taskQueueCount": 0,
  "runningTasksCount": 0,
  ...
}
```

### 確認4: WebODM管理画面でのアクセス

ブラウザで以下にアクセス：

**ローカルホスト:**
```
http://localhost:8014
```

**ローカルネットワーク（他のPC）:**
```
http://192.168.1.100:8014
```

- WebODM UIが正常表示される
- ログインできることを確認
- 「Settings」タブでNodeODMの接続状態を確認

---

## トラブルシューティング

### 問題1: `SyntaxError: Unexpected token '||='` が再度発生

**原因：** キャッシュが残っており、古いイメージが使用されている

**解決策：**
```bash
cd ~/custom-nodeodm
docker build --no-cache -t custom-nodeodm:latest .

cd ~/WebODM
docker-compose down
docker-compose up
```

### 問題2: `node-odm-1` コンテナが `Exited (1)` で停止

**原因：** 複数の可能性（ファイルパス、依存関係、設定エラー）

**確認方法：**
```bash
docker logs webodm-node-odm-1
```

**よくある原因と対処：**

| エラー内容 | 原因 | 対処 |
|----------|------|------|
| `FileNotFoundError: /code/opendm/config.py` | ODMエンジンが不完全 | Dockerfileのベースイメージを確認 |
| `git: not found` | git コマンド未インストール | Dockerfileに `apt-get install -y git` を追加 |
| `npm: command not found` | Node.js未インストール | Node.jsのインストール行を確認 |

### 問題3: docker-compose.override.yml が反映されない

**確認方法：**
```bash
docker ps | grep webapp
# PORTS列を確認、8014 が表示されているか
```

**解決策：**

1. ファイル位置確認
   ```bash
   ls -la ~/WebODM/docker-compose.override.yml
   ```

2. ファイル内容確認
   ```bash
   cat ~/WebODM/docker-compose.override.yml
   ```

3. WebODMディレクトリから起動
   ```bash
   cd ~/WebODM
   docker-compose down
   docker-compose up
   ```

### 問題4: ビルド時に `pull access denied for custom-nodeodm`

**原因：** `docker system prune -af` でカスタムイメージが削除された

**解決策：**
```bash
cd ~/custom-nodeodm
docker build --no-cache -t custom-nodeodm:latest .
```

### 問題5: ローカルネットワークの他のPCからアクセスできない

**確認手順：**

1. **IPアドレスが正しいか確認**
   ```bash
   ipconfig  # Windows
   ```

2. **ファイアウォール設定確認**
   ```bash
   netsh advfirewall firewall show rule name="WebODM 8014"
   ```

3. **接続テスト**
   ```bash
   ping 192.168.1.100
   ```

4. **ポート疎通確認**
   ```bash
   # 別PCから実行
   telnet 192.168.1.100 8014
   # または
   curl http://192.168.1.100:8014
   ```

---

## 参考情報

### 公式ドキュメント

- **NodeODM GitHub**: https://github.com/OpenDroneMap/NodeODM
- **WebODM GitHub**: https://github.com/OpenDroneMap/WebODM
- **OpenDroneMap**: https://www.opendronemap.org/

### Node.js バージョン選定

- **Node.js 14**: サポート終了（2023年4月）
- **Node.js 18**: LTS版（サポート期間：2026年4月）
- **Node.js 20**: LTS版（サポート期間：2026年10月）

最新セキュリティパッチとバグ修正を得るため、Node.js 18以上を推奨

### ディレクトリ構成の確認方法

```bash
# custom-nodeodm ディレクトリ
ls -la ~/custom-nodeodm/
# 出力例:
# Dockerfile

# WebODM ディレクトリ
ls -la ~/WebODM/ | grep override
# 出力例:
# docker-compose.override.yml
```

### ポート設定の比較

| アクセス元 | 設定 | URL例 |
|----------|------|------|
| ローカルホスト | docker-compose.override.ymlで設定 | `http://localhost:8014` |
| ローカルネットワーク | docker-compose.override.ymlで設定 + ファイアウォール許可 | `http://192.168.1.100:8014` |
| インターネット経由 | nginxリバースプロキシ等が別途必要 | N/A（別途設定） |

---

## 運用ガイド

### 日常的な起動・停止

**起動：**
```bash
cd ~/WebODM
./webodm.sh start
# または
docker-compose up -d
```

**停止：**
```bash
cd ~/WebODM
./webodm.sh stop
# または
docker-compose down
```

**ログ確認：**
```bash
docker-compose logs -f webapp
docker-compose logs -f webodm-node-odm-1
```

### 定期的なメンテナンス

**古いイメージの削除（3ヶ月ごと推奨）：**
```bash
docker image prune -a
```

**ディスク使用量確認：**
```bash
docker system df
```

---

## まとめ

このガイドに従うことで：

1. ✅ Node.js 18対応のカスタムNodeODMイメージを構築
2. ✅ ポート 8014 での永続アクセス設定
3. ✅ ローカルネットワーク内の他のPCからのアクセス実現
4. ✅ `||=`構文エラーを完全に解決
5. ✅ 最新のセキュリティパッチとバグ修正を適用

その結果、WebODMで写真処理を実行する際に：
- NodeODMの処理ノードが安定して稼働
- 複数PC からのアクセスが可能
- 毎回コマンドラインオプションを指定する必要がない

