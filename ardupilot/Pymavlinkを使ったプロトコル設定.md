# pymavlinkを使ったArdupilotのプロトコル設定

- pymavlinkを使ってRaspberry Pi 5からPichawk 6CにMavlinkメッセージを送信できる。これを使ってESCの制御信号を設定できる
- 今回はDshot300に設定する
  - まずはDShot600で試してみる（多くの機体で標準的に使用されている）
  - モーターの動作に問題がある場合はDShot300に下げる
  - 大型機体や長い配線を使用している場合は初めからDShot150を選択する
  
- 設定のPythonコードは　Mavlink_raspi\setup_Dshot300.py

    1. I/O PWM OUT (MAINとも呼ばれる)：
        - 主にモーター信号とGND接続用に設計されています
        - FMUが故障した場合でも、RC入力をバックアップとして通過させる機能があります
        - PX4ファームウェアではMAIN出力としてマッピングされます
    2. FMU PWM OUT (AUXとも呼ばれる)：
        - 主にサーボ信号、電源正極、GND接続用に設計されています
        - DshotやOne-shotなどの高速デジタルプロトコルをサポートしています
        - レーサードローンなど高速応答が必要な用途に適しています
        - PX4ファームウェアではAUX出力としてマッピングされます


  - 選択する'SERVO_BLH_MASK':は接続するポートを指定する
  
### 実行手順
1. コードをラズパイor PCで実行する。
2. BLHeli_S ESCの場合は「BLHeliSuite」をPCにインストール
3. ESC設定ソフトウェアを起動
4. 接続インターフェイスとして「BLHeli32/AM32 Bootloader (Betaflight/Cleanflight)」を選択
5. COMポートを選択して「Connect」→「Read Setup」をクリック

