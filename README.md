## [GraphQL](https://graphql.org)

### GraphQLって何それ。おいしいの？

「GraphQLは、APIのクエリ言語」

・API：クライアントがサーバーからデータを持ってくる方法を定義するもの

例）飲食店

1. お客：注文をする（トマトパスタ１つ）

2. ウェイター：厨房にトマトパスタの注文が入ったことを伝える

3. 厨房：トマトパスタを作ってホールの人に運んでもらう

4. ウェイター：お客さんへパスタを運ぶ（提供完了）

ここで言うお客さんをWebの世界では**クライアント**、ウェイターを**API**、厨房を**サーバー**と表現できる

＝ "APIはクライアントとサーバーの仲介役である"

・クエリ：問い合わせ（＝トマトパスタの注文）

**結論**：

『 GraphQLは、クライアント（お客さん）がAPI（ウェイター）に注文して、サーバー（厨房から）情報を提供してもらうためのお問い合わせ言語 』

### GraphQLとREST APIのAPIの叩き方

REAT APIはエンドポイント１個１個にリクエストを送らなければいけないのに対して、

GraphQLを使ってサーバーを設定すれば、欲しいリクエストを集約できる（＝自分の欲しいものだけを取ってくることが出来る）
```
// GraphQL

// リクエスト：必要な情報のみ含めて叩く
query {
    User(id: "er3tg439frjw") {
        name
        posts {
            title
        }
        followers(last: 3) {
            name
        }
    }
}

// レスポンス
{
    "data": {
        "User": {
            "name": "Mary",
            "posts": [
                { title: "Learn GraphQL today" }
            ],
            "followers": [
                { name: "John" },
                { name: "Alice" },
                { name: "Sarah" },
            ]
        }
    }
}
```
```
// REST API

// REST APIでは、サーバー側にエンドポイント（URL）が設定されている （例1： /users/<id>）
// そして、そのURLにアクセスするとデータが返ってくる
{
    "user": {
        "id": "er3tg439frjw"
        "name": "Mary",
        "address": { ... },
        "birthday": "July 26, 1982"
    }
}

// （例2： /users/<id>/posts）
{
    "posts": [{
        "id": "ncwon3ce89hs"
        "title": "Learn GraphQL today",
        "content": "Lorem ipsum ...",
        "comments": [ ... ],
    }]
}
```
**メリット**

1. １つのエンドポイントで欲しいデータが全て取得できる（アンダーフェッチ）

2. 必要としないデータを取得せずに済む（オーバーフェッチ）

3. 必要最低限のデータ量に抑えられるので、データロードの効率性が改善される

4. 型を定義することで、送信されるデータが明確化される（スキーマ）
