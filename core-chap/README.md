# GraphQL のコアコンセプト

## クエリとミューテーション

### クエリでデータを取得する

REST API では、特定のエンドポイントに基づいて、定義されたデータが返される。

GraphQL では、クライアントが必要とするデータを取得できるように、返すデータ構造を固定せずに複数のエンドポイントをまとめて１つのエンドポイントで情報を取得できる柔軟性を持つ。その結果として、クライアントはサーバーに必要な情報を定義したクエリを送信する必要がある。

**- 基本的なクエリ -**

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
    "allUsers": [
        {
            "name": "Johnny"
        },
        {
            "name": "Sarah"
        },
        {
            "name": "Alice"
        }
    ]
}
```

User タイプには、name と age のフィールドが作成されているが、ここでは name だけがクエリで指定されたフィールドなので、

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

GraphQL のメリットの一つは、ネストした情報を参照することができるところにある。例えば、ユーザー(User)が書いた全ての記事(posts)を読み込む場合、
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

**- 引数によるクエリ -**

GraphQL には、スキーマで指定している各フィールドに対して、０個以上の引数を持つことができる。例えば、特定の範囲のユーザー情報が欲しい場合は、
allUsers に対して特定の人数のユーザー情報のみ返すように要求するパラメーターを持つことができる。

```
{
    allUsers(last: 2) {     // データに記録されている最後の２人のユーザー情報を取得する
        name
    }
}
```

また、GraphQL では、すべてのフィールドとネストされたオブジェクトが独自の引数を取得することができる。

つまり、クエリ１つの中で複数の引数を指定することで、より柔軟なデータ要求を実現することが可能である。

```
{
    user(id: 1) {
        name
        posts(status: "published") {
            title
            content
        }
    }
}
```

上記のクエリでは、user フィールドの引数に'id: 1'を指定し、さらに posts フィールドに'status: "published"'という引数を追加している。

これにより、特定のユーザーの公開された投稿のみを取得することができる。

**- エイリアス -**

異なる引数を使用して同じフィールドを直接クエリとしてリクエストすることはできない。

したがって、エイリアスを使用してフィールドの結果を任意の名前に変更し、同じフィールドに異なる引数を指定することができる。

```
{
    user1: user(id: 1) {
        name
        age
    }
    user2: user(id: 2) {
        name
        age
    }
}
```

上記のクエリでは、user フィールドを２回リクエストしている。つまり、１つのクエリで異なる引数を持つ同じフィールドを複数回取得することが可能である。

これは GraphQL の特徴であり、REST API では、同じフィールドに異なる引数を渡す場合、通常は複数のリクエストを行う必要がある。

**- フラグメント -**

仮に、２人のユーザーとその年齢を並べてみることができるした場合、そのクエリではフィールドを少なくとも１回繰り返す必要がある。

そのような複雑なフィールドに対して、GraphQL にはフラグメントと呼ばれる再利用可能なユニットが含まれている。

フラグメントを使用すると、フィールドのテンプレートを構築し、必要に応じてそれをクエリに含むことができる。

```
{
    user1: user(id: 1) {
        ...userFields
    }
    user2: user(id: 2) {
        ...userFields
    }

    fragment userFields on User {
        name
        age
    }
}
```

**- オペレーション名 -**

GraphQL では、クエリに名前をつけることができ、これをオペレーション名と呼ぶ。

オペレーション名を使用し、クエリに一意の名前をつけることができ、複数のクエリやミューテーションをまとめる場合や、サーバー側のログでクエリを識別する場合に便利である。
また、オペレーション名を指定する場合、クエリの種類を明示する必要がある。

```
query GetUser {
    user(id: 1) {
        name
        email
    }
}
```

**- 変数 -**

GraphQL では、フィールドへの動的な引数を変数に入れて渡すことができる。

- 操作工程

1.  クエリ内の静的な値を置き換える。'$variableName'

2.  クエリで渡す変数として宣言する。

3.  variableName: value の形で変数を渡す。

全体をまとめると以下のようになる。

```
query GetUser($id: ID!, $status: String!) {
    user(id: $id) {
        name
        posts(status: $status) {
            title
            content
        }
    }
}

{
    "id": 1
    "status": "published"
}
```

また、型宣言のデフォルト値を追加することで、クエリ内の変数にデフォルト値を割り当てることができる。

```
query GetUser($id: ID!, $status: String! = "published") {
    user(id: $id) {
        name
        posts(status: $status) {
            title
            content
        }
    }
}
```

**- ディレクティブ -**

ディレクティブは、特定の条件に合わせて、フィールドや操作の動作をカスタマイズできる機能である。

- @include

引数としてブール値を受け取り、'true'の場合にのみフィールドに結果を含む。

```
query GetUser($id: ID!, $includePosts: Boolean!) {
    user(id: $id) {
        name
        posts @include(if: $includePosts) {
            title
            content
        }
    }
}

{
    "id": "1"
    "includePosts": "false"
}
```

上記の場合、'@include(if: Boolean)'の引数に false が渡されたことで、以下のようなレスポンスが返される。

```
{
    "data": {
        "user": {
            "name": "Taro"
        }
    }
}
```

### ミューテーションによるデータの変更

ほとんどのアプリケーションでは、サーバーから情報を取得するだけでなく、バックエンドに保存されているデータに変更を加える方法も必要である。

GraphQL では、そのようなデータへの変更をミューテーションを使用することで可能にしている。（ミューテーションには、３種類の機能がある）

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

ここから User タイプを拡張することで、新しいユーザーが作成されるたびにサーバー側で自動生成される一意の ID を追加することが可能。

```
type User {
    id: ID!
    name: String!
    age: Int!
}
```

### サブスクリプションによるリアルタイムアップデート

重要な情報をすぐに見られるように、サーバーにリアルタイムで接続することが重要であり、GraphQL では、これに対してサブスクリプションを提供している。
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

## スキーマ

### スキーマの定義言語（SDL）

GraphQL には、API のスキーマを定義するために使う[スキーマ定義言語（SDL)](https://github.com/DaisukeKarasawa/graphql-prc/tree/main/schema-definition-language)という構文がある。

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

### スキーマの定義

クエリ、ミューテーション、サブスクリプリョンのそれぞれに対して、それを実行できるスキーマを書く必要がある。
スキーマは、GraphQL API を操作する上で重要な概念であり、クライアントがデータを要求する方法を定義する。
API のスキーマを作成するときは、特別なルートタイプが存在する。

```
type Query { ... }
type Mutation { ... }
type Subscription { ... }
```

上記の「クエリでデータを取得する」で見た allUsers-query を有効にするには、Query タイプを定義する必要がある。

```
type Query {
    allUsers: [User!]!      // Userの情報もスキーマで定義されている
}
```

上記の allUsers がルートフィールドと呼ばれ、allUsers フィールドに対して引数を追加することもできる。

```
type Query {
    allUsers(last: Int): [User!]!
}
```

同様に、createUser-mutation では、Mutation タイプにルートフィールドを追加する必要がある。

```
type Mutation {
    createUser(name: String!, age: Int!): User!       // name, age の引数をとる
}
```

最後に、サブスクリプションでは、newUser ルートフィールドを追加する必要がある。

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
