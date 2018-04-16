# registration.jmx

会員登録のAPIを使った会員登録のシナリオテストサンプル。

## サンプルAPIの仕様

### POST /v1/registers HTTP/1.1

会員登録の申請API。

よくあるメールアドレスを入れるとメールに認証用URLが送信されるような形をイメージして頂ければと思います。

- リクエストサンプル

```bash
curl -v \
-X POST \
-H "Content-Type: application/json" \
-d \
'
{
  "email":"dummy-name+9@gmail.com"
}
' \
http://192.1.1.22/v1/registers
```

- レスポンスサンプル

```
< HTTP/1.1 201 Created
< Server: nginx
< Content-Type: application/json
< Transfer-Encoding: chunked
< Connection: keep-alive
< Location: http://192.1.1.22/v1/registers/6b76e8e1995a580ef2af69fcee19663fbb00ffe868e9326a077bae79d3467f41
< X-Request-Id: 6503b7c8-4ba2-493c-983f-73163db2e443
< Cache-Control: no-cache, private
< Date: Mon, 16 Apr 2018 03:14:26 GMT
<
* Connection #0 to host 192.1.1.22 left intact
{
  "register_token":"6b76e8e1995a580ef2af69fcee19663fbb00ffe868e9326a077bae79d3467f41"
}
```

`register_token` という値が認証用URLの部分だと理解して頂ければ良い。

### GET /v1/registers/{registerToken} HTTP/1.1

認証用URLから本登録の画面に遷移してきた際に利用するであろうAPI。

- リクエストサンプル

```
curl -v http://192.1.1.22/v1/registers/6b76e8e1995a580ef2af69fcee19663fbb00ffe868e9326a077bae79d3467f41
```

- レスポンスサンプル

```
< HTTP/1.1 200 OK
< Server: nginx
< Content-Type: application/json
< Transfer-Encoding: chunked
< Connection: keep-alive
< X-Request-Id: 04dbc563-dfaa-4c18-a864-dd2d0c94b4de
< Cache-Control: no-cache, private
< Date: Mon, 16 Apr 2018 03:20:21 GMT
<
* Connection #0 to host 192.1.1.22 left intact
{
  "register_token":"6b76e8e1995a580ef2af69fcee19663fbb00ffe868e9326a077bae79d3467f41"
}
```

このAPIのレスポンスが404だった場合は、再度会員登録申請を促すようにエラー画面を出す。

### PUT /v1/registers{registerToken} HTTP/1.1

- リクエストサンプル

エンドユーザーから本登録に必要な情報を受け取って送信する。

なお、仕様として同じメールアドレスの登録は出来ない。（申請自体は同じメールアドレスで何回も出来る）

```bash
curl -v \
-X PUT \
-H "Content-Type: application/json" \
-d \
'
{
  "password": "password1",
  "given_name": "Test",
  "family_name": "ユーザー",
  "given_name_kana": "てすと",
  "family_name_kana": "ゆーざー",
  "gender": 2,
  "birthdate": "1999-09-09"
}
' \
http://192.1.1.22/v1/registers/6b76e8e1995a580ef2af69fcee19663fbb00ffe868e9326a077bae79d3467f41
```

- レスポンスサンプル

```
< HTTP/1.1 201 Created
< Server: nginx
< Content-Type: application/json
< Transfer-Encoding: chunked
< Connection: keep-alive
< Location: http://192.1.1.22/v1/users/700432
< X-Request-Id: a9a2c9e7-984e-4656-8003-930821aff1bd
< Cache-Control: no-cache, private
< Date: Mon, 16 Apr 2018 03:24:39 GMT
<
* Connection #0 to host 192.1.1.22 left intact
{
  "sub":700432,
  "email":{"email":"dummy-name+9@gmail.com"
}
```

## シナリオを作る上で考慮すべき点

### リクエスト毎にユニークなメールアドレスを作成する

メールアドレスの重複が許されないので、何とかしてリクエスト毎にユニークなメールアドレスを作ってあげる必要がある。

これを実現する為にカウンタ変数とユーザー定義変数を組み合わせて利用する。

- カウンタ変数は普通に1から順にカウントアップしていく
- JMeterの場合 `${__time(YMDHMS,rTime)}` とすると現在日時が `20161009-143645` のような形で取れる

この2つを組み合わせて以下のように記載する。

```
{
  "email": "dummy-name+${nowDateTime}+${index}@gmail.com",
}
```


こうするとリクエスト毎に `dummy-name+20161009-143645+1@gmail.com` のようなメールアドレスを作る事が出来る。

### `POST /v1/registers HTTP/1.1` のレスポンスボディから `register_token` を取得する

後続のAPIにリクエストするには `POST /v1/registers HTTP/1.1` のレスポンスである `register_token` が必要。

ここは正規表現抽出の機能を使って取り出すしかなさそうだった。

詳しい方法に関しては `jmeter/scenario/registration.jmx` をJMeterのGUIで開いて頂ければと思う。

### マルチバイト文字の送信

`PUT /v1/registers{registerToken} HTTP/1.1` にリクエストする際にユーザー名としてマルチバイト文字の送信が発生する。

これの対処法は簡単でJMeterでHTTPリクエストを作る際に `Content encoding` を `UFT-8` に設定すれば良い。
