---
title: "TLSFアロケータ実装メモ"
date: 2019-10-27
tag: ["Algorhytm", "C", "C++", "Embeded"]
---

## はじめに

　アロケータを自前実装する必要が生じたのでTLSFアロケータを実装する事にした。
普通のTLSFアロケータなら本家のコードがあるので、どうせならヘッダオンリー
ライブラリにして、ターゲットとして組み込みマイコンを想定してみる。

- https://github.com/seagull-kamome/memalloc/ の tlfs.c_inc


## 設計メモ

### ヘッダオンリー構成

　ヘッダというかインライン展開用の.c_incファイル。
TLSF\_DECL\_ONLYを非0に定義すると宣言だけが展開され、そうでなければ実装部が
展開される。

    実装ルール
        1. ファイル内で定義するマクロには必ずprefixとしてTLSF_を付ける。
        2. ファイル内で定義したマクロは、アプリに提供する必要があるもの
           を除いて、全てキレイにundefする。
        3. 関数や型を定義する際には、必ずTLSF_PREFIX##tlsf_footypeという
           名前で展開されるようにする。


### アロケーションサイズ
　アロケーションはunit単位。１unitは size_tとポインタのunionなので
普通はそのマシンのワードサイズになる。

　最低アロケーションサイズは2unit。空きチャンクの先頭2unitにフリーリストの
為の双方向リンクが入っているので2unitより小さいサイズに分割できない。


### チャンクヘッダ

　チャンクの先頭に自チャンクのサイズと前チャンクのサイズがunit単位で入っている。
size_tは少なくとも(アドレッシング空間全体-1)を表現可能という事は、unit単位
にしておけば先頭数ビットがフラグとして使える事になるので、自チャンクサイズの
先頭2ビット[^1]を割当済フラグと、フリーリストの先頭にいるかどうかを表すフラグ
に使っている。

[^1] unitのサイズは少なくとも4バイトはあると仮定しているので2ビットがつがえる
     はず。さらに言えば自サイズと前チャンクサイズがあるので合計4ビットが使える。


## first-level-indexとsecond-level-index

　第二レベルのビット幅はマクロでカスタマイズできる。
　第一レベルのビット幅は計算で求まる。

      (sizeof(size_t) * 8) - 第２レベルのビット数 - 下駄履き

　下駄履きは最低確保数を2unitにする為の小細工として1を引いている。
　(NOTE: 単位はunitだからsize_t全域を使う訳は無い。つまりlog2(unitサイズ)を
         さらに引く事ができ、bitmapとfreelist表のサイズを節約できる。)

　FSIとSLIの求め方は下記の通り。

      nunits : unit単位のチャンクサイズ
      sldepth : 2^第二レベルのビット数
      fli = 第一レベルのビット幅 - CLZ((nunits - 1) | (sldepth - 1))
      sli = nunits >> (fli - 第一レベルのビット幅 - sldepth)
      ix = fli * sldepth + sli

### 第二レベルビット幅の制限事項

　第２レベルビットマップの要素としてunitptr_tを使っており、以下の条件を満たす
必要がある。

      (sizeof(uintptr_t) * 8) >= sldepth
      (sizeof(uintptr_t) * 8) mod sldepth == 0


--TODO: 静的検証を追加する。-- 2019-10-27:追加した


## 空きリストのマージ方法

　Boundary-Tag法を使う。なので、2 * sizeof(unit)分がチャンクあたりの
オーバーヘッドになる。


## 考えなくてはいけない残件

### マルチスレッド、マルチコアでの排他。

　割り込み応答性を考えると、ヒープ全体のジャイアントロックとかしたくない。
ロックプリミティブとアトミックプリミティブを提供してもらった上で細粒土ロック
でなんとかする方法が必要。lock-freeは流石に無理だろうし。

### 構造体戻しをレジスタにパックしてくれない処理系への対処

　fliとsliを計算するTLSF\_MAPPING\_INSERTは計算結果を構造体に入れて返すが、
流石にインライン展開されるだろうし、そうでなくても32bit以下の小さい構造体なので
レジスタにパックしてくれる事を期待している。

　一部の間抜けな処理系は最適化してくれないので対策が必要。かと言って明示的に
パックしてしまうと、インライン展開された時にも本当にビット操作にされそうで怖い。


### テスト方法、 カバレッジ計測

　nanocspecで地道にspecテストを書く。


### メモリ破壊チェッカ
### パフォーマンス統計、フラグメント計測
