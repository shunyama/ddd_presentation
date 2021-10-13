---
marp: true
theme: uncover
footer: "2021 Shunsuke Yamaguchi All Rights Reserved"
paginate: true
style: |
        footer {
           margin-top: 0%;
          text-align: center;
        },
          	
        h1 {
            font-size: 50px;
            left: 5%;
            position: absolute;
            top: 6%;
        },
        	
        section.front h1 {
            left: 50%;
        },

         ol > li {
            font-size: 95%;
         },
        	
        ol > li > ol > li {
            font-size: 75%;
            text-align: left;
            padding: 2%, 0;
            left: 2%;
        },

        h5, h6 {
            text-align: left
        },

        code {
           margin-top: 0;
           line-height: 1.05rem;
        }
---
## 山口と学ぶドメイン駆動設計

---
# 本稿の目的

- ドメイン駆動設計（以下、DDD）について何の知識も持たない山口が、DDDの概念及びパターンをなんとなく理解することを目的とする。

---
# 当発表会の進め方

##### 前頁の目的を達するため、以下の進行方法を採る


1. DDDについての概念や各パターンに関する山口なりの理解を発表
2. その理解への正否、補足事項等があれば、皆様方が都度突っ込む

---
# 目次

1. はじめに
   1. DDDとは？
   2. ドメインとは？ モデリングとは？
   3. DDD の学習の流れ
   4. 参考文献
2. 各パターンの紹介
   1. 知識を表現するパターン
   2. アプリケーションを実現するためのパターン
   3. 知識を表現する、より発展的なパターン

---
# DDDとは？

##### ドメインモデリングによって、ソフトウェアの価値を高めるというアプローチを採るソフトウェア開発の手法
###### (出典: エリック・エヴァンスのドメイン駆動設計)
---
# ドメインとは？ モデリングとは？

- ドメイン: ソフトウェアを適用する対象
  - 領域のこと。「お客様情報」や「メッセージ情報」的な
- モデリング: 問題解決のため、物事の特定の側面を抽象化したもの
  - 領域をシステム向けに抽象化したもの。コードに落とし込むと「customers」や「messages」的な

---
# DDDの学習の流れ

##### ドメインとかモデリングとか抽象的なこと（総論）を言われてもよく分からない
##### よって、各パターン（各論）からボトムアップ式に勉強することで、全体の把握に努める
---
# 参考文献

1. 「エリック・エヴァンスのドメイン駆動設計」(著: エリック・エヴァンス)
2. 「ドメイン駆動設計入門 ボトムアップでわかる! ドメイン駆動設計の基本」(著: 成瀬 允宣)
3. 「パーフェクト Ruby on Rails」(著: すがわらまさのり)
---
# 目次
1. はじめに
   1. DDDとは？
   2. ドメインとは？ モデリングとは？
   3. DDD の学習の流れ
   4. 参考文献
2. <各パターンの紹介>
   1. 知識を表現するパターン
   2. アプリケーションを実現するためのパターン
   3. 知識を表現する、より発展的なパターン
---
# 第二節(各パターンの紹介) 目次

   1. 知識を表現するパターン
      1. 値オブジェクト
      2. エンティティ
      3. ドメインサービス
   2. アプリケーションを実現するためのパターン
      1. リポジトリ
      2. アプリケーションサービス
      3. ファクトリ
   3. 知識を表現する、より発展的なパターン
      1. 集約
      2. 仕様
---
# 値オブジェクトとは？

##### システム固有の値を表現するために定義されたオブジェクトのこと
---
# 例えば…

###### addresses テーブルに「`postal_code: string`」というカラムがあるとする。
###### 人間から見れば、`100-0001` も `1000001` も、`〒100-0001` も同義であるが、プログラム的には異なるものとされてしまう。
```
'1000001' == '〒100-0001'
=> false
```
---
# かといって…

###### `addresses.rb` に同値チェックを実装すると、`addresses.rb` が肥大化し、他モデルでの再利用性にも欠ける。。。
```
class Address < ApplicationRecord
   def postal_code_is_equal?(other)
      // 一文字目の〒マークを除去して〜
      // ハイフンを除去して〜
      // 同値チェック！ 
   end
end
```
---
###### `postal_code` を `string` としてではなく、`object` 化して振る舞いを持たせた上で扱うと…
```
class PostalCode
  def initialize(postal_code)
    @postal_code = postal_code
  end

  def ==(other)
   // 一文字目の〒マークを除去して〜ハイフンを除去して〜
   // 同値チェック！ 
  end
  alias eql? ==
end
```
---
# すると…

