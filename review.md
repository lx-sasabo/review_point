# レビュー前にチェックしておきたいこと

## 全般
- linter, formatterは通したか
- typoはないか（Code Spell Checkerなどスペルチェッカーを使う）
- 不要なコメントアウトは残っていないか
- 不要になった変数が残っていないか
- binding pryなどのデバッグツールで確認用のコードが残ったままになっていないか
- メソッドや変数名は適切か<br>
  参考: https://qiita.com/Ted-HM/items/7dde25dcffae4cdc7923
- コードだけでは読み取れない重要な内容はコメントに残したか
- 単一責任の原則にのっとっているか
  参考: https://note.com/erukiti/n/n67b323d1f7c5

## Rails
### responseはActionController において特殊なオブジェクトとして存在するので変数名に使わない
参考: https://railsguides.jp/action_controller_overview.html#response%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88

### クエリストリングスの生成はHash#to_queryを使う
参考: https://railsguides.jp/active_support_core_extensions.html#to-query<br>
参考: https://qiita.com/QUANON/items/4a0a06e0ed76b9a07bc7


## JavaScript

### 早期リターンができないか検討する
理由: 早期リターンのほうが分かりやすい場合が多いため<br>
ただし場合によるので注意

### 分割代入を使う
理由：シンプルに書ける<br>
サーバー側からのレスポンスではスネークケースなのを
フロント側ではキャメルケースにしたいときなんかにも便利に使える

```js
const response = { id: 1, user_name: 'yamashita' , deleted_user: true, body: 'hello' }
const { user_name: userName, deleted_user: deletedUser } = response
console.log(userName, deletedUser) // yamashita true

```

###  GETリクエストでサーバーから情報をとってくる場合getよりfetchのほうが良い
理由: APIへのアクセスは重い処理のため。getは軽い処理のときに使う。<br>
背景: Fetch APIができてからはfetchの方が使われるようになっている。以前は、jQuery.getとかjQuery.postみたいに送信するHTTPリクエスト メソッド名に対応した名前をつけることが多かった。<br>
参考: https://qiita.com/Ted-HM/items/7dde25dcffae4cdc7923#%E4%BD%BF%E3%81%A3%E3%81%A6%E3%81%AF%E3%81%84%E3%81%91%E3%81%AA%E3%81%84%E8%A8%80%E8%91%89
参考: https://developer.mozilla.org/ja/docs/Web/API/Fetch_API
参考: https://api.jquery.com/category/ajax/shorthand-methods/


## html

### table-cellは安易に使わない
理由: レスポンシブレイアウトでの自由さがないなどの特性が色々あるため<br>
参考: https://qiita.com/sawadays0118/items/4c329fd05cdff14ffebc


## React
### keyにindexを使わない
理由: 要素を一意に識別できないため<br>
参考: https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-array-index-key.md


## テストについて
### テストの期待値ははなるべくベタ書きする
理由: DRYにすると依存関係や実行順を追うのが後から大変になりテストの修正コストが上がる<br>
参考: https://qiita.com/jnchito/items/eb3cfa9f7db752dcb796

### Rspecのdescribeにメソッド名を記載するときはインスタンスメソッドには頭に＃をつける、クラスメソッドは.をつける
理由: 慣例。ただしプロジェクトで別途ルールがある場合はプロジェクトごとのルールに従って下さい。<br>
参考: https://www.betterspecs.org/

### letとlet!の違いを理解する
`let`: 定義した定数が初めて使われたときに評価される遅延評価<br>
`let!`: 各テストのブロック実行前に定義した定数を作る<br>
参考: https://qiita.com/jnchito/items/42193d066bd61c740612<br>
一概にどちらを使うほうが良いということはないのでプロジェクトのルールや状況に応じて使い分ける<br>
let!派の意見↓<br>
参考: https://github.com/willnet/rspec-style-guide#let%E3%81%A8let%E3%81%AE%E4%BD%BF%E3%81%84%E5%88%86%E3%81%91

### beforeやlet!は必要な場所の近くに書く
理由: スコープが大きくなりがちでネストの中でオーバーライドできてしまうため、メンテコストがあがる<br>
contextに分けているテストの条件をlet!で書くと、let!の内容がテストの焦点となる条件なのかが明示できる

```ruby
context 'タイムアウトしていないとき' do
  let!(:within_limit) { 299 }

  example 'trueを返す' do
    hoge = hoge.build(locked_at: Time.zone.now.ago(within_limit))
    hoge.save!
    expect(hoge.islock).to be_truthy
  end
end

context 'タイムアウトしたとき' do
  let!(:over_limit) { 300 }

  example 'falseを返す' do
    hoge = hoge.build(locked_at: Time.zone.now.ago(over_limit))
    hoge.save!
    expect(hoge.islock).to be_falsey
  end
end
```

### テストのサンプルデータは実際に使われる値に近い値にする
理由：実際に生成されるような値を書いておくとテストを読んだときに仕様がわかりやすいため

```ruby
# NG
random_number = '123abc'

# OK
random_number = '8bb403c74f973edaac79f646998330a9'
```

### 処理の前後で値を比べるときに同じインスタンスを参照していないか確認する
理由: 実は同じインスタンスを比べてししまっていて意図しないテストになっていることがあるため

```ruby
# NG 同じインスタンスを参照しているのでテスト落ちる
hoge = 1
before_hoge = hoge
hoge = 2
expect(hoge).not_to eq before_hoge

# OK cloneをつかう
# ただしcloneは浅いコピーなのでオブジェクトがhashやarrayの場合は深いコピーが必要
hoge = 1
before_hoge = hoge.clone
hoge = 2
expect(hoge).not_to eq before_hoge

```
