## GraphQLのコアコンセプト

### スキーマの定義言語（SDL）

GraphQLには、APIのスキーマを定義するために使う[スキーマ定義言語（SDL)](https://github.com/DaisukeKarasawa/graphql-prc/tree/main/schema-definition-language)という構文がある。

```
// 例）Userという型の定義

type User {
    name: String!
    age: Int!
}
```

スキーマにはタイプとフィールドがあり、フィールド上ではタイプ同士を関連づけることも可能。

```
// 例）ユーザー（User）とそのユーザーが投稿したブログ（Post）を関連づける

type Post {
    title: String!
    author: User!
}

type User {
    name: String!
    age: Int!
    posts: [Post!]!     // User と Post の１対多の関係
}
```

###  クエリでデータを取得する

REST APIでは、特定のエンドポイントに基づいて、定義されたデータが返される。

GraphQLでは、クライアントが必要とするデータを取得できるように、返すデータ構造を固定せずに複数のエンドポイントをまとめて１つのエンドポイントで情報を取得できる柔軟性を持つ。その結果として、クライアントはサーバーに必要な情報を定義したクエリを送信する必要がある。

```
// クライアントが送信できるクエリの例

{
    allUsers {
        name
    }
}
```

このクエリの allUsers フィールドは、クエリのルートフィールドと呼ばれる。

ルートフィールドに続くものを、ペイロードと呼ぶ。（上記のクエリのペイロードに指定されているフィールドは name のみ）

