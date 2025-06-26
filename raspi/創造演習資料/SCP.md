# WindowsPCからRaspberry PiへPythonコードを転送・実行する手順

### **1. フォルダ作成・Pythonコード作成**

1. エクスプローラー等でフォルダを作成する。
2. その中にVSCの左上ファイルから新しいテキストファイルを選択し、保存するときに拡張子を.pyにしてPythonファイル（例: `main.py`）を作成する。

📁 MyPythonProject(適当な名前でOK)
└── main.py

3. そのファイルにPythonコードを書き込む（例： `print("Hello, World!")`）

### **2. フォルダのパスをコピー**

- エクスプローラーを開き、作成したフォルダを右クリック。「パスのコピー」を選択する。

エクスプローラーでパスをコピー  
*（例：`C:\Users\yourname\Documents\MyPythonProject`）*

--- 

### **3.ターミナルでscpコマンドを使いフォルダをラズパイに転送**

- PowerShellやコマンドプロンプト、ターミナル等を開き、以下のコマンドを実行。

scp -r "C:\Users\yourname\Documents\MyPythonProject" raspi@192.168.11.50:/home/pi/

- `-r` はフォルダごと再帰的にコピーするオプション。
- `raspi` はラズパイのユーザー名、`192.168.11.50` はラズパイのIPアドレス（各自の環境に合わせて変更してください）。

---

### **4. SSHでラズパイに接続**

- PowerShellまたはターミナルで以下のコマンドを実行します。
ssh pi@192.168.11.50


- パスワードを入力してログインします。

---

### **5. 転送したフォルダに移動し、Pythonコードを実行**

- 以下のコマンドを実行します。
cd ~/MyPythonProject
python3 main.py

---

### **Tips**

- Python3がインストールされていることを確認してください（`python3 --version`）。
- ラズパイ側でSSHが有効になっている必要があります（`sudo raspi-config` → インターフェース → SSHを有効化）。
- それでも解決しないときはAIかTAに質問する。

---

### **参考**

- 直接ラズパイの中をVSCで編集する方法 
[VSCodeのRemote-SSH機能を使えば、転送せず直接編集も可能](https://craft-gogo.com/raspberry-pi-remote-ssh/)  
- [ラズパイへのSSH接続・Python実行方法](https://qiita.com/c60evaporator/items/26ab9cfb9cd36facc8fd)

---

これで、**WindowsPCのVSCで作成したPythonコードをラズパイに送り、SSHで実行する作業**がスムーズに行えます。