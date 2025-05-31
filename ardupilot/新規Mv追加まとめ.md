# ArduPilotカスタムMAVLinkメッセージ追加手順

ArduPilotにカスタムMAVLinkメッセージを追加し、送受信を実装する手順を以下にまとめます。

## **手順概要**

### **1. XMLメッセージ定義**

まず、リクエスト用とレスポンス用の2つのメッセージを定義します。

**リクエストメッセージ (ID: 189)**

```xml
<message id="189" name="TAKI_CUSTOME1_REQUEST">
  <description>Custom request message</description>
</message>
```

**レスポンスメッセージ (ID: 190)**

```xml
<message id="190" name="TAKI_CUSTOME1">
  <description>Simple custom message for testing</description>
  <field type="uint32_t" name="test_counter">Test counter value</field>
  <field type="uint16_t" name="taki_system_id">System ID</field>
  <field type="uint16_t" name="taki_component_id">Component ID</field>
</message>
```


### **2. ビルド設定追加**

`GCS_config.h`にコンパイル設定を追加：

```cpp
// enable custom TAKI_CUSTOME1_REQUEST handling
#define AP_MAVLINK_TAKI_CUSTOME_REQUEST_ENABLED 1
```


### **3. メッセージ登録**

`ap_message.h`の最後にメッセージを追加：

```cpp
#if AP_MAVLINK_TAKI_CUSTOME_REQUEST_ENABLED
    MSG_TAKI_CUSTOME1,
#endif    
    MSG_LAST // MSG_LAST must be the last entry in this enum
```


### **4. ハンドラー宣言**

`GCS.h`にハンドラー関数を宣言：

```cpp
#if AP_MAVLINK_TAKI_CUSTOME_REQUEST_ENABLED
void handle_send_taki_custome1(const mavlink_message_t &msg);
void send_taki_custome1() const;
#endif
```


### **5. 受信処理実装**

`GCS_Common.cpp`の受信処理部分に追加：

```cpp
#if AP_MAVLINK_TAKI_CUSTOME_REQUEST_ENABLED
    case MAVLINK_MSG_ID_TAKI_CUSTOME1_REQUEST:
        handle_send_taki_custome1(msg);
        break;
#endif
```


### **6. 送信処理登録**

`GCS_Common.cpp`の送信処理部分に追加：

```cpp
#if AP_MAVLINK_TAKI_CUSTOME_REQUEST_ENABLED
    case MSG_TAKI_CUSTOME1:
        CHECK_PAYLOAD_SIZE(TAKI_CUSTOME1);
        send_taki_custome1();
        break;
#endif
```


### **7. ハンドラー実装**

`GCS_Common.cpp`にハンドラー関数を実装：

**受信ハンドラー**

```cpp
#if AP_MAVLINK_TAKI_CUSTOME_REQUEST_ENABLED
void GCS_MAVLINK::handle_send_taki_custome1(const mavlink_message_t &msg)
{
    send_message(MSG_TAKI_CUSTOME1);
}
#endif
```

**送信関数**

```cpp
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


## **メッセージ処理フロー**

```mermaid
graph TD
    A[External GCS] -->|TAKI_CUSTOME1_REQUEST| B[MAVLink受信]
    B --> C["GCS_Common.cpp
    メッセージ分岐処理"]
    C --> D{メッセージID判定}
    D -->|ID: 189| E[handle_send_taki_custome1]
    E --> F["send_message
    (MSG_TAKI_CUSTOME1)"]
    F --> G[送信キューに追加]
    G --> H["GCS_Common.cpp
    送信処理分岐"]
    H --> I[send_taki_custome1]
    I --> J[mavlink_msg_taki_custome1_send]
    J --> K[MAVLink送信]
    K -->|TAKI_CUSTOME1| L[External GCS]
    
    %% 手順対応の注釈
    M1["手順1: XMLメッセージ定義
    (ID: 189, 190)"] -.-> A
    M5["手順5: 受信処理実装
    メッセージ分岐"] -.-> C
    M7a["手順7: ハンドラー実装
    受信ハンドラー"] -.-> E
    M7b["手順7: ハンドラー実装
    send_message呼び出し"] -.-> F
    M3["手順3: メッセージ登録
    ap_message.h"] -.-> G
    M6["手順6: 送信処理登録
    送信分岐処理"] -.-> H
    M7c["手順7: ハンドラー実装
    送信関数"] -.-> I
    M7d["手順7: ハンドラー実装
    MAVLink送信API"] -.-> J
    
    style A fill:#e1f5fe
    style L fill:#e8f5e8
    style E fill:#fff3e0
    style I fill:#fff3e0
    style M1 fill:#f0f0f0
    style M3 fill:#f0f0f0
    style M5 fill:#f0f0f0
    style M6 fill:#f0f0f0
    style M7a fill:#f0f0f0
    style M7b fill:#f0f0f0
    style M7c fill:#f0f0f0
    style M7d fill:#f0f0f0

```


## **RTOS/MAVLink処理統合図**

```mermaid
sequenceDiagram
    participant GCS as 外部GCS
    participant MAV as MAVLink Layer
    participant RTOS as RTOS Scheduler
    participant HANDLER as Message Handler
    participant QUEUE as Send Queue
    participant TX as Transmit Task

    GCS->>MAV: TAKI_CUSTOME1_REQUEST
    MAV->>RTOS: 受信割り込み
    RTOS->>HANDLER: handle_send_taki_custome1()
    HANDLER->>QUEUE: send_message(MSG_TAKI_CUSTOME1)
    QUEUE-->>RTOS: キューイング完了
    
    Note over RTOS,TX: RTOSスケジューラによる<br/>タスク切り替え
    
    RTOS->>TX: 送信タスク実行
    TX->>QUEUE: 送信メッセージ取得
    QUEUE->>TX: MSG_TAKI_CUSTOME1
    TX->>HANDLER: send_taki_custome1()
    HANDLER->>MAV: mavlink_msg_taki_custome1_send()
    MAV->>GCS: TAKI_CUSTOME1
```


## **システム内部アーキテクチャ**

```mermaid
graph LR
    subgraph "ArduPilot Core"
        A[MAVLink Parser] --> B[Message Router]
        B --> C[Handler Functions]
        C --> D[Send Queue]
        D --> E[Transmit Scheduler]
        E --> F[MAVLink Encoder]
    end
    
    subgraph "RTOS Layer"
        G[Task Scheduler]
        H[Interrupt Handler]
        I[Message Queue]
    end
    
    subgraph "Hardware Layer"
        J[UART/USB Interface]
    end
    
    A -.->|割り込み| H
    C -.->|キューイング| I
    G -.->|タスク切り替え| E
    F --> J
    
    style A fill:#ffebee
    style F fill:#e8f5e8
    style G fill:#f3e5f5
```

この実装により、外部GCSからのリクエストメッセージに対して、ArduPilot内部でカウンターを含むカスタムレスポンスメッセージを返送する仕組みが構築されます。処理はRTOSのタスクスケジューラによって管理され、MAVLinkの送受信キューを通じて非同期に実行されます。

