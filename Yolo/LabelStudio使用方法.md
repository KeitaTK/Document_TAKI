# PowerShellでのDockerを使ったLabel Studio：インストールから編集・再起動まで

**1. Dockerイメージの取得**
管理者権限のPowerShellを開き、以下を実行：

```powershell
docker pull heartexlabs/label-studio:latest
```

**2. コンテナ起動＆データ永続化**
プロジェクトディレクトリに移動し、データフォルダをマウントして起動：

```powershell
docker run -d `
  --name label-studio `
  -p 8080:8080 `
  -e DATA_UPLOAD_MAX_MEMORY_SIZE=524288000 `
  -e FILE_UPLOAD_MAX_MEMORY_SIZE=524288000 `
  -e DATA_UPLOAD_MAX_NUMBER_FILES=2000 `
  -v ${PWD}\label-studio-data:/label-studio/data `
  heartexlabs/label-studio:latest
```

**3. 初回セットアップ＆アノテーション**

1. ブラウザ(chromeだとエラーになった)で http://localhost:8080 にアクセス
2. 管理者アカウントを作成
3. 「Create Project」→プロジェクト名／説明を設定→データをインポート
4. 「Label」画面で矩形ツールを選択→画像上でドラッグ→「Submit」をクリック

**4. 作業終了・コンテナ停止**

```powershell
docker stop label-studio
```

**5. 再起動（次回以降）**

```powershell
docker start label-studio
```

再起動後、再び http://localhost:8080 へアクセスして続きのアノテーションを行えます。

