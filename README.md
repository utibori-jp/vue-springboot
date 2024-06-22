# Memo
## Link list
* [Full-stack Spring Boot + Vue.js:CRUD example (一番網羅的に書いてある記事。)](https://www.bezkoder.com/spring-boot-vue-js-crud-example/)
* [Vue+SpringでSPA作成14[Create・Upadate・Delete処理]](https://techhotoke.hatenablog.com/entry/2022/02/27/010612)
* [Spring Boot + Vue.js:Authentication with JWT & Spring Security Example(アーキテクチャのイメージ図)](https://www.bezkoder.com/spring-boot-vue-js-authentication-jwt-spring-security/)
* [Restful APIについて（そもそもRestfulAPIってなんなん？ってところに踏み込んでる。）](https://qiita.com/mayu_w1223/items/43508b86cbccc1564734)

主な認証方式にはセッションベース認証とJWT認証の2種類がある。違いは以下の通り。
セッションベース認証
・送受信する情報はSessionIDのみ。
・サーバー側でSessionと紐づけされた情報を持っている必要がある。
・リクエスト時にCookieとして送信される。
・リクエストを受けると、サーバー側はSessionIDと紐づけされた情報を基に、アカウントと権限を確認し処理を行う。

JWT認証
・送受信する情報はJWTトークン。トークンには、アカウントIDや権限などの情報を含む
・サーバー側は、セッション情報を保持しない。
・リクエストが来たときに、そのトークンが保持しているアカウントIDや権限を基に処理を行う。
・リクエストにCookieとは違う形でHeaderにつけて送信される

```mermaid
classDiagram
    class SecurityConfig {
        +configure(HttpSecurity http)
    }
    class UserDetailsService {
        +loadUserByUsername(String username)
    }
    class UserDetails {
        +getUsername()
        +getPassword()
        +getAuthorities()
    }
    class AuthenticationProvider {
        +authenticate(Authentication authentication)
    }
    class SessionAuthenticationStrategy {
        +onAuthentication(Authentication, HttpServletRequest, HttpServletResponse)
    }

    SecurityConfig --> UserDetailsService
    SecurityConfig --> AuthenticationProvider
    SecurityConfig --> SessionAuthenticationStrategy
    UserDetailsService --> UserDetails
    AuthenticationProvider --> UserDetailsService
```

```mermaid
sequenceDiagram
    participant Client
    participant SecurityFilter
    participant AuthenticationProvider
    participant UserDetailsService
    participant SessionStrategy

    Client->>SecurityFilter: ログインリクエスト
    SecurityFilter->>AuthenticationProvider: 認証要求
    AuthenticationProvider->>UserDetailsService: ユーザー情報取得
    UserDetailsService-->>AuthenticationProvider: UserDetails
    AuthenticationProvider-->>SecurityFilter: 認証結果
    SecurityFilter->>SessionStrategy: セッション作成
    SessionStrategy-->>SecurityFilter: セッションID
    SecurityFilter-->>Client: ログイン成功 + SessionID (Cookie)
```

sessionベース
```mermaid
sequenceDiagram
    participant Client as Vue.js Client
    participant Server as Spring Boot Server
    participant Security as Spring Security
    participant Session as Session Store

    Client->>Server: ログインリクエスト
    Server->>Security: 認証処理
    Security->>Session: セッション作成
    Session-->>Security: セッションID
    Security-->>Server: セッションID
    Server-->>Client: セッションID（Cookie）
```

JWTトークン認証
```mermaid
sequenceDiagram
    participant Client as Vue.js Client
    participant Server as Spring Boot Server
    participant Security as Spring Security

    Client->>Server: ログインリクエスト
    Server->>Security: 認証処理
    Security-->>Server: JWTトークン生成
    Server-->>Client: JWTトークン
    Client->>Client: ローカルストレージにトークン保存
    Client->>Server: APIリクエスト（トークン付き）
    Server->>Security: トークン検証
    Security-->>Server: 検証結果
    Server-->>Client: APIレスポンス
```

```mermaid
graph TB
    A[認証方式] --> B[セッションベース認証]
    A --> C[トークンベース認証 JWT]
    B --> D[サーバーサイドでセッション情報を保存]
    B --> E[クライアントにSession IDを送信]
    C --> F[サーバーサイドでセッション情報を保存しない]
    C --> G[クライアントにJWTトークンを送信]
    G --> H[トークンに必要な情報を含む]
```


