# 福井大学文京キャンパス用プロキシ設定ガイド

福井大学のWSL2 Ubuntu環境でArduPilot開発やその他の作業を行う際の、文京キャンパス専用プロキシ設定手順です。

***

## プロキシサーバー情報（文京キャンパス）

プロキシサーバー: ufproxy.b.cii.u-fukui.ac.jp  
ポート: 8080  

除外設定: 学内サーバー（`.u-fukui.ac.jp`ドメイン）とローカルアドレス（`localhost`, `127.0.0.1`, `::1`）は直接接続します。

***

## ~/.bashrcへの関数追加手順

1. ターミナルで以下を実行し、`~/.bashrc`を開きます。

   ```bash
   nano ~/.bashrc
   ```

2. ファイル末尾に以下の内容を追加します。

   ```bash
   # === 福井大学文京キャンパス プロキシ設定関数 ===

   function proxy_on() {
     # 文京キャンパス公式プロキシ設定
     export HTTP_PROXY="http://ufproxy.b.cii.u-fukui.ac.jp:8080/"
     export HTTPS_PROXY="http://ufproxy.b.cii.u-fukui.ac.jp:8080/"
     export FTP_PROXY="http://ufproxy.b.cii.u-fukui.ac.jp:8080/"
     export http_proxy="http://ufproxy.b.cii.u-fukui.ac.jp:8080/"
     export https_proxy="http://ufproxy.b.cii.u-fukui.ac.jp:8080/"
     export ftp_proxy="http://ufproxy.b.cii.u-fukui.ac.jp:8080/"
     export no_proxy=".u-fukui.ac.jp,localhost,127.0.0.1,::1"

     # Gitのプロキシ設定
     git config --global http.proxy "$http_proxy" 2>/dev/null
     git config --global https.proxy "$https_proxy" 2>/dev/null

     # pipのプロキシ設定
     mkdir -p ~/.config/pip
     cat > ~/.config/pip/pip.conf  /etc/apt/apt.conf.d/95proxies  ~/.wgetrc  ~/.curlrc  ~/.condarc /dev/null
     npm config set https-proxy "$https_proxy" 2>/dev/null

     echo "✅ [proxy_on] 文京キャンパスプロキシを有効化しました"
     echo "   プロキシ: ufproxy.b.cii.u-fukui.ac.jp:8080"
   }

   function proxy_off() {
     unset HTTP_PROXY HTTPS_PROXY FTP_PROXY
     unset http_proxy https_proxy ftp_proxy no_proxy

     git config --global --unset http.proxy 2>/dev/null
     git config --global --unset https.proxy 2>/dev/null

     rm -f ~/.config/pip/pip.conf
     sudo rm -f /etc/apt/apt.conf.d/95proxies
     rm -f ~/.wgetrc ~/.curlrc ~/.condarc

     npm config delete proxy 2>/dev/null
     npm config delete https-proxy 2>/dev/null

     echo "🚫 [proxy_off] プロキシを無効化しました"
   }

   function proxy_status() {
     if [ -n "$http_proxy" ]; then
       echo "✅ プロキシが有効: $http_proxy"
       echo "除外設定: $no_proxy"
       echo ""
       echo "=== 設定状況 ==="
       echo "Git HTTP: $(git config --global --get http.proxy 2>/dev/null || echo '未設定')"
       echo "pip設定: $([ -f ~/.config/pip/pip.conf ] && echo '設定済み' || echo '未設定')"
       echo "apt設定: $([ -f /etc/apt/apt.conf.d/95proxies ] && echo '設定済み' || echo '未設定')"
     else
       echo "🚫 プロキシが無効"
     fi
   }
   ```

3. 保存して終了: `Ctrl+O` → Enter → `Ctrl+X`

4. 変更を反映:

   ```bash
   source ~/.bashrc
   ```

***

## 基本的な使用方法

- **プロキシ有効化（文京キャンパス）**  
  `proxy_on`
- **プロキシ無効化**  
  `proxy_off`
- **設定状況確認**  
  `proxy_status`

***

## トラブルシューティング

1. **プロキシ設定後も接続できない**  
   ```bash
   proxy_status
   curl -v --proxy http://ufproxy.b.cii.u-fukui.ac.jp:8080 http://www.google.com
   ```

2. **学内サーバーへの接続がプロキシ経由になる**  
   ```bash
   echo $no_proxy
   curl -I http://www.u-fukui.ac.jp
   ```

3. **apt update時にエラー**  
   ```bash
   cat /etc/apt/apt.conf.d/95proxies
   proxy_off; proxy_on; sudo apt update
   ```

4. **Git操作でタイムアウト**  
   ```bash
   git config --global --get http.proxy
   git config --global http.proxy "http://ufproxy.b.cii.u-fukui.ac.jp:8080"
   ```

***

このガイドに従うことで、福井大学文京キャンパス環境下でWSL2 Ubuntuを使用した開発作業がスムーズに行えます。問題があれば、`proxy_status`で設定状況を確認してください。
