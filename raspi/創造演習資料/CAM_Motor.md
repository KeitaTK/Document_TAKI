# カメラとモーターの使い方（Raspberry Pi & Dynamixel）

---

## 🔧 サーボモーター（Dynamixel XM430-W350R）

### 仕様・関連リンク

- **型番**: Dynamixel XM430-W350R
- [CADデータ（ROBOTIS公式）](https://en.robotis.com/service/downloadpage.php?ca_id=70)
- [設定・動作確認ソフト DYNAMIXEL Wizard 2.0](https://en.robotis.com/service/downloadpage.php?ca_id=10)
- [Dynamixel SDK（Pythonなど対応）](https://emanual.robotis.com/docs/en/software/dynamixel/dynamixel_sdk/overview/)
- [e-Manual（XM430-W350Rの解説）](https://emanual.robotis.com/docs/en/dxl/x/xm430-w350/)

---

### 1. Windowsでの基本操作手順

1. 配布されたサンプルプログラム内の内容をすべてダウウンロードし同じフォルダ内に入れる

2. モーターをPCに接続（以下のような接続）
<img src="motor1.png" alt="接続方法" width="40%">

3. デバイスマネージャーからCOMポート番号を確認  
   または [Dynamixel Wizard 2.0](https://emanual.robotis.com/docs/en/software/dynamixel/dynamixel_wizard2/) で認識確認。

4. `sample_position.py` の **7行目** を編集し、確認したCOMポートに合わせる：

```python
DEVICENAME = 'COM3'  # 例
```

5. プログラムを実行し、キーボードの左右キーでサーボを動かす。

6. エラー発生時：

   * エラー内容を確認し、**ChatGPTやTAに相談**
   * FTDIドライバーが必要な場合もある（[公式FTDIサイト](https://ftdichip.com/drivers/)）

---

## 📷 カメラ（Raspberry Pi Camera Module v2）

### 製品情報

* **製品名**: Raspberry Pi Camera Module v2
* [商品ページ（KSY）](https://raspberry-pi.ksyic.com/main/index/pdp.id/144/pdp.open/144)

---

### 1. 接続方法

* カメラとRaspberry Pi を**専用配布のフラットケーブル**で接続（標準ケーブルは使用不可）
* Raspberry Pi本体の\*\*カメラコネクタ（2ヶ所のいずれか）\*\*に接続可
* 💡 **画像参照（省略）**

---

### 2. 動作確認

ディスプレイを接続した状態で以下を実行：

```bash
libcamera-hello --list-cameras
# または
rpicam-hello --list-cameras 0
```

* 正常にビューが表示されれば接続成功

---

### 3. パッケージインストール（仮想環境を使う前に）

```bash
sudo apt update
sudo apt full-upgrade
sudo apt install -y python3-picamera2
sudo apt install python3-opencv
```

---

### 4. 仮想環境を有効化してカメラ使用

```bash
source default_venv/bin/activate
```

#### カメラ動作確認（写真保存）

* `CAM1.py` を実行し、写真が保存されていれば成功
* 書き込みエラーが出る場合は：

  * 実行ディレクトリの**書き込み権限**を確認
  * 必要なら仮想環境を再作成

---

### 5. ARマーカー読み取り（QRコード等）

```bash
pip install pyserial
```

その後、`ReadQR1.py` を実行すると、**ARマーカーを読み取る処理**が動作する。

---

## ✅ 補足・注意点

* 仮想環境は基本的に作成して使用するのが推奨（環境の汚染防止）
* カメラが認識されない場合は以下を確認：

  * `sudo raspi-config` → `Interface Options` → `Camera` が **有効** になっているか
  * OSが `Raspberry Pi OS (64bit)` であるか
* モーターやカメラに関する質問は、**ログとエラー文を含めて聞く**とスムーズです
