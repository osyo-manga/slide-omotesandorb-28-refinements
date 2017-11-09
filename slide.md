# 表参道.rb #28 ~gem~
- - -
## Refinements を使おう

---

## 自己紹介
- - -

* なまえ  : おしょー
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)
* 言語    : C++ / Ruby
* エディタ: Vim

---

## [自作 gem](https://rubygems.org/profiles/osyo-manga)
- - -

* [iolite](https://github.com/osyo-manga/gem-iolite)
  * メソッドを遅延呼び出しするための gem
* [use_arguments](https://github.com/osyo-manga/gem-use_arguments)
  * ブロック引数を省略出来る gem
* [unmixer](https://github.com/osyo-manga/gem-unmixer)
  * mixin したモジュールを削除する gem
* [proc-unbind](https://github.com/osyo-manga/gem-proc-unbind)
  * `Proc` から `UnboundMethod` を定義する

---

# 今日のテーマ

---

# gem

---

## gem といえば
- - -
<ul>
<li class="fragment visible" data-fragment-index="0">CLI ツール                  <!-- --><ul>
<li>bundler, middleman</li>
</ul>
</li>
<li class="fragment visible" data-fragment-index="1">フレームワーク              <!-- --><ul>
<li>rails, sinatra</li>
</ul>
</li>
<div class="fragment visible" data-fragment-index="2">
<li class="fragment highlight-red">ライブラリ                 <!-- -->   <!-- --><ul>
<li>pattern-match, lambda_driver</li>
</ul>
</li>
</div>
</ul>

---

# Ruby でライブラリといえば

---

# クラス拡張!

---

#### クラス拡張とは
- - -
クラス拡張とは定義済みのクラスに対して新しいメソッドや既存のメソッドを書き換えること。 

```ruby
# String をクラス拡張する例
class String
	def twice
		self + self
	end
end

p "homu".twice # => "homuhomu"
```

* 既存のクラスに対して新しいメソッドを定義する  <!-- .element: class="fragment" data-code-focus="3-5" -->
* 定義したメソッドが呼べる <!-- .element: class="fragment" data-code-focus="8" -->

---

# クラス拡張の問題点

---

## クラス拡張の問題点
- - -

* 影響するスコープが広すぎる                             <!-- .element: class="fragment" -->
  * 拡張したコードを読み込んでいる全てのコードに影響する   <!-- .element: class="fragment" -->
  * 自分が書いたコード外にまで影響する                     <!-- .element: class="fragment" -->
* 他のコードと競合してしまう                             <!-- .element: class="fragment" -->
  * 他の箇所でも同名のメソッドが拡張されている可能性がある <!-- .element: class="fragment" -->
  * メソッドを定義することで挙動が変わる可能性がある       <!-- .element: class="fragment" -->

---

# 今日話すこと

---

## Refinements を使おう

---

## Refinements とは
- - -
Refinements とはクラス拡張が影響する範囲を限定するための機能です。  
Ruby 2.0 で実験的に導入され、Ruby 2.1 で正式に導入されました。

---

# 早速使ってみよう

---

## Refinements 化する

↓ ↓ ↓
>>>

#### 元のコード
- - -

```ruby
class String
	def twice
		self + self
	end
end

```

>>>

#### Refinements で実装したコード
- - -

```ruby
module StringTwice
	refine String do
		def twice
			self + self
		end
	end
end
```

1. 基点となるモジュールを定義する <!-- .element: class="fragment" data-code-focus="1" -->
2. refine に拡張するクラスを渡す <!-- .element: class="fragment" data-code-focus="2" -->
3. 新しいメソッドを定義する <!-- .element: class="fragment" data-code-focus="3-5" -->


---

## Refinements 呼び出す

↓ ↓ ↓
>>>

#### Refinements を使う(トップレベル)
- - -

```ruby
module StringTwice
	refine String do
		def twice
			self + self
		end
	end
end

# Module#refine で定義しただけではメソッド呼び出せない
# Error: undefined method `twice' for "homu":String (NoMethodError)
"homu".twice

# using にモジュールを渡すことで、
# 定義した refine がそのファイル全体で使えるようになる
using StringTwice

p "homu".twice   # => "homuhomu"
```


<span class="code-presenting-annotation fragment current-only" data-code-focus="11"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="15-17"></span>

* ポイント：影響するのはそのファイルだけで他のファイルには影響しない  <!-- .element: class="fragment" -->


>>>

#### Refinements を使う(クラス内)
- - -

```ruby
module StringTwice
    # ...
end

class X
	# クラス内で using する
	using StringTwice

	def func x
        # そのクラス内でメソッドが呼び出せる
		x.twice + x.twice
	end
end

p X.new.func "homu"
# => "homuhomuhomuhomu"


# この場合トップレベルでは使えない
"homu".twice   # Error
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="7"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="11,15"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="20"></span>

---

### gem で Refinements を使う時の小技

---

## 要求
- - -
クラス拡張を行う gem をつくたい場合、 Refinements 化したクラス拡張と通常のクラス拡張の両方実装したい   <!-- .element: class="fragment" -->

* require "string/twice" した場合は using StringTwice で利用する             <!-- .element: class="fragment" -->
* require "string/twice/core_ext" した場合は using を行わないで利用する             <!-- .element: class="fragment" -->
* これらをユーザ側で使い分けたい              <!-- .element: class="fragment" -->

↓ ↓ ↓
>>>

#### 実装イメージ
- - -

```ruby
# "string/twice.rb"
module StringTwice
	refine String do
		def twice
			self + self
		end
	end
end
```

- - -

```ruby
# "string/twice/core_ext.rb"
class String
	def twice
		self + self
	end
end
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="4-6,11-13">コードが重複している！！！</span>


---

## 解決策

---

## 実装をモジュールに切りだそう

↓ ↓ ↓
>>>

#### 修正前
- - -

```ruby
# "string/twice.rb"
module StringTwice
	refine String do
		def twice
			self + self
		end
	end
end
```

- - -

```ruby
# "string/twice/core_ext.rb"
class String
	def twice
		self + self
	end
end
```

>>>

#### 修正後
- - -

```ruby
# "string/twice.rb"
module StringTwice
	def twice
		self + self
	end

	refine String do
		include StringTwice
	end
end
```

- - -

```ruby
# "string/twice/core_ext.rb"
require "string/twice"

class String
	include StringTwice
end
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="3-5"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="8,15"></span>


>>>

## これにより
- - -

* require "string/twice" した場合は using StringTwice で利用する             <!-- .element: class="fragment" -->
* require "string/twice/core_ext" した場合は using を行わないで利用する             <!-- .element: class="fragment" -->
* include StringTwice で任意のクラスに mixin することも出来る           <!-- .element: class="fragment" style="color: #ff2222" -->
* ただし、この実装でもいくつか問題点があるのでベストではない…    <!-- .element: class="fragment" -->
  * この問題を解説するには時間が足りないのでまた別の機会に…


---

## Refinements 注意点
- - -
* refine 内で定義された特異メソッドは反映されない          <!-- .element: class="fragment" -->
  * 特異クラスに対して refine する必要がある     <!-- .element: class="fragment" -->
* モジュールオブジェクトに対して refine できない    <!-- .element: class="fragment" -->
  * → Ruby 2.4 で対応された                         <!-- .element: class="fragment" -->

↓ ↓ ↓
>>>

## Refinements 注意点 2
- - -

* Object#send で呼び出せない                <!-- .element: class="fragment" -->
  * "homu".send :twice
  * → Ruby 2.4 で対応された                <!-- .element: class="fragment" -->
* #to_s が式展開("#{hoge}")で呼び出されない        <!-- .element: class="fragment" -->
  * → Ruby 2.5 で対応予定        <!-- .element: class="fragment" -->
* #respond_to? は false          <!-- .element: class="fragment" -->
  * "homu".respond_to? :twice # => false
* などなど、基本的に refine で定義したメソッドは Ruby 内部から呼び出されないことが多いです        <!-- .element: class="fragment" -->

---

## まとめ
- - -

* クラス拡張は影響する範囲が大きい   <!-- .element: class="fragment" -->
* クラス拡張を行う際は Refinements を使用して範囲を明示化しよう   <!-- .element: class="fragment" -->
* 基本的には using したスコープでのみ、そのメソッドが呼べる、と認識しておいたほうがよい   <!-- .element: class="fragment" -->

---

## 参照リンク
- - -

* [RubyのRefinement（翻訳: 公式ドキュメントより）](https://techracho.bpsinc.jp/hachi8833/2017_03_23/37464)
* [サンプルコードでわかる！Ruby 2.4の新機能と変更点 - Qiita](https://qiita.com/jnchito/items/9f9d45581816f121af07#refinements%E3%81%AB%E9%96%A2%E9%80%A3%E3%81%99%E3%82%8B%E6%96%B0%E6%A9%9F%E8%83%BD)

---

## 宣伝
- - -
各 Advent Calendar の参加者募集中です。

* [Ruby Advent Calendar 2017](https://qiita.com/advent-calendar/2017/ruby)
* [Ruby on Rails Advent Calendar 2017](https://qiita.com/advent-calendar/2017/ruby_on_rails)
* [初心者C++er Advent Calendar 2017](https://qiita.com/advent-calendar/2017/syoshinsya-cpper)


---

## ご清聴
## ありがとうございました


