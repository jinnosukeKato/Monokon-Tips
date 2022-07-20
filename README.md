# ものコンテクニック集

## DCモータ

5番ピンにクロックを送らないと動かない
Arduino側電源を抜いても動き続ける

## ステッピングモータ

12-15番ピンにパターン通りに信号を送り、クロックを入れると動く
| 12   | 13   | 14   | 15   | 
| :--: | :--: | :--: | :--: | 
| HIGH | HIGH | LOW  | LOW  | 
| LOW  | HIGH | HIGH | LOW  | 
| LOW  | LOW  | HIGH | HIGH | 
| HIGH | LOW  | LOW  | HIGH | 
