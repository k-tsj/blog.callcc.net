---
layout: post
title:  ruby-trunk-changesをgitから参照する
date:   2011-12-12 00:00:00 +0900
categories: ruby
---

Rubyの開発に興味がある人にはおなじみだと思いますが、[PB memo](http://d.hatena.ne.jp/nagachika/)にてruby-trunk-changesというものが公開されています。
コミッタのnagachikaさんがCRubyの変更内容についてコミット単位で解説をされているという非常に参考になるコンテンツですが、どうせならWebからだけでなくgitからも参照したいと思ってgit notes用のデータを用意してみました。
コミットの差分とその背景がまとめて見られるようになって便利です。

```
$ git log -p
commit 09c399b68e6162b68e52dfd22763408def2f813e
Author: naruse <naruse@b2dd03c8-39d4-4d8f-98ff-823fe69b080e>
Date: Wed Dec 7 22:13:04 2011 +0000

It is fixed in FreeBSD 9.0 RC3, not RC2.

So skip until it is released.

git-svn-id: svn+ssh://ci.ruby-lang.org/ruby/trunk@33973 b2dd03c8-39d4-4d8f-98ff-823fe69b080e

Notes (ruby-trunk-changes):
r33967 で FreeBSD でこけるテストの skip 条件を緩和していましたが、まだ Release
Candidate が出ているだけで RC3 から修正済みなので FreeBSD 9.0 がリリースされる
まではスキップするようにしています。

diff --git a/test/-ext-/wait_for_single_fd/test_wait_for_single_fd.rb b/test/-ext-/wait_for_single_fd/test_wait_for_single_fd.rb
index 72914f0..64c05c1 100644
--- a/test/-ext-/wait_for_single_fd/test_wait_for_single_fd.rb
+++ b/test/-ext-/wait_for_single_fd/test_wait_for_single_fd.rb
@@ -23,7 +23,7 @@ class TestWaitForSingleFD < Test::Unit::TestCase
def test_wait_for_invalid_fd
# FreeBSD 8.2 or prior sticks this
# http://redmine.ruby-lang.org/issues/5524
- skip if /freebsd[1-8]/ =~ RUBY_PLATFORM
+ skip if /freebsd[1-9]/ =~ RUBY_PLATFORM
with_pipe do |r,w|
wfd = w.fileno
w.close
```

使い方ですが、CRubyのgitリポジトリ(git://github.com/ruby/ruby.git)をcloneしてきた後.git/configに以下の設定を加えます。

```
[core]
# デフォルトで参照するnotes。この設定はせずに"git log --show-notes=ruby-trunk-changes"とする方法もあり。
notesRef = refs/notes/ruby-trunk-changes
[remote "k-tsj"]
# notesのfetch先。
fetch = +refs/notes/*:refs/notes/*
url = git://github.com/k-tsj/ruby.git
```

設定後、k-tsjをfetchすれば準備は完了です。
なお、nagachikaさんに許可をいただいたのでgithub上のデータは今後定期的に更新していく予定です。 参考までに更新用スクリプトへのリンクを張っておきます。

- [update-ruby-trunk-changes-notes.sh](https://gist.github.com/1467442)
