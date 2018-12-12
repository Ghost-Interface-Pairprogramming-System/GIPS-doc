# はじめに
こちらはGhost-Interface-Pairprogramming-System(以下GIPS)の各種仕様に関するドキュメントです。

# GIPS通信仕様
IDEプラグインと伺かとの通信仕様に関するドキュメントです。

各イベントはSSTP NOTIFY/1.1に基づいてSSTPサーバに通知されます。

IDEに関するイベントをゴーストに実装したい方、
GIPS通信仕様に則ったゴーストにIDEからイベントを送信したい方はご覧ください。


## 仕様はこちら
- [GIPS通信仕様 ver 0.1](gips/0_1.md)

## ゴースト側の実装例はこちら
- YAYA -> [yaya_example](gips/example/yaya_example.txt)
- 里々 -> [satori_example](gips/example/satori_example.txt)