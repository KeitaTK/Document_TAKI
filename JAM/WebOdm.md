## GPUを使うときの起動コマンド

- Windowsのgit bashdで起動する。
```bash
cd ~/WebODM
./webodm.sh --gpu start
```

```URL
http://localhost:8000/
```


## CUPでローカルネットワークからアクセスできるようにする

- Windowsのgit bashdで起動する。
```bash
./webodm.sh start --hostname 0.0.0.0 --port 8014
```

```URL
http://<your-ip-address>:8014/
```
- ルーターのポートフォワーディングで、外部からアクセスできるようにする。
- Windowsのファイアウォールで、ポート8014を開ける。 この設定をしないと、ローカルネットワークからアクセスできない。

私の環境でアクセスするときは、以下のURLになる。
```URL
http://192.168.11.14:8014/
``` 
