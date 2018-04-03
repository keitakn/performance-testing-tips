# login.jmx

ログイン処理を行うAPIの単体試験シナリオサンプル。

## サンプルAPIの仕様

curl だと以下のような形でリクエストを送信します。

```bash
curl -v \
-X POST \
-H "Content-Type: application/json" \
-d \
'
{
  "email":"dummy-name+10@gmail.com",
  "password": "password1234"
}
' \
http://192.1.1.22/v1/login
```

## 概要

1台のインスタンスに対して試験を行う事を想定しています。

当然、 `dummy-name+10@gmail.com` のようなテストデータは予めDBに登録しておく必要があります。

`+10` の部分を連番にしておく事が重要です。

同じ値を使った場合、負荷試験を行った際に特定のレコードだけにアクセスが集中するので実際のユーザーの動きと比べかなり不自然な試験内容になってしまいます。

JMeterにはrandomな値を変数として利用出来る機能があるので、それを使えば実現可能です。
