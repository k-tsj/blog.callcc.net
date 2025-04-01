---
layout: post
title:  test-unit-power_assert gem
date:   2014-05-31 00:00:00 +0900
categories: ruby
---

Groovyなどいくつかの言語のテストフレームワークには、テスト失敗時にコードの各式の値を出力するPower Assertと呼ばれる機能があります。 
今回、Ruby 2.0以上 + test-unit向けに同様のものを実装してみました([test-unit-power_assert](https://github.com/k-tsj/test-unit-power_assert))。

利用例は以下の通り。

```ruby
require 'test/unit'
require 'test/unit/power_assert'

class MyTest < Test::Unit::TestCase
  def test_failed
    power_assert do
      "0".class == "3".to_i.times.map {|i| i + 1 }.class
    end
  end
end
```

power_assertメソッドにテスト対象の式をブロックとして与えるだけですが、出来れば-rオプションを使うなどしてテストコードが読み込まれる前にこのライブラリをrequireするようにしておくと情報の取りこぼしを防げます。

```ruby
# 事前requireなし

        "0".class == "3".to_i.times.map {|i| i + 1 }.class
            |            |    |     |                |
            |            |    |     |                Array
            |            |    |     [1, 2, 3]
            |            |    #<Enumerator: 3:times>
            |            3
            String

# 事前requireあり

        "0".class == "3".to_i.times.map {|i| i + 1 }.class
            |     |      |    |     |                |
            |     |      |    |     |                Array
            |     |      |    |     [1, 2, 3]
            |     |      |    #<Enumerator: 3:times>
            |     |      3
            |     false
            String
```

## 追記(2014/08/04)
test-unitにPower Assertが標準機能として取り込まれassertメソッドにブロックを渡せるようになりました ([[ruby-list:49902] [ANN] test-unit 3.0.0](http://blade.nagaokaut.ac.jp/cgi-bin/scat.rb/ruby/ruby-list/49902))。 
これを受けてtest-unit-power_assertもバージョンアップを行い、同様の書き方ができるようにしています。
