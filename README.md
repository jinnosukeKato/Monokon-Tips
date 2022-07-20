# ものコンテクニック集

具体的なコードは[こちら](https://github.com/jinnosukeKato/Monokon-Template/blob/master/monokon_template.ino)

## 解き方

各部品の制御ごとに関数にして、それを組み立てるようにするのが基本

それぞれの具体的な実装は[俺の書いたコード](https://github.com/jinnosukeKato/Monokon-Template/blob/master/monokon_template.ino)を**参考**にするといいと思う
あくまで参考に、自分で考えるのが優先

## 入力処理

回路の作る人によってプルアップ・プルダウンとかコネクタのパターン変わると思うから臨機応変に対応できるとよい

`#define`でピン番号とプルアップ・ダウン置き換えするのがベターかな

### タクトスイッチ

普通に`digitalRead`で読み取ってやればいい

### フォトダイオード

アナログ値を出力してるけど、半分隠すとか問題に出てこないと思うので`digitalRead`でいいと思う

### 可変抵抗

こいつは`analogRead`で読み取って値の範囲で分岐させることが求められる

## 部品制御

### クロック

5番ピンに入れるパルス信号のこと

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

12-15番ピンにパターン通りに信号を送り、クロックを入れると1ステップ分進む

| 12   | 13   | 14   | 15   | 
| :--: | :--: | :--: | :--: | 
| HIGH | HIGH | LOW  | LOW  | 
| LOW  | HIGH | HIGH | LOW  | 
| LOW  | LOW  | HIGH | HIGH | 
| HIGH | LOW  | LOW  | HIGH | 

↑これで1ステップ分進む

二次元配列を使うと便利
逆回ししたいときはパターンを逆から回せばよい

### ブザー

`analogWrite`で100くらい入れてやると音が鳴る `digitalWrite`では鳴らないので注意

```c++
analogWrite(4, 100);
delay(任意の秒数);
analogWrite(4, 0);
```

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

##### 7セグメント両側同時点灯

2番と3番を高速に切り替えてあげればいい

それぞれ10ms程度光らせてあげる(delayを挟む)とちらつき・ぼやつきがない  
かなりシビアなので10ms程度を基準に調整してみてほしい

#### 7セグメント両点灯+DCモータ回転

DCモータは一度クロックを入れるとずっと回ってくれるので、それを利用してDCモータ制御→セグメント制御の順番にするといい

#### 7セグメント両点灯+ステッピングモータ回転

ものコンの県大会レベルの問題中で一番難易度が高い

```c++
void segStep(int l, int r) {
  for (int n = 0; n < 4; n++) {
    digitalWrite(2, LOW); // 消しとくと事故が起きにくい
    digitalWrite(3, LOW);
    _step(n); // 1パターン(1/4ステップ)進ませる関数
    if (n % 2 == 0) {
      segL(l); // 左セグメントを点灯させる関数
    } else {
      segR(r); // 右セグメントを点灯させる関数
    }
    delay(1); // ここを調整 限界まで短くてよさげ
  }
}
```

このように、1ステップ進ませる中に割り込ませるのがいいと思われる

かなり光り方が弱まってしまうので、ベストな光の具合になるようにアルゴリズムやそれぞれの秒数を研究してほしい

## 高度な内容

<details>
<summary>気になる人だけ読んでもらえればOKです</summary>
ここから下は将来誰かがやってくれると面白いと思う内容

全然直接的には関係ないです 読み飛ばしてくれてOK

### IDEの選択

Arduino IDEを使うのが標準だけど、 Jetbrains の [CLion](https://www.jetbrains.com/ja-jp/clion/) などの強力なIDE使えばめちゃくちゃ楽できそう

ちなみに Jetbrains の IDE は学生ならすべて無料なので強くお勧めする

### 言語仕様

俺はC系に詳しくないので間違ってたらごめんなさい

Cに準拠していると思われがちなArduino言語だけど、実はC++11が標準なのでfor-eachが普通に使える  
正確には`avr-g++`でAVR向けに[クロスコンパイルしている](https://garretlab.web.fc2.com/arduino/introduction/compile_process/)  
なので設定ファイルいじったりコンパイラ差し替えればC++17とかも行けたはず  
詳しくは Arduino forum とかを漁ると[出てくる](https://forum.arduino.cc/t/arduino-and-c-17-avr-gcc-8-x/545021/2)かも

そもそもAVR向けにコンパイルできればいいので、今Bunで話題のZig([成功してる](https://zenn.dev/k_abe/articles/1dc65f8345d908))とか最近でたCarbon(C++互換らしい)とか特に[Rust](https://book.avr-rust.com/)でやったら誰も採点できないみたいな風になりそうで面白いから誰かやってくれ  
ゴリゴリのC++とか高度なCとかでやっても普通に採点できるのかみたいなところはあるけどね

### そもそも

ルール読めばわかるけど、Arduino使わなくてもいいというか、ラズパイとかも想定していそうではある

となると、言語の幅はめちゃくちゃ広がると思われる(PythonとかJava, KotlinとかなんならJSとかでもできるのかな)

ぜひとも将来の後輩にそういうことやってくれる奴がいることを期待してます
</details>
