
## ArduPilotにおけるカスタムMAVLinkメッセージの追加とRaspberry Pi 5によるメッセージの受信

### 1. 全体の流れ
1. ardupilotのMavlinkmega.xmlまたはcommon.xmlファイルの中の既存のメッセージを編集し、送信型などを変更する。
2. この編集内容を新規ブランチなどを作成し、GitHabに保存する。
3. ラズパイにgit cloneしこれを用いてpymavlinkを生成する。


#### 1.1 ardupilotの編集

1. https://ardupilot.org/copter/docs/ArduCopter_MAVLink_Messages.html#arducopter-mavlink-messages
   
   このドキュメントの中で、使用しないコードを選択する。
2. そのコマンドをたどり、ardupilotから外部へ送信する方のコマンドを探す。
3. その.xmlファイルのデータ型の変更と、実際のメッセージ作成のコードを探し、変更する。
4. 編集内容をMavlinkの新規ブランチなどを作成し、GitHabに保存する。


#### 1.2 ラズパイのpymavlinkの生成

1. 先ほど生成したMavlinkの新規ブランチをラズパイにクローンする。
   
```bosh
git clone -b Mavlink_UMEMOTO https://github.com/KeitaTK/mavlink.git --recursive
```

2. 必要に応じて、以前のバージョンをアンインストールする。
```bash    
pip uninstall pymavlink
```
強制的にする場合は

```bosh
python3 -m pip uninstall pymavlink --break-system-packages
```
3. 仮想環境の中でインストールを行う
```bosh
source Mavlink_venv/bin/activate
```
抜けるときは
```bash 
deactivate
```

1. Pythonセットアップ プログラムを実行します。setup.pyの場所は随時確認すること。
```bash 
cd ~/mavlink/pymavlink 
python setup.py install
```
生成されたMAVLinkライブラリは、 pipを使用してインストールされたものと同じように使用できる。

受信用のプログラムは　
```bash    
git clone https://github.com/KeitaTK/Mavlink_raspi.git
```
で自分のローカル環境にクローンして使用。