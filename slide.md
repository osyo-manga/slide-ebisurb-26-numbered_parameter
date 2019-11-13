### 恵比寿.rb #26
- - -

### Ruby 2.7 の Numbered parameter の
### 予習をしよう

---

#### 自己紹介
- - -

* なまえ  : おしょー
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)
* Rails 歴 1年半              <!-- .element: class="fragment" -->
* 最近のトレンド              <!-- .element: class="fragment" -->
  * `binding_of_caller`
  * `solargraph`

---

## 今日話すこと

---

### Ruby 2.7 の Numbered parameter の
### 予習をしよう

---

## ！！！注意！！！
- - -

* ここに書かれている事は `ruby 2.7.0dev (2019-11-11T23:39:38Z trunk 2407e89725) [x86_64-linux]` に準じています
* Ruby 2.7 リリース後に挙動が変わっている可能性があるので注意して下さい


---

#### Numbered parameter とは
- - -

* ブロックの仮引数を省略して参照できる記法          <!-- .element: class="fragment" -->
* 第一引数を _1, 第二引数を _2 ...という記号で参照できる          <!-- .element: class="fragment" -->
   * _1 は普通の変数のように扱える          <!-- .element: class="fragment" -->
   * preview1 の頃は @1 だった          <!-- .element: class="fragment" -->
* 略してナンパラ      <!-- .element: class="fragment" -->

```ruby
# 今までは仮引数を書く必要があった
[1, 2, 3].map { |it| it.to_s + it.to_s } # => ["11", "22", "33"]
```
<!-- .element: class="fragment" -->
```ruby
# ナンパラを使うと
[1, 2, 3].map { _1.to_s + _1.to_s } # => ["11", "22", "33"]
# これは仮引数 _1 を定義しているのとだいたい同じ意味
[1, 2, 3].map { |_1| _1.to_s + _1.to_s }
```
<!-- .element: class="fragment" -->

---

#### Numbered parameter の利点
- - -

* 仮引数を書くのがつらい問題を解決できる         <!-- .element: class="fragment" -->
* &記法を使うのにも限界がある…               <!-- .element: class="fragment" -->

```ruby
# 単調なブロックでは
%w(homu mami mado).map { |it| it.upcase }
%w(homu mami mado).each { |it| puts it }
%w(homu mami mado).map { |it| "name is " + it }
```
 <!-- .element: class="fragment" -->

↓↓↓
<!-- .element: class="fragment" -->

```ruby
# Symbol#to_proc や Method#to_proc が利用できる
%w(homu mami mado).map(&:upcase)
%w(homu mami mado).each(&method(:puts))
%w(homu mami mado).map(&"name is ".method(:+))
```
<!-- .element: class="fragment" -->


>>>

```ruby
# 引数を取ったりメソッドチェーンすると &記法を使うのがきびしい
(1..5).map { |it| it.to_s(2) } # => ["1", "10", "11", "100", "101"]

%w{72 101 108 108 111}.map { |it| it.to_i.chr }
# => ["H", "e", "l", "l", "o"]

# メソッドチェーンであればこういうふうい記述できるが…
%w{72 101 108 108 111}.map(&:to_i.to_proc >> :chr.to_proc)
# => ["H", "e", "l", "l", "o"]
```

↓↓↓
<!-- .element: class="fragment" -->


```ruby
# ナンパラを使うことで簡潔にかける！
(1..5).map { _1.to_s(2) } # => ["1", "10", "11", "100", "101"]

%w{72 101 108 108 111}.map { _1.to_i.chr }
# => ["H", "e", "l", "l", "o"]
```
<!-- .element: class="fragment" -->

>>>


```ruby
# &記法を使うよりも
%w(homu mami mado).map(&:upcase)
%w(homu mami mado).each(&method(:puts))
%w(homu mami mado).map(&"name is ".method(:+))
```
 <!-- .element: class="fragment" -->

↓↓↓
<!-- .element: class="fragment" -->

```ruby
# こっちのほうがわかりやすくない？
%w(homu mami mado).map { _1.upcase }
%w(homu mami mado).each { puts _1 }
%w(homu mami mado).map { "name is " + _1 }
```
<!-- .element: class="fragment" -->

---

## これは素晴らしいものだ

---

## もうちょっと踏み込んだ話

---


#### lambda を使った場合
- - -

* lambda は仮引数を実引数の数が違うとエラーになる  <!-- .element: class="fragment" -->

```ruby
# proc の場合は引数が多かったり少なかったりしても OK
proc { |a, b| }.call 1
proc { |a, b| }.call 1, 2, 3
```
  <!-- .element: class="fragment" -->

```ruby
# lambda の場合は仮引数と実引数が違うとエラーになる
# OK
lambda { |a, b| }.call 1, 2

# Error
lambda { |a, b| }.call 1

# Error
lambda { |a, b| }.call 1, 2, 3
```
  <!-- .element: class="fragment" -->

>>>

#### lambda + ナンパラ
- - -

* lambda でナンパラを使った場合も同じ挙動            <!-- .element: class="fragment" -->

```ruby
# OK
lambda { _1 + _2 }.call 1, 2

# Error
lambda { _1 + _2 }.call 1

# Error
lambda { _1 + _2 }.call 1, 2, 3
```
<!-- .element: class="fragment" -->

>>>

#### ナンパラをパースするタイミング
- - -

* 読み込み時にパースしている                      <!-- .element: class="fragment" -->
* Proc#parameters で仮引数の定義を取得出来る            <!-- .element: class="fragment" -->

