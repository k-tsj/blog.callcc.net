---
layout: post
title:  RubyのFile.openをScalaで実装する
date:   2010-05-30 00:00:00 +0900
categories: scala
---

タイトルはやや大げさですが。

Scalaでファイル処理をする際、普段Rubyを使っている身としてはいちいち紋切り型のコードでInputStreamをBufferedReaderやらにラップしないといけないのが面倒で以下のようにできないかと思っていたものの、
うまい実装方法が浮かばなくてあきらめていた。

```scala
File.open("foo.txt") { (in: BufferedReader) =>
  File.open("bar.txt") { (out: BufferedWriter) =>
    out.write(in.read)
  }
}
```

と、これはScala 2.7までのお話。[^1]

先日[水島さん](http://d.hatena.ne.jp/kmizushima/)がポストした[2.8のimplicit parameterの応用例](http://gist.github.com/408693)を見て、
2.8なら出来るかもと試していたら上記のコードが動くようになった([gist](http://gist.github.com/418960)(2.8.0 RC3で動作))。

必要以上に難しく実装している気もするけど自分の今のスキルだとこの辺が限界。 特に下記のようにopenerの型を指定するためにいちいちResourceのサブクラスでopenを定義しているのは type使えば何とかなるんじゃないかという気がするのだけど。

```scala
object File extends Resource {
  def open[A <: Closeable, B, That](path: String)(f: (A) => B)
                                   (implicit inferencer: StreamTypeInferencer[A, That],
                                             opener: FileOpener[That],
                                             converter: (StreamConverter[That]) => A): B = {
    super.open(path)(f)
  }
}
```

## 追記
大事なことを書き忘れた。

上記のコードはBuffered〜だけを対象にしたものではないです。

Fileの例でいえば、下記のようにInputStream/OutputStreamを生成する定義だけ書いておけば、 あとはInputStreamReaderだろうがBufferedReaderだろうが好きなように使えるようになるというのがポイント。 
また、当然File以外にもいろいろ使えます。(gist上のコードはOpenURIも定義済み)

```scala
trait FileOpener[A] extends ResourceOpener[A]
object FileOpener {
  implicit object FileInputStreamOpener extends FileOpener[InputStream] {
    def open(path: String): InputStream = new FileInputStream(path)
  }
  implicit object FileOutputStreamOpener extends FileOpener[OutputStream] {
    def open(path: String): OutputStream = new FileOutputStream(path)
  }
}
```

[^1]: たぶん。実は2.7でも出来るかも。。。
