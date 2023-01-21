---
title: 'API Gateway をトリガーに AWS Lambda を動かしてみる'
emoji: '🐥'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['lambda', 'aws', 'python']
published: true
---

こんにちは 👋

仕事で AWS Lambda（以下、Lambda） を使うことになったので Lambda の概要理解から API Gateway をトリガーに Python で書かれた簡単な関数を実行するところまでをやってみたいと思います。

# Lambda とは？

何はともあれ、まずは公式サイトを見てみましょう。

https://aws.amazon.com/jp/lambda/

なるほど、どうやら Lambda を使うと自前でサーバーを用意することなく何らかのイベントをトリガーに、自分があらかじめ Lambda に設定しておいた処理を実行できるようです。これは世に言う「サーバーレス」というもので、前述のとおり開発側は自前でサーバーを用意する必要がないため、構築や保守・運用のコストを削減することができます。さらに、Lambda は従量課金制なので実行時間に応じた金額しか請求されない点も魅力です。

# Lambda で処理を動かしてみる

それでは、実際に Lambda を使ってみたいと思います。今回は AWS の API Gateway をトリガーに「JSON ファイルに書かれている本の情報リストからランダムに 1 冊の本の情報を返す」という処理を Python で行いたいと思います。

## 1.Lambda に関数を作成

まずは Lambda で実行される関数を Python で定義していきたいと思います。

:::message
AWS のアカウント作成は省略します。アカウント作成方法は以下が参考になるかなと思います。
[AWS アカウント作成の流れ](https://aws.amazon.com/jp/register-flow/)
:::

まずは AWS のコンソールにある検索欄に「Lambda」と入力し、検索結果にある「Lambda」をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/50e1d30ac3ba-20230121.png)

「関数の作成」をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/54f0740e2922-20230121.png)

「一から作成」を選択。関数名には任意の関数名を記入し、ランタイムには今回 Python を使いたいので「Python3.9」を選択し、「関数の作成」をクリック。

![](https://storage.googleapis.com/zenn-user-upload/63c3743352d4-20230121.png)

以下のような画面が出てきたら Lambda に関数を作る部分まで完了です。

![](https://storage.googleapis.com/zenn-user-upload/47ff15e0faaa-20230121.png)

## 2.Lambda の関数内で実行する処理の追加

次は実際に行われる処理（JSON ファイルに書かれている本の情報リストからランダムに 1 冊の本の情報を返す）の実装をしたいと思います。

```python:get_book_data.py
import json
import random

json_open = open('books.json', 'r')
json_load = json.load(json_open)
book_data = json_load['data']


def lambda_handler(event, context):
    """Bookデータからランダムに1冊のデータを返す"""
    book = random.choice(book_data)

    return {
        'statusCode': 200,
        'body': json.dumps({'book': book}),
    }

```

:::details 読み込む JSON ファイルはこちら

```json:books.json
{
  "data": [
    {
      "title": "C#クックブック ―プロフェッショナル開発者のためのモダンレシピ",
      "author": "Joe Mayo",
      "price": 3960
    },
    {
      "title": "詳解 システム・パフォーマンス 第2版",
      "author": "Brendan Gregg",
      "price": 6600
    },
    {
      "title": "オブザーバビリティ・エンジニアリング",
      "author": "Charity Majors、Liz Fong-Jones、George Miranda",
      "price": 3960
    },
    {
      "title": "リーダーの作法",
      "author": "Michael Lopp",
      "price": 1936
    },
    {
      "title": "UXデザインの法則",
      "author": "Jon Yablonski",
      "price": 1584
    },
    {
      "title": "入門 Python 3 第2版",
      "author": "Bill Lubanovic",
      "price": 3344
    },
    {
      "title": "マスタリングLinuxシェルスクリプト 第2版",
      "author": "Mokhtar Ebrahim、Andrew Mallett",
      "price": 2640
    },
    {
      "title": "Lean UX 第3版",
      "author": "Jeff Gothelf、Josh Seiden",
      "price": 2376
    },
    {
      "title": "UX戦略 第2版",
      "author": "Jaime Levy",
      "price": 2640
    },
    {
      "title": "戦略的UXライティング",
      "author": "Torrey Podmajersky",
      "price": 2024
    }
  ]
}
```

:::

以上が Lambda 内で実行させたい処理の内容になります。より具体的に説明すると、Lambda では`get_book_data.py`内で定義されている`lambda_handler()`が実行されます。

さて、ファイルの準備ができたので、作成したファイルたちを`.zip`ファイルにして Lambda にアップロードしたいと思います。

「アップロード元」から「.zip ファイル」をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/69f021327c6d-20230121.png)

「アップロード」からアップロードしたい`.zip`ファイルを選択した後、「保存」をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/0febe158627c-20230121.png)

