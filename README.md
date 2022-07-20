# ものコンテクニック集

## 部品制御

### クロック
5番ピンに入れるパルス信号のこと

よく使うので関数化するべき

```c++
// clockは予約語っぽいので使えない
void clk() {
  digitalWrite(5, HIGH);
  delay(3); //2-5msくらいがよい
  digitalWrite(5, LOW);
}
```

### DCモータ

クロックが必要
Arduino側電源を抜いても動き続ける(うるさいから気を付けたほうがいい)

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

12-15番ピンはステッピングモータと、16-17はDCモータと共用なので注意
