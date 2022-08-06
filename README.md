# ものコンテクニック集

具体的なコードは[こちら](https://github.com/jinnosukeKato/Monokon-Template/blob/master/monokon_template.ino)

## 解き方

各部品の制御ごとに関数にして、それを組み立てるようにするのが基本

それぞれの具体的な実装は[俺の書いたコード](https://github.com/jinnosukeKato/Monokon-Template/blob/master/monokon_template.ino)を**参考**にするといいと思う
あくまで参考に、自分で考えるのが優先

## アルゴリズム

### 一度しか動かさなくてよいもの(起動後だけ)

`void setup() {}`の中に書くと簡単

### ボタンが押されたらor押されている間の処理

`if`よりも`while`を使ったほうが簡単なことが結構ある

## 入力処理

回路の作る人によってプルアップ・プルダウンとかコネクタのパターン変わると思うから臨機応変に対応できるとよい

`#define`でピン番号とかは置き換えするのがベター  
できれば練習から統一しておくと楽

例)
```c++
#define BUTTON_PIN 22
#define BUTTON_ON HIGH
```

### タクトスイッチ

普通に`digitalRead`で読み取ってやればいい

### フォトダイオード

アナログ値を出力してるけど、半分隠すとか問題に出てこないので`digitalRead`でいいと思う

### 可変抵抗

`analogRead`で読み取って値の範囲で分岐させることが求められることが多い  
**analogReadの場合、ピンはAから始まる番号になる** `例) A0, A1, A2`  
`pinMode(A1, INPUT);`も忘れずに

0から1023までの値で読み取れる

## 部品制御

### クロック

5番ピンに入れるパルス信号のこと

DCモータとステッピングモータを制御するときに必要(Dフリップフロップが間にあるため)

間隔が短すぎるとステッピングモータが回らない場合がある

```c++
// clockは予約語なので使えない
void clk() {
  digitalWrite(5, HIGH);
  delay(3); //2-5msくらいがよい
  digitalWrite(5, LOW);
}
```

### DCモータ

クロックが必要
Arduino側電源を抜いても動き続ける(面倒くさいから気を付けたほうがいい)

LOWを入れて止めると若干惰性で動くので、ピタっと止めたい場合は下のようにする

```c++
digitalWrite(16, HIGH); // 両側にHIGHを入れる
digitalWrite(17, HIGH);
clk(); // クロック
delay(5); // ちょっと待つ
digitalWrite(16, LOW); // 負荷がかかるのでLOWにする
digitalWrite(17, LOW);
clk(); // クロック
```

### ステッピングモータ

12-15番ピンに下図パターン通りに信号を送り、クロックを入れると4ステップ分進む  
1番パターン送信→クロック→2番パターン送信→クロック...のように合間にクロックを送ればよい  
1ステップ3度なので下図パターンで12度回転する

|     | 12番ピン | 13番ピン | 14番ピン | 15番ピン  | 
| :-: | :------: | :------: | :------: | :------: | 
| 1   | HIGH     | HIGH     | LOW      | LOW      | 
| 2   | LOW      | HIGH     | HIGH     | LOW      | 
| 3   | LOW      | LOW      | HIGH     | HIGH     | 
| 4   | HIGH     | LOW      | LOW      | HIGH     | 

↑これで4ステップ分進む

二次元配列を使ってパターンを作ると便利
逆回ししたいときはパターンを逆から回せばよい  
部品は[これ](https://akizukidenshi.com/catalog/g/gP-11839/) 
関東大会とは部品が違うので注意

### ブザー

`analogWrite`で100くらい入れてやると音が鳴る `digitalWrite`では鳴らないので注意

```c++
analogWrite(4, 100); // ON
analogWrite(4, 0); // OFF
```

[`tone`](http://www.musashinodenpa.com/arduino/ref/index.php?f=0&pos=2484)を使うと周波数の指定ができ、消音する処理がいらなくなる

### 7セグメント

ピンは一番上のセグメントから時計周りで最後に中央→点と覚えると簡単

下のコードのように二次元配列を用いてパターン作成すると良いと思う

```c++
int segPattern[10][8] = {
  {HIGH, HIGH, HIGH, HIGH, HIGH, HIGH, LOW, LOW}, //　0
  {LOW, HIGH, HIGH, LOW, LOW, LOW, LOW, LOW}, //　1
  ...
  {LOW, LOW, LOW, LOW, LOW, LOW, LOW, LOW} //　消灯
};
```

消灯パターンや`r`, `l`表示も使うことがあるので入れておくといいかも

**12-15番ピンはステッピングモータと、16-17はDCモータと共用なので注意**

#### 7セグメント両側同時点灯

~2番と3番を高速に切り替えてあげればいい~  
~それぞれ10ms程度光らせてあげる(delayを挟む)とちらつき・ぼやつきがない~  
~かなりシビアなので10ms程度を基準に調整してみてほしい~

下記のようにすることでdelayが不要になることを発見

```c++
// 右セグメント点灯
void segR(int num) {
  seg(10);// ここでリセットかけるとdelayいらない
  digitalWrite(2, LOW);
  digitalWrite(3, HIGH);
  seg(num);
}

void segL(int num){
  // 左も同様に
}

void segW(int left, int right) {
  segL(left);
  delay(1); // 点灯する時間を少し設けたほうがいい
  segR(right);
  delay(1); // 直接loop()に入れるなら不要
}
```

#### 7セグメント点灯+DCモータ回転

DCモータは一度クロックを入れるとずっと回ってくれるので、それを利用してDCモータ制御→セグメント制御の順番にするといい

#### 7セグメント両点灯+ステッピングモータ回転

```c++
void segAndStep(int l, int r) {
  for (int n = 0; n < 4; n++) {
    digitalWrite(2, LOW); // 消しとくと事故が起きにくい
    digitalWrite(3, LOW);
    step(n); // 1ステップ進ませる関数
    segL(l); // 左セグメントを点灯させる関数
    delay(1); // ここを調整 限界まで短くてよさげ(delayMilliseconds使ってもよい)
    segR(r); // 右セグメントを点灯させる関数
    delay(1); // ここを調整 限界まで短くてよさげ(delayMilliseconds使ってもよい)
  }
}
```

このように、1ステップ進ませる中に割り込ませるのがいいと思われる

DCモータが動いちゃうときは16,17番ピンをLOWにしてからクロックを送るように修正するとよい

delayが長いほど光り方が弱まってしまうので、ベストな光の具合になるようにアルゴリズムやdelayの秒数を研究してほしい  
また、クロックのdelayを1msとかに設定してもステッピングモータが回る場合があるので、そこも調整してみてほしい
## 以下は直接関係ない内容です

<details>
<summary>気になる人だけ読んでもらえればOKです</summary>
ここから下は将来誰かがやってくれると面白いと思う内容

全然直接的には関係ないです 読み飛ばしてくれてOK

### IDEの選択

Arduino IDEを使うのが標準だけど、 Jetbrains の [CLion](https://www.jetbrains.com/ja-jp/clion/) などの強力なIDE使えばめちゃくちゃ楽できそう  
やってみたけどPlatform.ioがうまく入らず断念

ちなみに Jetbrains の IDE は学生ならすべて無料なので強くお勧めする

### 言語仕様

俺はC系に詳しくないので間違ってたらごめんなさい
  
`HIGH`は0以外の値、`LOW`は0と等価なので、頭いい人はbit演算とか使うと関東レベルでも楽に解けるかも

Cに準拠していると思われがちなArduino言語だけど、実はC++11なのでfor-eachとかが普通に使える  
正確には`avr-g++`でAVR向けに[クロスコンパイルしている](https://garretlab.web.fc2.com/arduino/introduction/compile_process/)  
なので設定ファイルいじったりコンパイラ差し替えればC++17とかも行けたはず  
詳しくは Arduino forum とかを漁ると[出てくる](https://forum.arduino.cc/t/arduino-and-c-17-avr-gcc-8-x/545021/2)かも

そもそもAVR向けにコンパイルできればいいので、今Bunで話題のZig([成功してる](https://zenn.dev/k_abe/articles/1dc65f8345d908))とか最近でたCarbon(C++互換らしい)とか[Rust](https://book.avr-rust.com/)でもやれそう
ゴリゴリのC++とか高度なCとかでやっても普通に採点できるのかみたいなところはある

Zigはavrdudeを入れるだけでできるっぽい  
自宅PCでやったけど日本語ユーザ名対応してなくて断念(対応するっぽい)
