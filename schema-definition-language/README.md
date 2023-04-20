## スキーマ定義言語 (Schema Definition Language)

スキーマ定義言語（SDL）：スキーマを書くための、GraphQLにある独自の言語。

### GraphQLスキーマ定義とは

GraphQLでスキーマ（データの構造）を指定する簡潔な方法。

**-指定方法-**

タイプとフィールドで構成されている。
```
type Post {
    id: String!
    title: String!
    publishedAt: DateTime!
    likes: Int! @default(value: 0)
    blog: Blog @relation(name: "Posts")
}

type Blog {
    id: String!
    name: String!
    description: String
    posts: [Post!]! @relation(name: "Posts")
}
```

**-タイプ-**

タイプには名前があり、１つ以上のインターフェースを実装できる。
```
type Post {
    ...
}
```

**-フィールド-**

フィールドには、名前とタイプがある。
```
age: Int
age: Int!           // ! が付いたフィールドには必ず値が入る（NOT NULL）
users: [String]!    // リストは [] で表す

// タイプ（スカラー型）の種類
・Int
・Float
・String
・Boolean
・ID
```
スカラー型の他にも、スキーマ定義で定義された型も使用できる。
```
type Post {
    ...
    blog: Blog      // スキーマ定義で定義した型を使用
}

// 定義
type Blog {
    ...
    description: String
}
```

**-インターフェース-**

フィールドのリストであり、必ず同じフィールド・型を共有で実装する仕組み。
```
// Itemインターフェースは必ず title: String! というフィールドを持つ
interface Item {
    title: String!
}
```

**-スキーマディレクティブ-**

要素の後ろに記述することで、定義したスキーマの要素に任意の情報を付けることができる。
```
type User {
    // ユーザーのフォロワーをデフォルトで０に設定する
    follower: Int! @default(value: 0)
}
```