アップロードに成功すると以下のように`get_book_data`フォルダの中にアップロードした`.zip`ファイルの中身が解凍され、それぞれのファイルが格納されていると思います。

![](https://storage.googleapis.com/zenn-user-upload/8aabb1d73fdf-20230121.png)

それでは次にランタイムの設定を編集し、アップロードした`get_book_data.py`の`lambda_handler()`が実行されるようにします。ページ下部にあるランタイムの設定から「編集」をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/eb87f60058bf-20230121.png)

「ハンドラ」を`ファイル名.lambda_handler`の形に修正します。今回は`get_book_data.py`の`lambda_handler()`を実行したいので、`get_book_data.lambda_handler`に修正し、保存をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/d453fc870402-20230121.png)

これでやるべきことは一通り完了なのですが、せっかくなので、処理が正常に動くかテストをしたいと思います。「Test」から「Configure test event」をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/1faf4b527fa9-20230121.png)

今回は特にパラメータを渡したりする必要はないので、「イベント名」にテストの内容がわかるような名前を入力して後は何も変更せずに「保存」をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/c2c771b2fcfe-20230121.png)

「Test」をクリックすると以下のようにちゃんと期待される処理が行われていることが確認できます。

![](https://storage.googleapis.com/zenn-user-upload/8556e499d4af-20230121.png)

## 3.API Gateway の設定

それでは次に API Gateway の設定を行い、エンドポイントから処理を実行できるようにしたいと思います。

まず、「トリガーを追加」をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/fef952690612-20230121.png)

検索欄から「API Gateway」を選択します。

![](https://storage.googleapis.com/zenn-user-upload/881bf5d13bf4-20230121.png)

今回はお試しなので、以下のように設定して「追加」をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/c8b9e5e53112-20230121.png)

トリガーの追加に成功すると、以下のような画面に遷移し、「API endpoint」が表示されていると思います。

![](https://storage.googleapis.com/zenn-user-upload/0a82e2cadba6-20230121.png)

Postman から API を叩いてみると、しっかりレスポンスが返ってきました！

![](https://storage.googleapis.com/zenn-user-upload/2d4c6e6c0079-20230121.png)

さて、画像ベースで説明してきましたが、なんとなく Lambda がどんなものか理解できたのではないでしょうか。

AWS には数多くのサービスがあるので、他のサービスと組み合わせて色々なことができそうですね。

# 補足

ここまで駆け足で説明してきましたが、個人的に気になった部分などについて少し補足を。

## `event`と`context`とは？

`get_book_data.py`の`lambda_handler()`に渡されている`event`と`handler`の中身が気になったので調べてみました。

https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-concepts.html#gettingstarted-concepts-event

上記によると、`event`の中には API Gateway などのトリガーから渡されたデータが JSON 形式で入っているようです。例えば API Gateway を例にすると、HTTP メソッドやパスパラメータなどが渡されることになります。

次に`context`についてですが、こちらには「実行関数に関する情報」が入っているようです。詳細は以下のページが非常に分かりやすいので、気になる方はぜひ。

https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/python-context.html

## 対応言語は?

公式によると以下の通りです。

> Q: AWS Lambda がサポートする言語は何ですか?
>
> AWS Lambda は、ネイティブでは、Java、Go、PowerShell、Node.js、C#、Python、Ruby のコードをサポートしています。また、関数の作成にその他のプログラミング言語を使用できるようにするための Runtime API を提供しています。Node.js、Python、Java、Ruby、C#、Go、PowerShell の使用に関するドキュメントをご覧ください。

引用元：[AWS Lambda よくある質問](https://aws.amazon.com/jp/lambda/faqs/)

---

以上で「API Gateway をトリガーに AWS Lambda を動かしてみる」の記事を終わりにしたいと思います。

最後までお読みいただきありがとうございました。
