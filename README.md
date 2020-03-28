## アナログリモコン ライブラリ
* [1.アナログリモコン特徴](#1アナログリモコン特徴)
* [2.リモコンコード](#2リモコンコード)
* [3.エンコード、タイミング](#3エンコードタイミング)
* [4.使用例](#4使用例)
* [5.UART通信プロトコル](#5uart通信プロトコル)

### 1.アナログリモコンライブラリについて
このライブラリはBitTradeOne社 [リモコンロボ](https://bit-trade-one.co.jp/adkrbt/) と [クアッドクローラー](https://bit-trade-one.co.jp/adcrbt/) 用に開発した赤外線リモコン用のArduinoライブラリです。

**対応リモコン**
- NECフォーマットのリモコン
- BitTradeOne社 [アナログリモコン](https://bit-trade-one.co.jp/adkrbt/)

**対応ロボット**
- 外部割込み対応のポートに赤外線受光部を接続したArduinoロボット/ボード
[mBot＋アナログリモコン(動画)](http://sohta02.web.fc2.com/images/MAQ04884.MP4)  
- Atmega328p(D2,D3), ATSAMD21(D4以外), 他  
　https://www.arduino.cc/reference/en/language/functions/external-interrupts/attachinterrupt/
![remote3](../../../docs/raw/master/images/remoteA3.png)　　
![remote](../../../docs/raw/master/images/remoteA.JPG)

**開発環境**
- Arduino IDE
- スクラッチ互換アプリ [「つくるっち」](http://sohta02.web.fc2.com/familyday_app.html)

### 2.アナログリモコン特徴
アナログリモコンはリモコンロボなどArduinoを使ったロボット用の赤外線リモコンです。NECフォーマットなど通常の赤外線方式比較して下記特徴があります。衝突を前提としたフォーマットになっており、CH1, CH2, CH3それぞれで送信周期を変えることで3つまでの赤外線リモコンを同時に使うことが出来ます。
||NECフォーマット|アナログリモコン|効果|
|---|:-:|:-:|---|
|送信周期|108ms|CH1-50ms<br />CH2-66ms<br />CH3-40ms|複数CH対応、レスポンス向上、衝突した時の挙動改善|
|送信bit数|32bit|15bit||
|送信時間|~85.4ms(~152T)|11.6ms(33T)|レスポンス向上、複数のリモコンが送信したとき衝突しにくいように|
|リピートコード|使用|未使用|最初の送信が衝突しても問題ないように|
|ボタンを離したとき|送信停止|2秒間送信|ボタンを離したときの挙動改善|



### 3.ライブラリ仕様

```
ヘッダファイル
  analogRemote.h

クラス名
  analogRemote

コンストラクタ
  analogRemote(
    uint8_t _mode_xyKeys = MODE_NORMAL,      // MODE_xx
    uint8_t _port_irrx = 2,                  // IR_RX port(L active)
    void (*_funcLed)(uint8_t onoff) = NULL); // turn LED on/off

メソッド
  int checkUpdated(void);              // update remocon data
                                       // return REMOTE_xx

プロパティ
  uint8_t  keys;    // BUTTON_xx
  int16_t  x;
  int16_t  y;
  uint8_t  xyKeys;	// XY_xx, joystick direction
  uint8_t  xyLevel;	// joystick level

定義
  MODE_NORMAL         // disable xyKewys
  MODE_XYKEYS         // enable xyKeys
  MODE_XYKEYS_MERGE   // enable xyKeys and merge it to keys

  REMOTE_OFF       // no data
  REMOTE_YES       // get NEC remote / released
  REMOTE_ANALOG    // get analog remote
  
```

### 4.使用例 (atmega328p)
```
#include <analogRemote.h>

void swLed(uint8_t onoff)
{
  digitalWrite(13, onoff);
}
analogRemote remote(MODE_NORMAL, 2, swLed);

void setup()
{
  pinMode(13, OUTPUT);
  Serial.begin(115200);
}

void loop()
{
  if(remote.checkUpdated()) {
    Serial.println(remote.keys);
  }
}
```

### 2.リモコンコード
アナログリモコンは15bitの独自フォーマットになっています。
|ビット|機能|データ|
|---|---|---|
|bit14~13|リモコンチャンネル|01-Channel1<br />10-Channel2<br />00-Channel3|
|bit12~8|ジョイスティック上下|11111-上<br />10000-中立<br />00000-下|
|bit7~3|ジョイスティック左右|11111-右<br />10000-中立<br />00000-左|
|bit2~0|キー|000-OFF<br />001-CENTER<br />010-UP<br />011-RIGHT<br />100-LEFT<br />101-DOWN|

### 3.エンコード、タイミング
キャリア周波数 38kHz  
T=350us  
リモコンコードはスタートビットから始まる [biphaseエンコード](https://ja.wikipedia.org/wiki/%E4%BC%9D%E9%80%81%E8%B7%AF%E7%AC%A6%E5%8F%B7) になってます。01 10000 10000 000のとき
![remote](../../../docs/raw/master/images/remoteA2.png)
|データ|赤外線出力|
|---|---|
|スタートビット|000(3T)|
|bit[n]=1|10又は01(1Tx2)|
|bit[n]=0|11又は00(2T)|

### 5.UART通信プロトコル
リモコン通信は特殊なプロトコルを使っています。
||戻り値|
|---|---|
|byte4|キーコード|
|byte5~6|ジョイスティックX|
|byte7~8|ジョイスティックY|

キーコードはアナログリモコン、標準リモコン（NECフォーマット）両対応です。
|ボタン|キーコード|
|---|---|
|POWER|0x45|
|B|0x46|
|MENU|0x47|
|TEST|0x44|
|RETURN|0x43|
|C|0x0D|
|UP|0x40|
|LEFT|0x07|
|CENTER|0x15|
|RIGHT|0x09|
|DOWN|0x19|
|0|0x16|
|1|0x0C|
|2|0x18|
|3|0x5E|
|4|0x08|
|5|0x1C|
|6|0x5A|
|7|0x42|
|8|0x52|
|9|0x4A|
|アナログCENTER|0x61|
|アナログUP|0x62|
|アナログRIGHT|0x63|
|アナログLEFT|0x64|
|アナログDOWN|0x65|

クアッドクローラー用ライブラリはジョイスティック操作もキーコードとして返します。
|ボタン|キーコード|
|---|---|
|ジョイスティックUP_R|0x70|
|ジョイスティックUP|0x71|
|ジョイスティックUP_L|0x72|
|ジョイスティックRIGHT|0x73|
|ジョイスティックLEFT|0x74|
|ジョイスティックDOWN_R|0x75|
|ジョイスティックDOWN|0x76|
|ジョイスティックDOWN_L|0x77|
