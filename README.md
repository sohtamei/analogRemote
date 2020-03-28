## アナログリモコン ライブラリ
* [1.アナログリモコンライブラリについて](#1アナログリモコンライブラリについて)
* [2.アナログリモコン特徴](#2アナログリモコン特徴)
* [3.ライブラリ仕様](#3ライブラリ仕様)
* [4.使用例(atmega328p)](#4使用例atmega328p)
* [5.キー定義](#5キー定義)
* [6.エンコード、タイミング](#6エンコードタイミング)

### 1.アナログリモコンライブラリについて
このライブラリはBitTradeOne社 [リモコンロボ](https://bit-trade-one.co.jp/adkrbt/) と [クアッドクローラー](https://bit-trade-one.co.jp/adcrbt/) 用に開発した赤外線リモコン用のArduinoライブラリです。アナログリモコンとの組み合わせで3chまでの同時制御＝ロボット対戦に対応しています。

**対応リモコン**
- NECフォーマットのリモコン
- BitTradeOne社 [アナログリモコン](https://bit-trade-one.co.jp/adkrbt/)

**対応ロボット**
- BitTradeOne社 [リモコンロボ](https://bit-trade-one.co.jp/adkrbt/) と [クアッドクローラー](https://bit-trade-one.co.jp/adcrbt/) 
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

### 4.使用例(atmega328p)
D2にIR受光部、D13にLEDを接続したときのサンプルコードです。
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

### 5.キー定義
キーコードはアナログリモコン、標準リモコン（NECフォーマット）両対応です。
|ボタン|キーコード|
|---|---|
|BUTTON_POWER|0x45|
|BUTTON_B|0x46|
|BUTTON_MENU|0x47|
|BUTTON_TEST|0x44|
|BUTTON_RETURN|0x43|
|BUTTON_C|0x0D|
|BUTTON_UP|0x40|
|BUTTON_LEFT|0x07|
|BUTTON_CENTER|0x15|
|BUTTON_RIGHT|0x09|
|BUTTON_DOWN|0x19|
|BUTTON_0|0x16|
|BUTTON_1|0x0C|
|BUTTON_2|0x18|
|BUTTON_3|0x5E|
|BUTTON_4|0x08|
|BUTTON_5|0x1C|
|BUTTON_6|0x5A|
|BUTTON_7|0x42|
|BUTTON_8|0x52|
|BUTTON_9|0x4A|
|BUTTON_A_XY|0x60|
|BUTTON_A_CENTER|0x61|
|BUTTON_A_UP|0x62|
|BUTTON_A_RIGHT|0x63|
|BUTTON_A_LEFT|0x64|
|BUTTON_A_DOWN|0x65|

MODE_XYKEYSのときはジョイスティック操作もキーコードとして返します。
|ボタン|キーコード|
|---|---|
|XY_UP_R|0x70|
|XY_UP|0x71|
|XY_UP_L|0x72|
|XY_RIGHT|0x73|
|XY_LEFT|0x74|
|XY_DOWN_R|0x75|
|XY_DOWN|0x76|
|XY_DOWN_L|0x77|

### 6.エンコード、タイミング
アナログリモコンの技術仕様です。  
キャリア周波数 38kHz  
T=350us  
リモコンコードはスタートビットから始まる [biphaseエンコード](https://ja.wikipedia.org/wiki/%E4%BC%9D%E9%80%81%E8%B7%AF%E7%AC%A6%E5%8F%B7) になってます。01 10000 10000 000のとき  
![remote](../../../docs/raw/master/images/remoteA2.png)
|データ|赤外線出力|
|---|---|
|スタートビット|000(3T)|
|bit[n]=1|10又は01(1Tx2)|
|bit[n]=0|11又は00(2T)|

データは15bitの独自フォーマットになっています。
|ビット|機能|データ|
|---|---|---|
|bit14~13|リモコンチャンネル|01-Channel1<br />10-Channel2<br />00-Channel3|
|bit12~8|ジョイスティック上下|11111-上<br />10000-中立<br />00000-下|
|bit7~3|ジョイスティック左右|11111-右<br />10000-中立<br />00000-左|
|bit2~0|キー|000-OFF<br />001-CENTER<br />010-UP<br />011-RIGHT<br />100-LEFT<br />101-DOWN|
