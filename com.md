1. Purpose

本仕様は、PCソフトウェアとファームウェア間の通信プロトコルを定義する。

本プロトコルは以下を目的とする。

- PCとファームウェアを独立して更新可能とする
- 新旧バージョンの組み合わせでも動作可能とする
- コマンド仕様の長期的な拡張を容易にする
- 高速通信ではオーバーヘッドを最小限にする

---

2. Design Principles

本プロトコルでは、プロトコル全体のバージョン管理は行わない。

以下を設計原則とする。

- パケットフォーマットは変更しない
- Header構造は変更しない
- エンディアンはリトルエンディアン固定
- コマンド仕様は後方互換を維持しながら拡張する
- Capabilityによって相手機能を判定する

---

3. Packet Format

+------------+---------------+----------+
| Command ID | Payload Length| Payload  |
+------------+---------------+----------+

Field| Description
Command ID| コマンドID
Payload Length| Payloadサイズ(Byte)
Payload| コマンド固有データ

数値データはすべてリトルエンディアンとする。

文字列データはASCIIまたはUTF-8とし、エンディアン変換は行わない。

---

4. Payload Format

コマンドは以下の2種類のPayload形式を使用する。

4.1 Fixed Structure

高速通信を目的としたコマンド。

対象例

- Measurement
- Streaming
- Binary Data
- Image Transfer

特徴

- 固定構造体
- パースが高速
- PayloadLengthで後方互換を実現

例

Version1

typedef struct
{
    uint32_t channel;
    uint32_t count;
} MeasureRequest;

Version2

typedef struct
{
    uint32_t channel;
    uint32_t count;
    uint32_t option;
} MeasureRequest;

拡張ルール

- フィールド追加は末尾追加のみ
- フィールド削除禁止
- フィールド順変更禁止

FWはPayloadLengthまでを解析する。

存在しないフィールドはデフォルト値を使用する。

---

4.2 Flexible Structure

仕様変更が多いコマンドに使用する。

対象例

- Get Configuration
- Set Configuration
- Device Information
- Diagnostic

各パラメータはパラメータIDで識別する。

例

Parameter ID : TIMEOUT
Value        : 100

Parameter ID : MODE
Value        : 2

Parameter ID : RETRY_COUNT
Value        : 10

拡張ルール

- Parameter ID追加は自由
- Parameter IDの意味は変更しない
- 未対応Parameter IDは無視する

---

5. Capability

接続直後にCapabilityを取得する。

Capabilityは項目ごとに問い合わせる。

CMD_GET_CAPABILITY

Request

Capability ID

Response

Supported

または

Capability Value

Capabilityは必要なものだけ問い合わせる。

---

5.1 Supported Commands

Capability ID

CAP_COMMAND_START_MEASURE
CAP_COMMAND_STOP_MEASURE
CAP_COMMAND_READ_LOG
CAP_COMMAND_SET_CONFIG
CAP_COMMAND_GET_CONFIG

戻り値

Supported = true / false

---

5.2 Command Capability

コマンドごとの対応Parameterを問い合わせる。

例

Request

CAP_SET_CONFIG_RETRY_COUNT

Response

Supported = true

Request

CAP_SET_CONFIG_AUTO_RESET

Response

Supported = false

START_MEASUREも同様

CAP_START_MEASURE_TRIGGER

CAP_START_MEASURE_TIMESTAMP

CAP_START_MEASURE_AVERAGE

などを個別に問い合わせる。

---

5.3 Limits

能力値を取得する。

例

CAP_MAX_PAYLOAD_SIZE

CAP_MAX_CHANNEL

CAP_MAX_LOG_SIZE

CAP_MAX_FILE_SIZE

Response

Value

---

6. Sending Rule (PC)

PC内部では常に最新のデータモデルを保持する。

接続時に必要なCapabilityを取得し、

対応しているParameterのみ送信する。

例

PC内部

Timeout
Mode
RetryCount
AutoReset

Capability

RetryCount : Supported

AutoReset : Unsupported

送信

Timeout
Mode
RetryCount

AutoResetは送信しない。

---

7. Receiving Rule (FW)

Fixed Structure

- PayloadLengthまでを解析する
- 存在しないフィールドはデフォルト値を使用する

Flexible Structure

- 未対応Parameter IDは無視する
- 未送信Parameterはデフォルト値を使用する

---

8. Connection Sequence

Case1

PC：最新版

FW：最新版

sequenceDiagram

participant PC
participant FW

PC->>FW: Connect

PC->>FW: GET_CAPABILITY(CAP_SET_CONFIG_RETRY_COUNT)
FW-->>PC: Supported

PC->>FW: GET_CAPABILITY(CAP_SET_CONFIG_AUTO_RESET)
FW-->>PC: Supported

PC->>FW: SET_CONFIG(TIMEOUT, MODE, RETRY_COUNT, AUTO_RESET)

FW-->>PC: ACK

---

Case2

PC：最新版

FW：旧版

sequenceDiagram

participant PC
participant FW

PC->>FW: Connect

PC->>FW: GET_CAPABILITY(CAP_SET_CONFIG_RETRY_COUNT)
FW-->>PC: Supported

PC->>FW: GET_CAPABILITY(CAP_SET_CONFIG_AUTO_RESET)
FW-->>PC: Unsupported

Note over PC: AUTO_RESETは送信しない

PC->>FW: SET_CONFIG(TIMEOUT, MODE, RETRY_COUNT)

FW-->>PC: ACK

---

Case3

PC：旧版

FW：最新版

sequenceDiagram

participant PC
participant FW

PC->>FW: Connect

PC->>FW: GET_CAPABILITY(CAP_SET_CONFIG_RETRY_COUNT)
FW-->>PC: Supported

Note over PC: 旧PCはAUTO_RESETを知らない

PC->>FW: SET_CONFIG(TIMEOUT, MODE)

Note over FW: RETRY_COUNT, AUTO_RESETはデフォルト値

FW-->>PC: ACK

---

9. Compatibility Rules

Item| Rule
Header変更| 禁止
エンディアン変更| 禁止
Protocol Version| 管理しない
Fixed Structure追加| 末尾追加のみ
Fixed Structure削除| 禁止
Fixed Structure順序変更| 禁止
Parameter ID追加| 自由
Parameter ID削除| 非推奨
未対応Parameter| 無視
Capability追加| 自由

---

10. Summary

項目| 採用方式
Header| Command ID + Payload Length
数値データ| Little Endian
高速通信| Fixed Structure
設定系| Parameter ID方式
Capability| 項目単位問い合わせ
後方互換| Payload Length + Capability
機能判定| Capability
Protocol Version| 使用しない
固定構造体拡張| 末尾追加のみ
Parameter追加| Parameter ID追加