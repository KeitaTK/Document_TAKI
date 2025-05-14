# WSL2 Ubuntuに学内プロキシ (ufproxy.b.cii.u-fukui.ac.jp:8080) を設定する手順

## 1. シェル全体にプロキシ設定する

Ubuntuの**環境変数**として、すべてのコマンドにプロキシを適用します。

### 手順

```bash
nano ~/.bashrc
```

末尾に次を追記：
```
function proxy_on() {
  export http_proxy="http://ufproxy.b.cii.u-fukui.ac.jp:8080/"
  export https_proxy="http://ufproxy.b.cii.u-fukui.ac.jp:8080/"
  export ftp_proxy="http://ufproxy.b.cii.u-fukui.ac.jp:8080/"
  export no_proxy="localhost,127.0.0.1,::1"

  # Git
  git config --global http.proxy $http_proxy
  git config --global https.proxy $https_proxy

  # pip
  mkdir -p ~/.config/pip
  cat > ~/.config/pip/pip.conf <<EOF
[global]
proxy = $http_proxy
EOF

  # apt
  sudo bash -c "cat > /etc/apt/apt.conf.d/95proxies <<EOF
Acquire::http::Proxy \"$http_proxy\";
Acquire::https::Proxy \"$https_proxy\";
EOF"

  # wget
  cat > ~/.wgetrc <<EOF
use_proxy = on
http_proxy = $http_proxy
https_proxy = $https_proxy
EOF

  # curl
  echo "proxy = \"$http_proxy\"" > ~/.curlrc

  # conda（使っていれば）
  cat > ~/.condarc <<EOF
proxy_servers:
  http: $http_proxy
  https: $https_proxy
EOF

  echo "[proxy_on] プロキシを有効にしました。"
}

function proxy_off() {
  unset http_proxy https_proxy ftp_proxy no_proxy

  # Git
  git config --global --unset http.proxy
  git config --global --unset https.proxy

  # pip
  rm -f ~/.config/pip/pip.conf

  # apt
  sudo rm -f /etc/apt/apt.conf.d/95proxies

  # wget
  rm -f ~/.wgetrc

  # curl
  rm -f ~/.curlrc

  # conda
  rm -f ~/.condarc

  echo "[proxy_off] プロキシを無効にしました。"
}


```

追記後、すぐ反映させるには：

```bash
source ~/.bashrc
```
使い方
```
source ~/.bashrc    # 反映（初回だけ）
proxy_on            # プロキシON
proxy_off           # プロキシOFF
```

# （補說）認証付きプロキシの場合

もしプロキシがユーザ名・パスワードで認証を要求する場合、URLの形式を次のようにします。

```
http://<username>:<password>@ufproxy.b.cii.u-fukui.ac.jp:8080/
```

※ パスワードに `@` や `:` など特殊文字が含まれる場合はエスケープ（URLエンコード）が必要です。

# 🚀 これでできること

- `sudo apt update`が通る
- `pip install`が通る
- `curl`や`wget`もそのまま使える
- 毎回手動でプロキシ設定しなくてよい

