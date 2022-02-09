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
            font-size: 42px;
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
  - 領域のこと。
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
##### ドメインサービスはまた次回？
---
# 第二節(各パターンの紹介) 目次

   1. <知識を表現するパターン>
      1. 値オブジェクト
      2. エンティティ
      3. <ドメインサービス>
   2. アプリケーションを実現するためのパターン
      1. リポジトリ
      2. アプリケーションサービス
      3. ファクトリ
   3. 知識を表現する、より発展的なパターン
      1. 集約
      2. 仕様
---
# ドメインサービスとは

##### 値オブジェクトやエンティティに記述するのは不自然なふるまいを記述するサービス
---
# 例えば...

##### User オブジェクトに対して、ある user と同姓同名の人物が存在するか否かのチェックをしたい場合

```
class User
  def initialize(fst_n, lst_n) end;

  def exists?
    (DB に存在したら) ? true : false
  end
end
```
---
# 例えば...

##### User オブジェクトに対して、ある user と同姓同名の人物が存在するか否かのチェックをしたい場合

```
user = User.new(lst_n: '田中', fst_n: '太郎')
user.exists?
=> true
```
##### そもそも具体化された個々のオブジェクトに問い合わせるものなのか？？
---
# 例えば...

```
user = User.new(lst_n: '田中', fst_n: '太郎')

user.lst_n
=> '太郎'
// user 自身に問い合わせるべき

user.exists?
=> true
// user 自身に問い合わせるべきではない

```
---
# service クラスの導入

```
class UserService
  self.exists?(user)
    (DB に存在すれば) ? true : false
  end
end
-----------------------------------
user = User.new(lst_n: '田中', fst_n: '太郎')

UserService.exists?(user)
=> true
```
---
# ドメインサービスの濫用はよくない

```
class UserService
  def eql?(user1, user2)
    user1.lst_n == user2.lst_n && user1.fst_n && user2.fst_n
  end
end
-----
user1 = User.new(lst_n: '田中', fst_n: '太郎')
user2 = User.new(lst_n: '田中', fst_n: '太郎')

UserService.eql?(user1, user2)
=> true
```
---
# 結果、値ｵﾌﾞｼﾞｪｸﾄやｴﾝﾃｨﾃｨが有名無実化する
##### ドメインサービス濫用のデメリット

- ロジックが点在化し、保守が難しくなる
- システム特有の「ふるまい」が分かりづらくなる
---
# ドメインサービスを利用する際は…

- オブジェクト自身に問い合わせることが不自然な処理のとき
- 複数のクラスを横断するような処理のとき

##### ※ ただし、あくまで値オブジェクトやエンティティでの実装を優先して検討すべき

---
# 例
---
# 第二節(各パターンの紹介) 目次

   1. 知識を表現するパターン
      1. 値オブジェクト
      2. エンティティ
      3. ドメインサービス
   2. <アプリケーションを実現するためのパターン>
      1. <リポジトリ>
      2. アプリケーションサービス
      3. ファクトリ
   3. 知識を表現する、より発展的なパターン
      1. 集約
      2. 仕様
---
# リポジトリとは？

##### データを永続化し再構築するといった処理を抽象的に扱うためのオブジェクト

###### Rails でいう ActiveRecord のこと？
---
# リポジトリがないと…
###### 内容把握のためには、Ruby だけでなく SQL も読み込まなければならない

```
class User < ActiveRecord
  def save
    return false lst_n.length > 2
    return false fst_n.length > 2

    ActiveRecord::Base.connection.execute(
       'INSERT INTO users VALUES `lst_n`, `fst_n`';
    )
  end
end
```
---
# データの永続化処理は分離する（例）
###### ActiveRecord の `save` はあえて使わず…

```
class UserRepository < ActiveRecord
  def initialize end;

  def save
    ActiveRecord::Base.connection.execute(
       'INSERT INTO users VALUES `@lst_n`, `@fst_n`';
    )
  end
end
```
---
# データの永続化処理は分離する（例）
###### ActiveRecord の `save` はあえて使わず…

```
class User
  def save
    return false lst_n.length > 2
    return false fst_n.length > 2

    UserRepository.new(lst_n, fst_n).save
  end
end
```
---
# リポジトリのメリット
##### リポジトリ側・ドメイン側双方のコードの保守性があがる
1. インフラ側の要件が決まっていなくとも実装を開始できる
2. ドメイン側のコードが簡潔になる
   1. SQLもドメイン側に記載すると、コードの大半がSQL文で埋め尽くされるため
---
# 第二節(各パターンの紹介) 目次

   1. 知識を表現するパターン
      1. 値オブジェクト
      2. エンティティ
      3. ドメインサービス
   2. <アプリケーションを実現するためのパターン>
      1. リポジトリ
      2. <アプリケーションサービス>
      3. ファクトリ
   3. 知識を表現する、より発展的なパターン
      1. 集約
      2. 仕様
---
# アプリケーションサービスとは？

##### ユースケースを実現するもの
##### ex) Create, Read, Update, Delete etc..
###### 無理やり Rails で当てはめるなら、controller ?
---
# ドメインサービスとの差異
##### そもそもレイヤーが異なる

- ドメインサービス
  - エンティティや値オブジェクトに書くべきでないふるまいを実装する
