# Swaggerを使ってRESTful API環境構築

このチュートリアルではSwaggerを使ってRESTful APIのテスト環境の構築を行う。  
Swaggerの概観のため、言語やフレームワークは任意のものを用いるが、以下の環境を推奨する  

```
OS     : MacOS (POSIX系ならなんでもいいと思う)
Docker : 19.x.x
```

## Swaggerとは

詳細な説明は検索すればよろし。  

Swaggerは`Open API`を用いてRESR APIを構築するためのツールセットで、一度`YAML`ファイルを作成すればドキュメントの生成から実際のリクエストを投げるなど様々なことができる。  
あとAPIのバージョンも管理できるのでとても便利

元々はSwaggerが仕様策定していたものを`Open API`として拡張しているっぽい。  
なので`YAML`ファイルでのバージョン指定では`swagger:2.0`と `openapi: 3.0.0`があり、若干分かりづらい。  
基本的には最新の`openapi:3.0.0`を使うものだと考えていいと思う。


Swaggerが提供しているツールを以下に示す。

|   **ツール**     |                                  **説明**                             |
| :-------------: | :-------------------------------------------------------------------: |
|  [Swagger Spec](https://swagger.io/specification/)   |             REST APIに対して Swagger の仕様に準じたドキュメント             |
| [Swagger Editer](https://swagger.io/tools/swagger-editor/)  |                Swagger Spec の設計書を記載するためのエディタ                |
|   [Swagger UI](https://swagger.io/tools/swagger-ui/)    | Swagger Spec で記載された設計からドキュメントをHTML形式で自動生成するツール     |
| [Swagger Codegen](https://swagger.io/tools/swagger-codegen/) |           Swagger Spec で記載された設計からAPIのスタブを自動生成            |

これらはスタンドアロンで使うためにダウンロード可能になっているが、[Swagger Hub](https://app.swaggerhub.com/home)に登録すれば上記のものをオンライン上で利用することが可能になる。  
まずはこの`Swagger Hub`を使ってなれることをオススメしたいのでここからは`Swagger Hub`上での説明になる。

## Swaggerのプロジェクトを作成する。
[Build, Collaborate & Integrate APIs \| SwaggerHub](https://app.swaggerhub.com)にアクセスしてログイン後、プロジェクトを作成する。

設定は以下の感じでいいでしょう。

```js
OpenAPI Version : 3.0.0       // 2.0は旧バージョンなので3.0を指定
Template        : Simple API  // テンプレートを使うときはここで。
Visibility      : Public      // 公開にするかどうか。
Auto Mock API   : ON          // APIのモックを作成するか否か
```

ちなみに`Template`の種類が豊富で悩むけど、基本的な`CRUD`を作るなら`Simple API`がちょうどいい気がする。


## 簡単なYAMLの書き方説明

```yaml

  # OpenAPI 仕様のバージョン
openapi: 3.0.0

  # APIのモックアップサーバーのURL
  # ここは自動生成されるので基本的には触らない。
  # 実際にAPIが配置しているサーバーを登録することでテストも可能
servers:
  # Added by API Auto Mocking Plugin
  - description: SwaggerHub API Auto Mocking
    url: https://virtserver.swaggerhub.com/TanisukeGoro/REST_TEST/1.0.0
  - description: SwaggerHub API Auto Mocking
    url: http://localhost:8000/api/v1/

  # APIの情報を記述
info:
  description: Homie prototypeのAPI
  version: "1.0.0"
  title: Homie API 
  contact:
    url: https://twitter.com/okita_kamegoro


  # APIの種類をまとめるためにタグを定義する
tags:
  - name: auth
    description: 認証関連
  - name: user
    description: ユーザー関連
  - name: matching
    description: マッチング関連
  - name: chat
    description: チャット関連
  - name: settings
    description: 設定の取得・変更

  # ここにAPIの定義を記述する。詳細は次節にて。
paths:

  # pathsでリクエスト・レスポンスの中身を全て書くのではなく、
  # このComponents Object と連携して定義を書いていく
components:
  schemas:
    # 共通モデル(MVCのM)

  parameters:
    # GET などで取得範囲を変更するときなどに利用

  requestBodies:
    # 共通で使う POST データ

  responses:
    # 共通のレスポンス定義、 paths の中で利用
```

### pathsについて

`YAML`ファイルの`paths`には、`エンドポイント`や`メソッド`・`リクエスト`などAPIの定義を入れ子構造で記述する。

```yaml
paths:
  # エンドポイント
  /user:
    # メソッド
    post:
      # タグづけ
      tags:
        - user
      summary: Create user
      description: This can only be done by the logged in user.
      operationId: createUser
      responses:
        default:
          description: successful operation
      requestBody:
        content:
          application/json:
            # こんな感じにしてComponentのデータを引用する
            schema:
              $ref: '#/components/schemas/User'
        description: Created user object
        required: true
  /user/createWithArray:
    post:
      tags:
        - user
      summary: Creates list of users with given input array
      operationId: createUsersWithArrayInput
      responses:
        default:
          description: successful operation
      requestBody:
        $ref: '#/components/requestBodies/UserArray'
   # 動的URLを定義する場合は以下のようにする。
  '/user/{username}':
    get:
      tags:
        - user
      summary: Get user by user name
      operationId: getUserByName
      parameters:
        - name: username
          in: path
          description: The name that needs to be fetched. Use user1 for testing.
          required: true
          schema:
            type: string
        # ステータスコードによって返却を帰るとも可能
      responses:
        '200':
          description: successful operation
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
            application/xml:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          description: Invalid username supplied
        '404':
          description: User not found
```


## YAMLのダウンロード

ドキュメントの作成ができたらエディターの右上にある`Export`> `Download API` > `YAML Resolved`クリックして`YAML`ファイルをダウンロードする



## APIソースのビルド

```
GENERATOR=php-laravel
```
composerのインストールがされていない場合は以下を実行したのちに、terminalを再起動する。

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === 'a5c698ffe4b8e849a443b120cd5ba38043260d5c4023dbf93e1558871f1f07f58274fc6f4c93bcfd858c6bd0775cd8d1') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
```
再起動したら確認してみる

```bash
composer -V
```

`composer`の高速化を図る

```bash
composer config -g repos.packagist composer https://packagist.jp
composer global require hirak/prestissimo
```


[OpenAPI generatorを試してみる - Qiita](https://qiita.com/amuyikam/items/e8a45daae59c68be0fc8)

[Composer](https://getcomposer.org/download/)