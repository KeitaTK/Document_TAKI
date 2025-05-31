# ArduPilotでMotiveから位置情報を取得

ArduPilotにカスタムMAVLinkメッセージを追加し、Motiveからの位置情報を受信を実装する手順

## **手順概要**

### **1. XMLメッセージ定義**

まず、メッセージを定義します。

**メッセージ (ID: 2201)**
/home/memoto/UMEMOTO2/modules/mavlink/message_definitions/v1.0/ardupilotmega.xml
```xml
<message id="2201" name="TAKI_POS_Motive">
    <description>External coordinate input</description>
    <field type="float" name="x_coord">X coordinate</field>
    <field type="float" name="y_coord">Y coordinate</field>
    <field type="float" name="z_coord">Z coordinate</field>
</message>
```
定義したら一旦コンパイル

```bash
# ビルドクリーン
./waf clean
# ビルド設定
./waf configure --board Pixhawk6C
# ビルド実行
./waf copter
```

### **2. 受信処理実装**


受信したxyz座標データを姿勢制御器で使用するための実装方法は以下の通りです。

## **内部変数の定義場所**

**ArduCopter/Copter.h** に座標変数を追加：
355行目
```cpp
// Custom external coordinates
struct {
    Vector3f position;  // x, y, z coordinates
    bool valid;         // データの有効性フラグ
    uint32_t timestamp; // 最後の更新時刻
} external_coords;
```

## **受信ハンドラーの実装**

**GCS_Mavlink.cpp** の `handle_message` 関数内で座標データを格納：
```cpp
case MAVLINK_MSG_ID_TAKI_CUSTOME1_REQUEST:
{
    // メッセージをデコード
    mavlink_taki_custome1_request_t packet;
    mavlink_msg_taki_custome1_request_decode(&msg, &packet);
    
    // Copterクラスのインスタンスにアクセス
    Copter &copter = AP::copter();
    
    // 受信したxyz座標を内部変数に格納
    copter.external_coords.position.x = packet.x_coord;
    copter.external_coords.position.y = packet.y_coord; 
    copter.external_coords.position.z = packet.z_coord;
    copter.external_coords.valid = true;
    copter.external_coords.timestamp = AP_HAL::millis();
    
    break;
}
```

## **XMLメッセージ定義**

座標データ用のメッセージ定義：

```xml

External coordinate input
X coordinate
Y coordinate
Z coordinate

```

## **姿勢制御器での使用**

**control_modes.cpp** 等の制御ループで座標データを使用：

```cpp
// データの有効性確認（5秒以内の新しいデータか）
if (external_coords.valid && 
    (AP_HAL::millis() - external_coords.timestamp) set_pos_target(target_pos);
}
```

この実装により、外部から送信されたxyz座標が内部変数に格納され、姿勢制御器から直接アクセス可能になります[1][4]。データの有効性チェックとタイムスタンプによる古いデータの判定も含まれており、安全な制御が可能です。

Citations:
[1] https://discuss.ardupilot.org/t/send-and-receive-custom-message-over-mavlink/116148
[2] https://discuss.ardupilot.org/t/how-to-create-your-own-custom-messages-in-mavlink/26818
[3] https://discuss.ardupilot.org/t/received-custom-mavlink-message-on-gcs/120241
[4] https://discuss.ardupilot.org/t/add-custom-msg-using-mavlink-protocol/119235
[5] https://gazebosim.org/api/gazebo/3/ardupilot.html
[6] https://www.semanticscholar.org/paper/dd0f7a79e4bdba375f9a4a9eddb16616ac6ced21
[7] https://www.semanticscholar.org/paper/7181b34bfe497a99ccba879442dd44a441871765
[8] https://mavlink.io/en/messages/common.html
[9] https://mavlink.io/en/services/parameter.html
[10] https://ardupilot.org/dev/docs/code-overview-adding-support-for-a-new-mavlink-gimbal.html
[11] https://mavlink.io/en/messages/ardupilotmega.html
[12] https://ardupilot.org/copter/docs/logmessages.html
[13] https://discuss.ardupilot.org/t/controlling-attitude-with-mavlink/27739
[14] https://stackoverflow.com/questions/37383904/changing-mavlink-message-rate-ardupilotmega
[15] https://ardupilot.org/dev/docs/code-overview-adding-a-new-mavlink-message.html
[16] https://discuss.ardupilot.org/t/new-message-and-variables-in-apm-planner-2/11641
[17] https://discuss.px4.io/t/external-navigation-for-ardupilot/33367
[18] https://github.com/mavlink/qgroundcontrol/issues/2731
[19] https://ardupilot.org/dev/docs/mavlink-get-set-params.html

---
Perplexity の Eliot より: https://www.perplexity.ai/search/ardupilotkasutamumavlinkmetuse-Co_F0DsZT4mrhu9h8VifGA?utm_source=copy_output