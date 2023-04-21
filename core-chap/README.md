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

**-基本的なクエリ-**

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

このリクエストに対して、データベースに保存されている全てのユーザーのリストをレスポンスで返す。
```
// レスポンス

{
    "allUsers": {
        { "name": "Johnny" },
        { "name": "Sarah" },
        { "name": "Alice" },
    }
}
```

Userタイプには、name と age のフィールドが作成されているが、ここでは name だけがクエリで指定されたフィールドなので、

結果として age のデータはサーバーから返されない。age も取得する場合は、クエリのペイロードに新しいフィールドを追加する。
```
// ageフィールドの追加

{
    allUsers {
        name
        age
    }
}
```

GraphQLのメリットの一つは、ネストした情報を参照することができるところにある。例えば、ユーザー(User)が書いた全ての記事(posts)を読み込む場合、
タイプ構造の構造と同じようにクエリに記述することで、その構造に沿った情報を要求できる。
```
{
    allUsers {
        name
        age
        posts {
            title       // 記事のタイトルだけを取得する
        }
    }
}
```

**-引数によるクエリ-**

GraphQLには、スキーマで指定している各フィールドに対して、０個以上の引数を持つことができる。例えば、特定の範囲のユーザー情報が欲しい場合は、
allUsers に対して特定の人数のユーザー情報のみ返すように要求するパラメーターを持つことが出来る。
```
{
    allUsers(last: 2) {     // データに記録されている最後の２人のユーザー情報を取得する
        name
    }
}
```

### ミューテーションによるデータの変更

ほとんどのアプリケーションでは、サーバーから情報を取得するだけでなく、バックエンドに保存されているデータに変更を加える方法も必要である。

GraphQLでは、そのようなデータへの変更をミューテーションを使用することで可能にしている。（ミューテーションには、３種類の機能がある）

1. 新しいデータの作成

2. 既存のデータの更新

3. 既存のデータの削除

ミューテーションはクエリと同じような構造だが、必ず mutation キーワードを先頭に記述してから始める必要がある。
```
mutation {
    createUser(name: "Bob", age: 36) {
        name
        age
    }
}
```

ミューテーションにもルートフィールドが存在し(createUser)、ペイロード(name, age)を指定することも可能。

上記の場合、createUser フィールドには新しいユーザーの名前(name)と年齢(age)を引数にとる。

ここから User タイプを拡張することで、新しいユーザーが作成されるたびにサーバー側で自動生成される一意のIDを追加することが可能。
```
type User {
    id: ID!
    name: String!
    age: Int!
}
```

### サブスクリプションによるリアルタイムアップデート

重要な情報をすぐに見られるように、サーバーにリアルタイムで接続することが重要であり、GraphQLでは、これに対してサブスクリプションを提供している。
クライアントがイベントを購読すると、サーバーへの安定した接続を開始する。その特定のイベントが発生するたびに、サーバーは対応するデータをクライアントに対してプッシュする。
サブスクリプションは、リスエスト・レスポンス形式のクエリやミューテーションとは異なり、クライアントに送信されるデータに連続性がある。

サブスクリプションは、クエリとミューテーションと同じ構文で使用される。
```
// Userタイプで発生するイベントを購読する例

subscription {
    newUser {
        name
        age
    }
}
```

クライアントがこのサブスクリプションをサーバーに送信することで、接続が開始される。
上記のサブスクリプションでは、新しいユーザーが作成するミューテーションが実行されるたびに、サーバーが作成されたユーザーに関する情報（name, age）を、接続してるクライアントに送信する。
```
{
    "newUser": {
        "name": "Jane",
        "age": 23
    }
}
```

### スキーマの定義

クエリ、ミューテーション、サブスクリプリョンのそれぞれに対して、それを実行できるスキーマを書く必要がある。
スキーマは、GraphQL API を操作する上で重要な概念であり、クライアントがデータを要求する方法を定義する。
APIのスキーマを作成するときは、特別なルートタイプが存在する。
```
type Query { ... }
type Mutation { ... }
type Subscription { ... }
```

上記の「クエリでデータを取得する」で見た allUsers-query を有効にするには、Queryタイプを定義する必要がある。
```
type Query {
    allUsers: [User!]!      // Userの情報もスキーマで定義されている
}
```

上記の allUsers がルートフィールドと呼ばれ、allUsersフィールドに対して引数を追加することもできる。
```
type Query {
    allUsers(last: Int): [User!]!
}
```

同様に、createUser-mutation では、Mutationタイプにルートフィールドを追加する必要がある。
```
type Mutation {
    createUser(name: String!, age: Int!): User!       // name, age の引数をとる
}
```

最後に、サブスクリプションでは、newUserルートフィールドを追加する必要がある。
```
type Subscription {
    newUser: User!
}
```

**-ここまで見た全てのクエリ・ミューテーション・サブスクリプションの完全なスキーマ-**

```
// スキーマの定義
type Query {
    allUsers(last: Int): [User!]!
    allPosts(last: Int): [Post!]!
}

type Mutation {
    createUser(name: String!, age: Int!): User!
    updateUser(id: ID!, name: String!, age: String!): User!
    deleteUser(id: ID!): User!
}

type Subscription {
    newUser: User!
}

type User {
    id: ID!
    name: String!
    age: Int!
    posts: [Post!]!
}

type Post {
    title: String!
    author: User!
}
```