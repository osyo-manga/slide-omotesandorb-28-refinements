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
クラス拡張とは定義済みのクラスに対して新しいメソッドや既存のメソッドを書き換えることです。 

```ruby
# String をクラス拡張する例
# String#twice メソッドを新たに追加する
class String
	def twice
		self + self
	end
end

# String のインスタンスオブジェクトに対して
# 定義したメソッドが呼べる
p "homu".twice # => "homuhomu"
```

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
## ライブコーディング

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
# まず、module を定義する
module Ex
    # Module#refine を使用して任意のクラスに対してメソッドを定義する
	refine String do
		# 通常と同じようにメソッドを定義する
		def twice
			self + self
		end
	end
end
```

---

## Refinements 呼び出す
## ライブコーディング

↓ ↓ ↓

>>>

#### Refinements を使う(トップレベル)
- - -

```ruby
module Ex
	refine String do
		def twice
			self + self
		end
	end
end

# Module#refine で定義しただけでは反映されない
# Error: undefined method `twice' for "homu":String (NoMethodError)
"homu".twice


# using に拡張するモジュール渡すことでそのファイル全体で使えるようになります
using Ex

p "homu".twice   # => "homuhomu"
```

>>>

#### Refinements を使う(クラス内)
- - -

```ruby
module Ex
    # ...
end

class X
	# このクラス内で String#twice を呼び出すことが出来る
	using Ex

	def func x
		x.twice + x.twice
	end
end

p X.new.func "homu"
# => "homuhomuhomuhomu"


# この場合トップレベルでは使えない
"homu".twice   # Error
```

---

## 注意点
- - -
* Module オブジェクトに対して refine できない    <!-- .element: class="fragment" -->
  * → Ruby 2.4 で対応された                         <!-- .element: class="fragment" -->
* Object#send で呼び出せない                <!-- .element: class="fragment" -->
  * "homu".send :twice                <!-- .element: class="fragment" -->
  * → Ruby 2.4 で対応された                <!-- .element: class="fragment" -->

>>>

## 注意点 2
- - -
* #respond_to? は false          <!-- .element: class="fragment" -->
  * "homu".respond_to? :twice # => false
* 特異メソッドは反映されない          <!-- .element: class="fragment" -->
  * 特異クラスに対して refine する必要がある

---

## まとめ
- - -

* クラス拡張は影響する範囲が大きい
* クラス拡張を行う際は Refinements を使用して範囲を明示化しよう
* 基本的には using したスコープでのみ、そのメソッドが呼べる、と認識しておいたほうがよい

---

## 参照リンク
- - -

* [RubyのRefinement（翻訳: 公式ドキュメントより）](https://techracho.bpsinc.jp/hachi8833/2017_03_23/37464)

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


