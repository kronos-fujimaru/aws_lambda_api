# AWS LambdaによるREST API

## 1. AWS Lambda

### 1.1. jarファイルの生成

AWS Lambda（以下、Lambda）にアップロードするAPIをJavaで作成する。ここでは、リクエストで商品名、単価、数量を渡すと、レスポンスで税抜価格と税込価格を返すAPIを作成する。

**com.lambda.data.Request.java**

```java
package com.lambda.data;

public class Request {
  // 商品名
  private String itemName;

  // 単価
  private Integer price;

  // 数量
  private Integer quantity;

  // getter/setterは中略
}
```

**com.lambda.data.Response.java**

```java
package com.lambda.data;

public class Response {
  // リクエスト
  private Request request;

  // 税抜価格
  private Integer taxExcluded;

  // 税込価格
  private Integer taxIncluded;

  // getter/setterは中略
}
```

**com.lambda.Calculator.java**

```java
package com.lambda;

import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;
import com.lambda.data.Request;
import com.lambda.data.Response;

public class Calculator implements RequestHandler<Request, Response> {
  @Override
  public Response handleRequest(Request request, Context context) {
    // リクエスト情報を基に税抜価格と税込価格を算出する
    Integer taxExcluded = request.getPrice() * request.getQuantity();
    Integer taxIncluded = (int)(request.getPrice() * request.getQuantity() * 1.1);

    // レスポンス情報を作成する
    Response response = new Response();
    response.setRequest(request);
    response.setTaxExcluded(taxExcluded);
    response.setTaxIncluded(taxIncluded);

    return response;
  }
}
```

<br>

ビルドツール（Mavenなど）でjarファイルを生成する。

<br>

### 1.2. Lambda関数の生成

AWS Lambdaを用いてAPIを作成する。Lambdaの画面で[関数の作成]ボタンをクリックする。

<img src="img/01_01.jpg" width="700">

[一から作成]を選択、関数名やランタイムなどを設定し、[関数の作成]ボタンをクリックする。

<img src="img/01_02.jpg" width="600">

関数の概要下にあるコードタブのアップロード元から[.zip または .jarファイル]を選択する。

<img src="img/01_03.jpg" width="700">

[アップロード]ボタンをクリックし、jarファイルを選択し、[保存]ボタンをクリックする。

<img src="img/01_04.jpg" width="700">

コードタブ > ランタイム設定の[編集]ボタンをクリックする。ランタイム設定画面のハンドラにあるパッケージ名とクラス名の部分を作成したAPIのものに修正し、[保存]ボタンをクリックする。

<img src="img/01_05.jpg" width="700">

テストタブ > イベントJSON欄にリクエストで送るデータ（商品名、単価、数量）をJSON形式で記述し、[テスト]ボタンをクリックする。

<img src="img/01_06.jpg" width="700">

実行結果としてレスポンス（税抜価格、税込価格）の内容が表示されることを確認する。

<img src="img/01_07.jpg" width="700">

<br><br>

## 2. Amazon API Gateway

Amazon API Gateway（以下、API Gateway）をLambda関数のトリガーとして連携することで、Lambda関数をWeb API（REST API）として使用することができる。<br>Lambda関数の概要内の[トリガーを追加]ボタンをクリックする。

<img src="img/02_01.jpg" width="700">

トリガーの設定で[API Gateway] > [REST API]を選択し、SecurityやAPI nameを設定し、[追加]ボタンをクリックする。

<img src="img/02_02.jpg" width="600">

設定タブのAPI名リンクをクリックし、API Gatewayの詳細に遷移する。

<img src="img/02_03.jpg" width="700">

アクションから[メソッドの作成]を選択する。

<img src="img/02_04.jpg" width="700">

[POST]を選択し、決定ボタン（✓）をクリックする。

<img src="img/02_05.jpg">

Lambda関数欄に実行対象の関数名を指定し、[保存]ボタンをクリックする。Lambda関数への権限追加に関するメッセージが表示されたら[OK]ボタンをクリックする。

<img src="img/02_06.jpg" width="600">

アクションから[APIのデプロイ]を選択する。

<img src="img/02_07.jpg" width="210">

ステージと説明を指定し、[デプロイ]ボタンをクリックする。

<img src="img/02_08.jpg" width="500">

<br><br>

## 3. 通信確認

curlコマンドやRESTクライアントツールで、API Gatewayのエンドポイントに対してリクエストを送り、レスポンスが返ってくることを確認する。

Windowsでは、次のcurlコマンドで通信確認ができる。

```
curl -X POST [API Gatewayのエンドポイント] -H "Content-Type: application/json" -d {\"itemName\":\"Apple\",\"price\":100,\"quantity\":5}
```
