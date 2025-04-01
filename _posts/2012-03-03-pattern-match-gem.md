---
layout: post
title:  pattern-match gem
date:   2012-03-03 00:00:00 +0900
categories: rubya
---

先日ポストした[ScalaっぽいパターンマッチをRubyで実装する]({{ site.baseurl }}{% link _posts/2012-02-04-scala-like-pattern-matching-in-ruby.md %})をベースに 一通り機能を揃えてライブラリ化した([pattern-match](https://github.com/k-tsj/pattern-match))。
面白そうなパターンをいくつか例に取ってみると、まず多重代入。

```ruby
match([0, [1, 2, 3, 4]]) {
  with(_[a, _[b, *c, d]]) { # `Array.(a, Array.(b, *c, d))'と同じ
    p [a, b, c, d] #=> [0, 1, [2, 3], 4]
  }
}
```

Gaucheの[util.match](http://practical-scheme.net/gauche/man/?l=jp&p=util.match)由来の___、__k。

```ruby
match([[0, 1], [2, 3]]) {
  with(_[_[a, b], ___]) {
    p [a, b] #=> [[0, 2], [1, 3]]
  }
}
```

赤黒木のbalance(参考までに[Scala版実装](http://d.hatena.ne.jp/kmizushima/20090415/1239778393))。

```ruby
Node = Struct.new(:left, :key, :right)
class R < Node; end
class B < Node; end

def balance(left, key, right)
  match([left, key, right]) {
    with(_[R.(a, x, b), y, R.(c, z, d)]) { R[B[a, x, b], y, B[c, z, d]] }
    with(_[R.(R.(a, x, b), y, c), z, d]) { R[B[a, x, b], y, B[c, z, d]] }
    with(_[R.(a, x, R.(b, y, c)), z, d]) { R[B[a, x, b], y, B[c, z, d]] }
    with(_[a, x, R.(b, y, R.(c, z, d))]) { R[B[a, x, b], y, B[c, z, d]] }
    with(_[a, x, R.(R.(b, y, c), z, d)]) { R[B[a, x, b], y, B[c, z, d]] }
    with(_) { B[left, key, right] }
  }
end
```

特殊なパターンとしてDuck Typing的なことも出来るようにしてみた。ただ、実用性があるかというと、どうなんだろう……。

```ruby
match(10) {
  with(Object.(:to_i => a, :foobar => b)) { :not_match }
  with(Object.(:to_i => a, :to_s.(16) => b)) {
    p [a, b] #=> [10, "a"]
  }
}
```

ちなみに、[ruby-trunk - Feature #4085 : Refinements and nested methods](http://bugs.ruby-lang.org/issues/4085)をサポートする環境であれば、 次のような書き方も出来るようになっている[^1]。Refinements素晴らしい。

```ruby
def balance(left, key, right)
  match([left, key, right]) {
    with(_[R[a, x, b], y, R[c, z, d]]) { R[B[a, x, b], y, B[c, z, d]] }
    with(_[R[R[a, x, b], y, c], z, d]) { R[B[a, x, b], y, B[c, z, d]] }
    with(_[R[a, x, R[b, y, c]], z, d]) { R[B[a, x, b], y, B[c, z, d]] }
    with(_[a, x, R[b, y, R[c, z, d]]]) { R[B[a, x, b], y, B[c, z, d]] }
    with(_[a, x, R[R[b, y, c], z, d]]) { R[B[a, x, b], y, B[c, z, d]] }
    with(_) { B[left, key, right] }
  }
end
```

[^1]: r29944向けパッチで動作確認済み
