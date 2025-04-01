---
layout: post
title:  ScalaっぽいパターンマッチをRubyで実装する
date:   2012-02-04 00:00:00 +0900
categories: ruby
---

例によって思いつきドリブン実装。var @ patternをpattern :: varで表現していることを除けば見た目はそれなりになった感がある、ような。
- [match.rb](https://gist.github.com/1734716)

```ruby
require 'match'

class EMail
  def self.unapply(value)
    value.to_s.split(/@/).tap {|parts| raise PatternNotMatchError.new unless parts.length == 2}
  end
end

match([1, "foo-bar@example.com"]) {
  pattern(Array.(i, EMail.(/(w+)-(w+)/.(firstname, 'bar') :: name, domain) :: mail)) {
    p [i, firstname, name, domain, mail] # => [1, "foo", "foo-bar", "example.com", "foo-bar@example.com"]
  }
}
```

パターンマッチは時々欲しくなるのでRuby 3.0あたりで言語仕様に取り込まれないかなぁと思うんだけど、あまりパターンマッチについて議論されているところを見た記憶がない。 
Refinementsが入ればライブラリレベルでがんばれるようになるんだろうか。

## 追記(2012/03/03)
[pattern-match gem]({{ site.baseurl }}{% link _posts/2012-03-03-pattern-match-gem.md %})にてライブラリ化済み。