###### `addresses.rb` が skinny になり、`postal_code` は他モデルでも再利用可能に！
```
class Address < ApplicationRecord
  
  def not_allow_chiyoda_1
    postal_code = PostalCode.new(self.postal_code)
    raise '千代田区千代田のお客様は対象外です' postal_code.eql?('100-0001')
  end
end
```
---
# 上記例での値オブジェクト導入のメリット

###### 住所録に関する事柄は `addresses` クラスに、郵便番号に関する事柄は `postal_codes` クラスにフォーカスできるので、、

1. `addresses` モデルの肥大化防止
2. `postal_code` を他モデルでも再利用可能に
3. `postal_code` クラスを読むことで、当システムでの郵便番号の仕様把握の容易化
---
# 値オブジェクトとは？（再掲）

##### システム固有の値を表現するために定義されたオブジェクトのこと
---
# 値オブジェクトの特徴
1. 不変である
2. 属性値により区別される
---
# 値オブジェクトの特徴1
##### 不変である
```
pry(main)> pc = PostalCode.new('100-0001')
pry(main)> pc.with_icon // => '〒100-0001'
pry(main)> pc.only_num  // => '1000001'
pry(main)> pc.change_pc // => '560-0043'
```
---
# 値オブジェクトの特徴2
##### 属性値により区別される
```
pry(main)> pc1 = PostalCode.new('100-0001')
pry(main)> pc2 = PostalCode.new('1000001')
pry(main)> pc1 == pc2
=> true
```
---
# 第二節(各パターンの紹介) 目次

   1. <知識を表現するパターン>
      1. 値オブジェクト
      2. <エンティティ>
      3. ドメインサービス
   2. アプリケーションを実現するためのパターン
      1. リポジトリ
      2. アプリケーションサービス
      3. ファクトリ
   3. 知識を表現する、より発展的なパターン
      1. 集約
      2. 仕様
---
# エンティティとは？

##### 属性ではなく、同一性によって識別されるオブジェクトのこと

##### Rails で言うところの、モデルのインスタンスのこと
---
# 例えば…

###### `user1` と `user2` は同じ？異なる？
```
pry(main)> user1 = User.create!(first_name: '太郎', last_name: '田中')
pry(main)> user2 = User.create!(first_name: '太郎', last_name: '田中')
pry(main)> user1 == user2
=> false
```
---
###### どのようにして同値か否かをチェックしているか？
```
pry(main)> user1 = User.create!(first_name: '太郎', last_name: '田中')
pry(main)> user2 = User.create!(first_name: '太郎', last_name: '田中')
pry(main)> user1.id // => 1
pry(main)> user2.id // => 2
```
###### `id` で判別している
https://github.com/rails/rails/tree/main/activemodel/lib/active_model

---
# エンティティの特徴
1. 可変である
2. 同じ属性であっても区別される
3. 同一性により区別される
---
# エンティティの特徴1
##### 可変である
###### ただし、識別子は不可変にすべき
```
pry(main)> user1 = User.create!(first_name: '太郎', last_name: '田中')
pry(main)> user1.last_name = '木村'
pry(main)> user1.last_name
=> '木村'
```
---
# エンティティの特徴2
##### 同じ属性であっても区別される
###### ユニークな識別子をオブジェクトに持たせ
```
pry(main)> user1 = User.create!(first_name: '太郎', last_name: '田中')
pry(main)> user2 = User.create!(first_name: '太郎', last_name: '田中')
pry(main)> user1 == user2
=> false
```
---

# エンティティの特徴3
##### 同一性により区別される
###### 属性が変更されても、同じオブジェクトと判定されるべき
```
pry(main)> user1 = User.create!(first_name:'太郎',last_name:'田中')
=> #<User id: 1, first_name:'太郎',last_name:'田中'>
pry(main)> user1.last_name = '鈴木' // user1.last_name == '鈴木'
pry(main)> user2 = User.find(1)
// user1.last_name=>'鈴木',user2.last_name=>'田中'
pry(main)> user1 == user2
=> true
```
---
# 値オブジェクトとエンティティの比較
| | 値オブジェクト | エンティティ |
| --- | ------ | ------- |
| 等価基準 | 属性値 | 識別子 |
| 可変性 | 不変 | 可変 |
---
# 今日はここまで
##### サービスオブジェクトはまた次回？