```ruby
p proc { |a, b| }.parameters
# => [[:opt, :a], [:opt, :b]]

p proc { _1 + _2 }.parameters
# => [[:opt, :_1], [:opt, :_2]]

p lambda { |a, b| }.parameters
# => [[:req, :a], [:req, :a]]

p lambda { _1 + _2 }.parameters
# => [[:req, :_1], [:req, :_2]]
```
<!-- .element: class="fragment" -->

---

#### 配列を渡した時の挙動
- - -

* 配列を渡した時は仮引数を定義した時と同じ                   <!-- .element: class="fragment" -->

```ruby
# 引数が1つの時はそのまま配列を受け取る
# a = [1, 2]
proc { |a| [a] }.call [1, 2] # => [[1, 2]]

# 引数が1つの時は配列を展開して受け取る
# a = 1, b = 2
proc { |a, b| [a, b] }.call [1, 2] # => [1, 2]
```
<!-- .element: class="fragment" -->

```ruby
# ナンパラも同じ
# _1 = [1, 2]
proc { [_1] }.call [1, 2] # => [[1, 2]]

# _1 = 1, _1 = 2
proc { [_1, _2] }.call [1, 2] # => [1, 2]
```
<!-- .element: class="fragment" -->

>>>

#### 注意点！
- - -

* _2 を使ったか使ってないかで _1 の意味が変わる

```ruby
# _1 だけ
proc { _1 }.call [1, 2] # => [1, 2]

# _2 も使用した場合
proc { _2; _1 }.call [1, 2] # => 1
```
<!-- .element: class="fragment" -->

```ruby
# Hash をイテレーションする場合に注意
user = { id: 1, name: "homu", age: 14 }

# [key, value] の配列をブロックで受け取る
user.map { _1 }      # => [[:id, 1], [:name, "homu"], [:age, 14]]

# 配列を展開して受け取るので _2 = value
user.map { _2 }      # => [1, "homu", 14]

# _2 を使用していると _1 = key になる
user.map { _2; _1 }  # => [:id, :name, :age]

```
<!-- .element: class="fragment" -->


---

#### ブロックがネストしている時の挙動
- - -

* ナンパラをネストして使用するとエラー          <!-- .element: class="fragment" -->
* RSpec だとブロックがネストするので注意          <!-- .element: class="fragment" -->

```ruby
proc {
  # 外でナンパラを使っている場合は
  _1
  proc {
    # 内側でナンパラを使うとエラーになる
    # Error: numbered parameter is already used in
    _1
  }
}
```
<!-- .element: class="fragment" -->


```ruby
proc {
  # 別々にナンパラを使う分には OK
  proc { _1 }.call "A" # => "A"
  proc { _1 }.call "B" # => "B"
}.call
```
<!-- .element: class="fragment" -->


---

#### 警告になったりエラーになったり
- - -

* _1 という変数を定義する         <!-- .element: class="fragment" -->

```ruby
# warning: `_1' is used as numbered parameter
_1 = 42
```
<!-- .element: class="fragment" -->

* _1 に再代入する         <!-- .element: class="fragment" -->

```ruby
proc {
  _1
  # Error: Can't assign to numbered parameter _1
  _1 = 42
}.call
```
<!-- .element: class="fragment" -->

* 仮引数と併用         <!-- .element: class="fragment" -->

```ruby
# Error: ordinary parameter is defined
proc { |i| _1 }.call
```
<!-- .element: class="fragment" -->

---

#### その他
- - -

* _1 ~ _9 まで使用できる
* def で定義したメソッド内では参照できない
* ナンパラでブロック引数は参照できない
  * 全部の仮引数を定義する必要がある
* _1 という名前は変数名やメソッド名で有効な名前なのでそういうケースで使用する場合は注意する

---

## ここで問題

---

* Q. 以下は何が返る？

```ruby
_1 = :local_variable
proc {
  _1
  proc {
    eval("_1")
  }.call "B"
}.call "A"
```

1. :local_variable            <!-- .element: class="fragment" -->
2. "A"           <!-- .element: class="fragment" -->
3. "B"           <!-- .element: class="fragment" -->
4. 構文エラー           <!-- .element: class="fragment" -->

---

## 答えは
## 3. :local_variable                        <!-- .element: class="fragment" -->
# 🤔                        <!-- .element: class="fragment" -->

---

#### eval("_1") を使用した場合の挙動
- - -

```ruby
# Error: undefined local variable or method `_1' for main:Object (NameError)
proc { eval("_1") }.call 42
```
<!-- .element: class="fragment" -->

```ruby
# OK : ナンパラ _1 を参照する
proc { _1; eval("_1") }.call 42   # => 42
```
<!-- .element: class="fragment" -->

```ruby
proc {
  _1
  # OK : 一番内側のナンパラを参照する
  proc { eval("_1") }.call "B"   # => "A"
}.call "A"
```
<!-- .element: class="fragment" -->

```ruby
_1 = :local_variable
p proc {
  _1
  # OK? : 内側のナンパラではなくて外にあるローカル変数を参照する
  proc { eval("_1") }.call "B"   # => :local_variable
}.call "A"
```
<!-- .element: class="fragment" -->

---

## ナンパラなにもわからない

---


### まとめ
- - -

* ナンパラはめっちゃ便利                      <!-- .element: class="fragment" -->
* 普通に使う分には問題ない                    <!-- .element: class="fragment" -->
* ブロックが配列を受け取る場合に _1 _2 が何になるかを意識する必要はある    <!-- .element: class="fragment" -->
* 普通に使う分には問題ない!!!                 <!-- .element: class="fragment" -->
* 変な使い方をするとハマる                   <!-- .element: class="fragment" -->
  * eval とか binding.local_variable_get とか使えばメタ的に取得できなくもないが…
* 変な使い方があれば教えて下さい！                 <!-- .element: class="fragment" -->

---

## ご清聴
## ありがとうございました
