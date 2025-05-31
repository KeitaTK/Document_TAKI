## **全体構成とデータフロー**

```
Windows PC (Motive) → Python SDK → UDP → Raspberry Pi → pymavlink → Pixhawk6C
```

## **推奨MAVLinkメッセージ**



### **最推奨: ATT_POS_MOCAP**

```
<!-- 引数構造 -->
time_usec    : uint64_t     (1個)
q           : float[4]      (配列として1個) - (w, x, y, z)順序
x           : float         (1個)
y           : float         (1個)  
z           : float         (1個)
covariance  : float[21]     (オプション)

```

## **座標系変換**

### **Motive → NED変換**

検索結果[5]のNatNet SDKデータを基に：

```python
# Windows PC側 (Python SDK)
def motive_to_ned(motive_data):
    # Motiveのデフォルト座標系からNEDへ変換
    # Motive: X=Right, Y=Up, Z=Forward
    # NED: X=North, Y=East, Z=Down
    
    ned_x = motive_data.z      # Forward → North
    ned_y = motive_data.x      # Right → East  
    ned_z = -motive_data.y     # Up → Down（反転）
    
    # クオータニオンも対応する変換
    ned_qw = motive_data.qw
    ned_qx = motive_data.qz    # Forward → North
    ned_qy = motive_data.qx    # Right → East
    ned_qz = -motive_data.qy   # Up → Down（反転）
    
    return ned_x, ned_y, ned_z, ned_qw, ned_qx, ned_qy, ned_qz
```

## **実装コード**

### **Windows PC側 (NatNet SDK + UDP送信)**

実際の送信プログラムは  https://github.com/KeitaTK/Motive_SDK.git   の  NatNet_5F50_UDP を使用する

### **Raspberry Pi側 (UDP受信 + MAVLink送信)**

このプログラムは    https://github.com/KeitaTK/Mavlink_raspi.git    の  MocaptoPixhawk.py   を使用する

## **ArduPilotパラメータ設定**

推奨設定：

```py
    # 基本EKF設定
    'AHRS_EKF_TYPE': 3,        # EKF3を使用
    'EK2_ENABLE': 0,           # EKF2を無効
    'EK3_ENABLE': 1,           # EKF3を有効
    
    # 外部位置データ用設定（モーションキャプチャ）
    'EK3_SRC1_POSXY': 6,       # ExternalNav（水平位置）
    'EK3_SRC1_POSZ': 6,        # ExternalNav（垂直位置）
    'EK3_SRC1_VELXY': 0,       # None（速度は位置から推定）
    'EK3_SRC1_VELZ': 0,        # None
    'EK3_SRC1_YAW': 6,         # ExternalNav（ヨー角）
    
    # イノベーションゲート調整
    'EK3_POS_I_GATE': 3,       # 位置イノベーションゲート
    'EK3_VEL_I_GATE': 5,       # 速度イノベーションゲート
    'EK3_HGT_I_GATE': 3,       # 高度イノベーションゲート
```

## **送信レート**

検索結果[1][2]から：
- **ArduPilot**: 4Hz以上
- 今回は20ヘルツあたりで送信

この構成により、Motiveのデータを適切にNED座標系に変換してArduPilotのEKFに送信できます。
