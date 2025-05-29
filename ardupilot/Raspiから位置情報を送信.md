# ラズパイから位置情報を送信

## Mavlinkメッセージの "AUTOPILOT_VERSION_REQUEST" の構造解析

1. ardupilotmega.xmlのAUTOPILOT_VERSION_REQUESTをラズパイから送る

```xml
    <message id="183" name="AUTOPILOT_VERSION_REQUEST">
      <description>Request the autopilot version from the system/component.</description>
      <field type="uint8_t" name="target_system">System ID.</field>
      <field type="uint8_t" name="target_component">Component ID.</field>
    </message>
```

2. メッセージが受信されると、handle_send_autopilot_version(msg)　が実行されるようになる。
その実行がされる周期の制御や、内容を以下のGCS_Common.cppの二つの記述が担当している。

- 関数の呼び出し - 「いつ実行するか」を制御
まず、AUTOPILOT_VERSION_REQUESTが受信されると、このコマンドが実行され、GCS_MAVLINK::handle_send_autopilot_version(msg)を呼び出す。
```bash
4302行目
#if AP_MAVLINK_AUTOPILOT_VERSION_REQUEST_ENABLED
    case MAVLINK_MSG_ID_AUTOPILOT_VERSION_REQUEST:
        handle_send_autopilot_version(msg);
        break;
#endif
```

- 関数の定義 - 「何をするか」を実装
この場合は、MSG_AUTOPILOT_VERSIONを送る。
```bash
#if AP_MAVLINK_AUTOPILOT_VERSION_REQUEST_ENABLED
void GCS_MAVLINK::handle_send_autopilot_version(const mavlink_message_t &msg)
{
    send_message(MSG_AUTOPILOT_VERSION);
}
#endif
```

3. 実際のメッセージの生成
GCS_Common.cppの中で定義されている

```bash/*
  send AUTOPILOT_VERSION packet
 */
void GCS_MAVLINK::send_autopilot_version() const
{
    uint32_t flight_sw_version;
    uint32_t middleware_sw_version = 0;
#ifdef APJ_BOARD_ID
    uint32_t board_version { uint32_t(APJ_BOARD_ID) << 16 };
#else
    uint32_t board_version = 0;
#endif
    char flight_custom_version[MAVLINK_MSG_AUTOPILOT_VERSION_FIELD_FLIGHT_CUSTOM_VERSION_LEN]{};
    char middleware_custom_version[MAVLINK_MSG_AUTOPILOT_VERSION_FIELD_MIDDLEWARE_CUSTOM_VERSION_LEN]{};
    char os_custom_version[MAVLINK_MSG_AUTOPILOT_VERSION_FIELD_OS_CUSTOM_VERSION_LEN]{};
#ifdef HAL_USB_VENDOR_ID
    const uint16_t vendor_id { HAL_USB_VENDOR_ID };
    const uint16_t product_id { HAL_USB_PRODUCT_ID };
#else
    uint16_t vendor_id = 0;
    uint16_t product_id = 0;
#endif
    // uint64_t uid = 100;
    uint64_t uid_taki = 100;
    uint8_t  uid2[MAVLINK_MSG_AUTOPILOT_VERSION_FIELD_UID2_LEN] = {0};

    uint8_t uid_len = sizeof(uid2); // taken as reference and modified
                                    // by following call:
    hal.util->get_system_id_unformatted(uid2, uid_len);

    const AP_FWVersion &version = AP::fwversion();

    flight_sw_version = version.major << (8 * 3) | \
                        version.minor << (8 * 2) | \
                        version.patch << (8 * 1) | \
                        (uint32_t)(version.fw_type) << (8 * 0);

    if (version.fw_hash_str) {
        strncpy_noterm(flight_custom_version, version.fw_hash_str, ARRAY_SIZE(flight_custom_version));
    }

    if (version.middleware_hash_str) {
        strncpy_noterm(middleware_custom_version, version.middleware_hash_str, ARRAY_SIZE(middleware_custom_version));
    }

    if (version.os_hash_str) {
        strncpy_noterm(os_custom_version, version.os_hash_str, ARRAY_SIZE(os_custom_version));
    }

    mavlink_msg_autopilot_version_send(
        chan,
        capabilities(),
        flight_sw_version,
        middleware_sw_version,
        version.os_sw_version,
        board_version,
        (uint8_t *)flight_custom_version,
        (uint8_t *)middleware_custom_version,
        (uint8_t *)os_custom_version,
        vendor_id,
        product_id,
        uid_taki,
        uid2
    );
}
```

4. libraries/GCS_MAVLink/ap_message.h
の中に送信するメッセージを追加しないといけないかも

## コンパイル設定

1. GCS_config.hの中でコンパイルするか設定できる。1にすればコンパイル。0にすればコンパイルしない。

```bash
#ifndef AP_MAVLINK_AUTOPILOT_VERSION_REQUEST_ENABLED
#define AP_MAVLINK_AUTOPILOT_VERSION_REQUEST_ENABLED 1
#endif
```


# これを参考に外部からpixhawk6cに新規カスタムMavlinkを追加してみる

1. まだ使用していないIDで.xmlにメッセージid="189" name="TAKI_CUSTOME1_REQUEST"を定義
2. build/Pixhawk6C/libraries/GCS_MAVLink/include/mavlink/v2.0/ardupilotmega/（common）などにファイルができたか確認
3. 関数を登録する
   - 定期的に送信する場合には　ap_message.h
   - リクエストに対して送信する場合は書かなくてよい
