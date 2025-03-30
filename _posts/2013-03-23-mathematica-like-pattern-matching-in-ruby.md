---
layout: post
title:  MathematicaっぽいパターンマッチをRubyで実装する
date:   2013-03-23 00:00:00 +0900
categories: ruby
---

[ScalaっぽいパターンマッチをRubyで実装する](http://www.callcc.net/diary/20120204.html#p01)から始まった 一連の[pattern-match](https://github.com/k-tsj/pattern-match)ライブラリ関連エントリのおそらく最終章。 
任意のオブジェクトに対して正規表現相当のマッチをできるようにしてみたのでその紹介です。

どういうことかと言うと、例えば配列をFixnumの前後で分割したりとか

```ruby
match([:a, 0, :b, :c]) do
  with(_[*a, Fixnum, *b]) do
    a #=> [:a]
    b #=> [:b, :c]
  end
end
```

連続した要素の積が12になるところを探したりとか

```ruby
match([1, 2, 3, 4, 5]) do
  with(_[*_, *a, *_], guard { a.inject(:*) == 12 }) do
    a #=> [3, 4]
  end
end
```

「シーケンス中で連続して同じ値が入っている各箇所について，2 個目以降は削除したシーケンス」を取得したり ([前後の値も利用したシーケンス処理 - NyaRuRuが地球にいたころ](http://d.hatena.ne.jp/NyaRuRu/20090311/p1))とか

```ruby
def replace_repeated(obj, &block)
  ret = match(obj, &block)
  if ret == obj
    ret
  else
    replace_repeated(ret, &block)
  end
rescue PatternMatch::NoMatchingPatternError
  obj
end

replace_repeated([1, 2, 4, 4, 3, 3, 4, 0, 0]) do
  with(_[*a, x, x, *b]) { [*a, x, *b] }
end #=> [1, 2, 4, 3, 4, 0]
```

こういった処理が簡単に書けるようになりました。

先日のRuby開発者会議でパターンマッチを入れたいねみたいな話をしてきたのですが、 せっかくならこのぐらい出来るようになると楽しそうです。
