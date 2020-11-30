[HTTP通信 · JavaScript Primer \#jsprimer](https://jsprimer.net/use-case/ajaxapp/http/)

## HTTP通信
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