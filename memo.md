# ユースケース: Ajax通信

## エントリーポイント
### プロジェクトディレクトリを作成
### HTMLファイルの用意
### ローカルサーバーでHTMLを確認する
### ウェブブラウザとDOM
### このセクションのチェックリスト

---
## HTTP通信
[HTTP通信 · JavaScript Primer \#jsprimer](https://jsprimer.net/use-case/ajaxapp/http/)

GitHubのAPIを呼び出す処理を実装する<br>
↓<br>
そのためにはHTTP通信をする必要がある<br>
↓<br>
ウェブブラウザ上でJavaScriptからHTTP通信するために、 **Fetch API** という機能を使う<br>


### Fecth API
- HTTP通信を行ってリソースを取得するためのAPI
  - 「リソース」ってなんだ？
  - 「API」ってなんだっけ？
    - アプリケーションインターフェイス（Application Interface）の略
      - ここで言う「アプリ」とは？
- ページ全体を再読み込みせずに指定したURLからデータを取得できる
- 同じくHTTP通信を行う[XMLHttpRequest](https://developer.mozilla.org/ja/docs/Web/API/XMLHttpRequest)よりも強力

リクエストを送信する
- `fecth`コマンドを使う
- `fecth`にURLを与えることでHTTPリクエストが作成され、サーバーとのHTTP通信を開始する
- `https://api.github.com/users/GitHubユーザーID`
  - GitHubに用意された、ユーザー情報を取得するAPI
  - 英数字とハイフン以外はIDに利用できない
    - そのため、`encodeURIComponent`関数で`/`や`%`などURLとして特殊な意味を持つ文字列をただの文字列としてエスケープしたものをユーザーIDとして結合する

指定したGitHubユーザーIDの情報を取得するURLに対して`fecth`メソッドでGETのHTTPリクエストを送るコード

```javascript
const userID = "任意のGitHubユーザーID";
fecth(`https://api.github.com/users/${encodeURIComponent(userID)});
```

### レスポンスの受け取り
- HTTPリクエストを送信した
  - まだレスポンスを受け取る処理を書いていない！！
  - なのでサーバーからのレスポンスのログをコンソールに出力する処理を実装する
-  `fetch`メソッドは`Promise`を返す
  - `Promise`インスタンスは`Response`オブジェクトでresolveされる
    - ？？？
    - `Response`：リクエストのレスポンスを表すオブジェクト
    - resolveってなんだ？
  - 送信したリクエストにレスポンスが返されると`then`コールバックが呼び出される
    - ？コールバック？then？


以下のコードについて
- `Response`オブジェクトの`status`プロパティからはHTTPレスポンスのステータスコードが入手できる
  - 200とか404とかのことか
- `json`メソッドも`Promise`を返す
  - HTTPレスポンスボディをJSONとしてパースしたオブジェクトでresolveされる
    - だからresolveてなんやねんな
```javascript
const userId ="js-primer-example";
fetch(`https://api.github.com/users/${encodeURIComponent(userId)}`)
  .then(response => {
    console.log(response.status); // => 200
    return response.json().then(userInfo => {
      // JSONパースされたオブジェクトが渡される
      console.log(userInfo); // => {...}
    });
  });
```

### エラーハンドリング
- HTTP通信にはエラーがつきもの
  - やんなっちゃうなん
  - だからFetch APIを使った通信においてもエラーをハンドリングする必要がある
    - 「ハンドリング」？
- 例えば
  - サーバー通信でネットワークエラーが発生したとき
    - ネットワークエラーを表す`NetworkError`オブジェクトでrejectされた`Promise`が返される
      - rejectってなんだ！！
    - すなわち…`then`メソッドの第二引数or `catch`メソッドのコールバック関数が呼び出される
  
```javascript
const userId = "js-primer-example";
fetch(`https://api.github.com/users/${encodeURIComponent(userId)}`)
    .then(response => {
      console.log(response.status);
      return response.json().then(userInfo => {
        console.log(userInfo);
      });
    }).catch(error => {
      console.error(error);
    });
```

- リクエストが成功したかどうか
  - `Response`オブジェクトの`ok`プロパティで認識できるう！
    - HTTPステータスコードが
      - 200番台→`true`
      - それ以外（400番台や500番台）→`fasle`
  - 以下のコード…`ok`プロパティが`false`となるエラーレスポンスをハンドリング

```javascript
const userId = "js-primer-example";
fetch(`https://api.github.com/users/${encodeURIComponent(userId)}`)
    .then(response => {
      console.log(response.status);
      // エラーレスポンスが返されたことを検知
      if(!response.ok) {
        console.error("エラーレスポンス", response);
      } else {
        return response.json().then(userInfo => {
          console.log(userInfo);
        });
      }
    }).catch(error => {
      console.error(error);
    });
```

### [コラム] XMLHttpRequest
### このセクションのチェックリスト
- 
- 
- 
- 

---
## データを表示する
前項で取得したデータをHTMLに整形し、アプリケーションにユーザー情報を表示する

### HTMLを組み立てる
- HTML文字列の生成には、テンプレートリテラルを使う
  - 利点
    - HTMLのインデントが表現できる
      - 文字列内改行が可能なため
    - 変数の埋め込みが簡単
      - HTMLのテンプレートに対し動的データを当てはめるのに適する

```javascript
// GitHubのユーザー情報から組み立てるHTMLのテンプレートを宣言する
const view = `
<h4>${userInfo.name} (@${userInfo.login})</h4>
<img src="${userInfo.avatar_url}" alt="${userInfo.login}" height="100">
<dl>
    <dt>Location</dt>
    <dd>${userInfo.location}</dd>
    <dt>Repositories</dt>
    <dd>${userInfo.public_repos}</dd>
</dl>
`
// `userInfo`オブジェクトの値を当てはめるとこんなHTML文字列になる
<h4>js-primer-example (@js-primer-example)</h4>
<img src="https://github.com/js-primer-example.png" alt="js-primer-example" height="100">
<dl>
    <dt>Location</dt>
    <dd>Japan</dd>
    <dt>Repositories</dt>
    <dd>1</dd>
</dl>
```

### HTML文字列をDOMに追加する
- 生成したHTML文字列をDOMツリーに追加して表示する
  - 動的にHTMLをセットするため、目印となる要素を`index.html`に追加
    - 今回は`result`というidを持ったdiv要素（以下`div#result`）を配置


```html
<html lang="ja">
    <head>
        <meta charset="utf-8">
        <title>Ajax Example</title>
    </head>
    <body>
        <h2>GitHub User Info</h2>

        <button onclick="fetchUserInfo('HARAKATSU');">Get user info</button>
        <!-- 整形したHTMLの挿入先 -->
        <div id="result"></div>
        
        <script src="index.js"></script>
    </body>
</html>
```

- `div#result`の子要素としてHTML文字列を挿入する
  - 「子要素」ってどういうことだっけ？
  - [document.getElementById](https://developer.mozilla.org/ja/docs/Web/API/Document/getElementById)メソッドを使ってid属性が設定された要素にアクセス
  - `div#result`要素が取得できたら、生成したHTML文字列を`innerHTML`プロパティにセットする
```javascript
const result = document.getElementById("result");
result.innerHTML = view;
```

- JavaScriptによってHTMLをDOM要素に追加する方法
  1. HTML文字列を[Element#HTML](https://developer.mozilla.org/ja/docs/Web/API/Element/innerHTML)にセットする
     - 今回のような方法
  2. 文字列ではなく[Element](https://developer.mozilla.org/ja/docs/Web/API/Element)オブジェクトを生成して[手続き的にツリー構造を構築する](https://developer.mozilla.org/ja/docs/Web/API/Node/appendChild)
     - より安全だがより冗長なコード


### HTML文字列をエスケープする
- `Element#HTML`に文字列をセットすると、その文字列はHTMLとして解釈される
  - もしユーザー名に`<`や`>`があると、意図しないHTMLの挙動になってしまう可能性がある
  - それを回避するために、安全な記号に置換する
    - この処理のことを一般にHTMLの **エスケープ** と呼ぶ
- 多くのViewライブラリは内部にエスケープ機構を持っている
- エスケープ用のライブラリを使うケースもある
- 今回の独自実装は特殊なケース

```js
// 特殊な記号に対するエスケープ処理を行う関数
function escapeSpecialChars(str) {
  return str
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&rt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#039;")
}
```

- この`escapeSpecialChars`関数を行う
  - どこで？→HTML文字列の中で`userInfo`から値を注入しているすべての箇所で
  - 手間じゃない？→手間
    - なのでテンプレートリテラルをタグ付けする
      - テンプレートによる値の埋め込みを関数の呼び出しとして扱える
      - 明示的なエスケープ関数の呼び出しを不要にする！！

```js
// タグ関数escapeHTMLを定義
// 文字列リテラルと値が元の順番通りに並ぶように文字列を組み立てつつ
// 値が文字列型であればエスケープする
function escapeHTML(strings, ...values) {
  // resultの初期値はsirings[0]の値
  return strings.reduce((result, str, i) => {
    const value = values[i - 1];
    if (typeof value === "string") {
      return relust + escapeSpecialChars(value) + str;
    } else {
      return result + String(value) + str;
    }
  });
}
```
- reduceってどんな関数だっけか？
  - `累積値, 要素, インデックス, 配列`を引数として渡す
    - `要素`（第二引数）は`累積値`の初期値


### このセクションのチェックリスト


## Promiseを活用する
### 関数の分割
### Promiseのエラーハンドリング
### Promiseチェーンのリファクタリング
### Async Functionへの置き換え
### ユーザーIDを変更できるようにする
### このセクションのチェックリスト