- アプリケーションサービス
  - ユーザのシナリオを一対一で表現したもの
---
# アプリケーションサービス（例）

```
class UserApplicationService
  def register(fst_n, lst_n)
    user = User.new(fst_n, lst_n) // エンティティ
    // ドメインサービス
    unless UserService.new(user).exists?
      // リポジトリ
      UserRepository.new(lst_n, fst_n).save
    else
      raise '存在してます'
    end
  end
end
```
---
# 注意点: ドメインルールの流出を防ぐ
###### コードが散逸し変更に弱くなるため
```
class UserApplicationService
  def register(fst_n, lst_n)
    raise 'ValidationError' unless fst_n || lst_n
    UserRepository.new.save(fst_n, lst_n)
  end

  def update(fst_n, lst_n)
    raise 'ValidationError' unless fst_n || lst_n
    UserRepository.new.update(fst_n, lst_n)
  end
end
```
---
# 今日はここまで
##### ファクトリはまた次回？
---
# おまけ1
![bg 60%](https://i.imgur.com/PC7Cg2G.jpg)

---
# おまけ2
![bg 60%](https://i.imgur.com/3zUiYRD.jpg)

---
# 第二節(各パターンの紹介) 目次

   1. 知識を表現するパターン
      1. 値オブジェクト
      2. エンティティ
      3. ドメインサービス
   2. <アプリケーションを実現するためのパターン>
      1. リポジトリ
      2. アプリケーションサービス
      3. <ファクトリ>
   3. 知識を表現する、より発展的なパターン
      1. 集約
      2. 仕様
---
# ファクトリとは

#### オブジェクトを生成する責務をもったプログラム要素
---
# ファクトリに書くべきオブジェクト生成の例
```
# シナリオ層が複雑化してしまう
ticket = Ticket.new
ticket.message = Message.new
ticket.message.mail = Mail.new
ticket.message.mail.send!

-----

# オブジェクト生成を切り分けて、呼び出すだけ！
mail = CreateMailFactory.new
mail.send!
```
---
# シナリオ層でオブジェクト構築すると

#### ドメインの内部情報をシナリオ層が知りすぎてしまい、シナリオを組み立てる責務であるべきシナリオ層が複雑化する
---
# ファクトリに書くべきオブジェクト生成の例
```
# ドメインが複雑化する！
class User
  def initialize(old)
    @old = old
  end

  def adlut_users_with_no_children
    User.where(old >= ?, '20').xxx
  end
end
```
---
# ドメイン層でオブジェクト構築すると

#### ドメイン内で複雑なオブジェクトの生成をすると、領域・振る舞いの定義という本来の役目をはみ出して、理解しづらい定義となってしまう
---
# AngularJS における Factory とは？

| | Service | Factory |
| --- | ------ | ------- |
|| アプリ内で1つしかインスタンスが生成されない、シングルトン | 呼び出しごとに初期化される |


##### 参考: https://github.com/angular/angular.js/blob/v1.2.0-rc.3/src/auto/injector.js#L620
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
   3. <知識を表現する、より発展的なパターン>
      1. <集約>
      2. 仕様
---
# 集約とは

##### 外部から内部のオブジェクトに対して直接操作するのではなく、そのオブジェクトを保持するオブジェクトに依頼する形で操作すること

#### => 要は、デメテルの法則のこと
---
# 集約のサンプルコード1

```
user = User.new(fst_n = 'Taro', lst_n='Tanaka')

user.fst_n = '次郎'
// => ダメな例

user.change(:fst_n, '次郎')
// => 良い例
```
##### user ドメイン内で定義された振る舞い（文字数制限・重複チェック等）を行えるから
---
# 集約のサンプルコード2

```
# group has_many users

group = Group.find_by(id: 1)
group.users.first.lst_n = 'Suzuki'
// => ダメな例

group.change_lst_n(user_id: 1, lst_n: 'Suzuki')
// => 良い例
```

##### Aggregate Root を通して内部のオブジェクトを操作すべき！
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
   3. <知識を表現する、より発展的なパターン>
      1. 集約
      2. <仕様>
---
# 仕様とは
#### あるオブジェクトが何らかの基準を満たしているかどうかを判定するもの

##### `is_xxx?` のような判定メソッドのこと
---
# わざわざ判定メソッドのみの層を作る理由

1. ドメイン層に書くべき領域定義とは似て非なる
2. 似たようなメソッド(名)が乱立して識別しづらくなる
---

# ドメインに書いてしまうと…


```
class User
  def is_adult?
    self.old >= 20
  end

  def is_old?
    seld.old >= 60
  end
end
```
---
# 判定メソッド層を導入した例(定義元)


```
class UserOldSpecification
  def initialize; end

  def is_adult?
    @user.old >= 20
  end

  def is_elder?
    @user.old >= 60
  end
end
```
---
# 判定メソッド層を導入した例(呼び出し側)


```
user_spec = UserOldSpecification.new(User.first)

unless user_spec.is_adult?
  raise '子どもはダメです'
end

fee = user_spec.is_elder? ? 1000 : 500

```
---
# Rails では使いみちなさそう？

##### Rails の Model は、ドメイン定義の役割のみを担ってるわけではなく、判定メソッドを Model に書いても不自然でないため
---
以上です！
---