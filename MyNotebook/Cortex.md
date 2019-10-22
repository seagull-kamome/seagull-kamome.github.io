---
title: "Cortexメモ"
layout: "default"
author: Hattori, Hiroki
tags: [ "EmbededProgramming", "ARM", "Cortex-M" ]
---

　忘れがちなくせにいざググるとInfocenterが重くて面倒なのでここにメモとして残す。

## レジスタ別名

　R13(SP), R14(LR), R15(PC)。


## ジャンプ条件

　符号なしと符号付きはよく間違えるのでアセンブラ書く時は表を見ながら
慎重に書くこと。

  |      | 覚え方               |意味           | 条件フラグ
   ------|----------------------|---------------|------------
   AL    | Always               | 常に          |
   EQ    | Equal-to             | 等しい        | Z
   NE    | Not-Equal            | 等しくない    | NZ
   MI    | MInus                | 負            | N
   PL    | PLus                 | 非負          | NN
   VS    | V-Set                | overflow      | V
   VC    | V-Clear              | 非overflow    | NV
   CS/HS | Carry-Set ※1        |               | C
   CC/LO | Carry-Clear ※1      |               | NC
   HI    | HIgher ※1           | >             | C and NZ
   LS    | LeSs ※1             | >=            | NC or Z
   GE    | Greater-or-Equal ※2 | >=            | N=V
   LT    | Less-Than ※2        | <             | N/=V
   GT    | Greater-Than ※2     | >             | N=V and NZ
   LE    | Less-or-Equal ※2    | <=            | N/=V and NZ

  ※1: 符号なし ※2: 符号付き



## 割り込み関係

### 割り込みフレーム

　上からR0, R1, R2, R3, R12, R14(LR), R15(PC), xPSRの順。作られるのはもちろんスレッド側のスタック。

 eabiではSPはpre-decrement。

### basepri, primask, faultmask

  primaskとbasepriがどっちがどっちだかわからなくなって困る。

  - basepri: 割り込みマスク優先度。こいつに指定した優先度*以下*の割り込みが禁止になる。
  - primask: 割り込み禁止。1にするとNMIとFAULT以外の割り込み禁止になる。
      cpsid iの状態が取れるし設定できる。
  - faultmask: 例外禁止。1にするとNMI以外の割り込みが禁止になる。
      cpsid fの状態が取れるし設定できる。

### 割り込みのかかるタイミング

  - LDM, STMは連続load/storeしてる最中にも割り込みがかかる。
     (TODO:復帰後に途中から再開するのか、最初からやり直すのか調べる事。
           やり直してくれるなら、シングルコアでのアトミック命令として使える？)

### 割り込みが禁止になるタイミング

  TODO: cpsidあるいはprimaskへのロードを行った後、実際に禁止になるタイミング
        が何処か調べる事。次の命令との間で割り込みはかかっても問題無いけど、
        パイプラインに入ってる命令はどうなる？

### リターンする時にmov pc, lrを使うべきではない

　mov 命令は割り込みからの復帰に対応していないので、lrが割り込み復帰
だったりした場合に普通にfaultを引き起こす。割り込み復帰でなければなんの
問題も無いけど、素直にbx lrかpop {pc}に統一するべき。

  追記: movs pc, lrだとフラグも更新されるので復帰できる？

　Cortex-Mだと関係ないけど、mov pc. lrはARM/thumbのモード切り替えもサポート
していないらしい。(TODO: もしかしてthumb限定動作でよければ、ジャンプ先アドレス
の最下位ビットを立てる手間が省ける？)


### 割り込み発生時のLRの値 (EXC_RETURN値)

　Infocenterから抜粋。

  EXC_RETURN   | Return-to      | Return Frame  | Return SP
  -------------|----------------|---------------|------------
  0xFFFFFFF1   | Handler mode   | main stack    | MSP
  0xFFFFFFF9   | Thread mode    | main stack    | MSP
  0xFFFFFFFD   | Thread mode    | process stack | PSP

  All ther values are reserved.