4. GCS_Common.cppに受信した時に実行することを記述
```bash
#if AP_MAVLINK_AUTOPILOT_VERSION_REQUEST_ENABLED
    case MAVLINK_MSG_ID_TAKI_CUSTOME1_REQUEST:
        handle_send_autopilot_version(msg);
        break;
#endif
```
とりあえずここまでで、コンパイルして動くか確認。この場合TAKI_CUSTOME1_REQUESTを送るとautopilot_versionがかえって来るはず。
送信するコマンドはpymavlink_TAKI_CUSTOME_REQUEST.py
成功した。

6. ビルド設定を追加する。
   - GCS_config.hの中にコンパイルするか設定できる。1にすればコンパイル。0にすればコンパイルしない。
```bash
// enable custom TAKI_CUSTOME1_REQUEST handling
#define AP_MAVLINK_TAKI_CUSTOME_REQUEST_ENABLED 1
```
   - コンパイル設定を追加したので4の1行目を変更
```bash
#if AP_MAVLINK_TAKI_CUSTOME_REQUEST_ENABLED
    case MAVLINK_MSG_ID_TAKI_CUSTOME1_REQUEST:
        handle_send_autopilot_version(msg);
        break;
#endif
```

8.  メッセージのリクエストが来たら、それに対してメッセージを送るような処理に飛ばす設定を記述する。その前に処理を登録するハンドラを /home/memoto/UMEMOTO2/libraries/GCS_MAVLink/GCS.h に追加

```bash
#if AP_MAVLINK_TAKI_CUSTOME_REQUEST_ENABLED
void handle_send_taki_custome1(const mavlink_message_t &msg);
#endif
```
9. 4のハンドラを変更

```bash
#if AP_MAVLINK_TAKI_CUSTOME_REQUEST_ENABLED
    case MAVLINK_MSG_ID_TAKI_CUSTOME1_REQUEST:
        handle_send_taki_custome1(msg);
        break;
#endif
```
10. ハンドラの実装

```bash
#if AP_MAVLINK_TAKI_CUSTOME_REQUEST_ENABLED
void GCS_MAVLINK::handle_send_taki_custome1(const mavlink_message_t &msg)
{
    send_message(MSG_AUTOPILOT_VERSION);
}
#endif
```

ここまでは成功!!

11. カスタムメッセージの追加。1と同様に返信用のメッセージ　"id="190" name="TAKI_CUSTOME1"　を定義

```bash
<message id="190" name="TAKI_CUSTOME1">
  <description>Simple custom message for testing</description>
  <field type="uint32_t" name="test_counter">Test counter value</field>
  <field type="uint16_t" name="taki_system_id">System ID</field>
  <field type="uint16_t" name="taki_component_id">Component ID</field>
</message>
```

12. ardupilotから外部に送信するメッセージに  MSG_AUTOPILOT_VERSION が追加されるように /home/memoto/UMEMOTO2/libraries/GCS_MAVLink/ap_message.h の一番最後に追加

```bash
#if AP_MAVLINK_MSG_FLIGHT_INFORMATION_ENABLED
    MSG_FLIGHT_INFORMATION,
#endif
#if AP_MAVLINK_TAKI_CUSTOME_REQUEST_ENABLED
    MSG_TAKI_CUSTOME1,
#endif    
    MSG_LAST // MSG_LAST must be the last entry in this enum
};
```
これでコンパイルすれば、 send_message(MSG_TAKI_CUSTOME1) を呼んだときに GCS_MAVLink の送信キューに入るようになる

13.  /home/memoto/UMEMOTO2/libraries/GCS_MAVLink/GCS.h に　void send_taki_custome1()　の定義を追加
```bash
void send_taki_custome1() const;
```

14. /home/memoto/UMEMOTO2/libraries/GCS_MAVLink/GCS_Common.cpp に登録しておけば、メッセージを送る必要があるか確認して全体の送信メッセージに追加してくれる

```bash
    case MSG_AUTOPILOT_VERSION:
        CHECK_PAYLOAD_SIZE(AUTOPILOT_VERSION);
        send_autopilot_version();
        break;

// ← ここに追加
#if AP_MAVLINK_TAKI_CUSTOME_REQUEST_ENABLED
    case MSG_TAKI_CUSTOME1:
        CHECK_PAYLOAD_SIZE(TAKI_CUSTOME1);
        send_taki_custome1();
        break;
#endif
```
15. 実際に送信するメッセージを作る。　/home/memoto/UMEMOTO2/libraries/GCS_MAVLink/GCS_Common.cpp　に記述する

```bash

/*
  send TAKI_CUSTOME1 packet
 */
void GCS_MAVLINK::send_taki_custome1() const
{
    static uint32_t test_counter = 0;
    test_counter++;
    
    mavlink_msg_taki_custome1_send(
        chan,
        test_counter,                    // test_counter
        mavlink_system.sysid,           // taki_system_id
        mavlink_system.compid           // taki_component_id
    );
}

```

16. send_message()中身を変更

```bash
#if AP_MAVLINK_TAKI_CUSTOME_REQUEST_ENABLED
void GCS_MAVLINK::handle_send_taki_custome1(const mavlink_message_t &msg)
{
    send_message(MSG_TAKI_CUSTOME1);
}
#endif
``` 

17. コンパイルを行い、TAKI_CUSTOME1_REQUESTを送信すると、カウントアップでの返答できた。


# 上を基に、位置情報をラズパイから送信し、ardupilotで内部変数に格納する