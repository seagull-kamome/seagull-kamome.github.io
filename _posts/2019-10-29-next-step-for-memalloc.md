---
titie: "memallocプロジェクトの今後"
author: "Hattori, Hiroki"
layout: "post"
tag: ["ProjectMemalloc"]
categories: ["blog"]
---

　とりあえずTLSFが動くようになったっぽいので、やるかどうかは別として
 この先何ができるか考えてみよう。

<!-- more -->

　色々なアルゴリズムのコードをつっこんでおけば後々使えるし、教材と
しても世間様のお役に立てるかもしれない。個人的にどうしても欲しいのは
バディとslabだな。

　TLSFもよく考えると、オプションとしてベストフィットが選べるとか、
第二レベルシフトを0にするとシングルレベルになる的な拡張をすると便利
なのかもしれない。

　slabは前に作った事あるけど、問題になったのは空いたページを何時ページ
アロケータに返すかという事と、binの範囲をどうするかって事だった。
次やるとしたら、適当にアイドル時にページ開放を呼んでもらったり、binに
ついても開始サイズと増分、総数をコンフィグで与えるようにすれば
いいんだろうと思う、用途的に考えて。

　あと、どうせだったらGC付きのアロケータも部品用に確保しておきたい。

　次に排他だな。軽量排他プリミティブを提供してもらって細粒度ロックする
方法と、単にジャイアントロックするラッパをかぶせてもらう方式があるわけ
だけど、前者はライブラリ側でサポートしてやらないといけない。
　あとできればアトミック操作使って部分的にでもlock-freeを使っていきたい
ところだが、果たして使える部分があるだろうか…



