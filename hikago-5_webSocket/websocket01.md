##WebSocket
今回はgoとwebsocketを使って、軽いチャットアプリを作るハンズオンをしてみようかと思います。
パッケージは、goのサブパッケージであるcode.google.com/p/go.net/websocketと、golliraWebToolKitの中のgithub.com/gorilla/websocketがあるのですが、今回は後者を使っていこうと思います。

##websocketとは？
websocketとは従来のhttp通信のようなリクエストに対してレスポンスが返ってくるような形ではなく、
一度TCP/IP通信を確立させたら、自由に双方向通信が可能になる、といったようなものです。
実際には、TCP/IP通信を確立させて、upgreade：websocketと、ハンドシェイクリクエストを送り、その後はwebsocketプロトコルで双方向通信をする、という形です。実際にもっと詳しく知りたい方は、http://www.hcn.zaq.ne.jp/___/WEB/RFC6455-ja.html　仕様の日本語訳を読んだり、などするといいかもしれませんが、このハンズオンである程度の動きはつかめるように書こうとは思っています。

##websocketを使う(クライアント側)
まず、手っ取り早く理解するため、クライアント側の実装から始めたいと思っています。
正直、ここに関しては十分サンプル自体が簡素化されてるんで、そのまま載せています。
```html
<!DOCTYPE html>
<html lang="en">
<head>
<title>Chat Example</title>
<script src="//ajax.googleapis.com/ajax/libs/jquery/2.0.3/jquery.min.js"></script>
<script type="text/javascript">
    $(function() {
    var conn;
    var msg = $("#msg");
    var log = $("#log");
    function appendLog(msg) {
        var d = log[0]
        var doScroll = d.scrollTop == d.scrollHeight - d.clientHeight;
        msg.appendTo(log)
        if (doScroll) {
            d.scrollTop = d.scrollHeight - d.clientHeight;
        }
    }
    $("#form").submit(function() {
        if (!conn) {
            return false;
        }
        if (!msg.val()) {
            return false;
        }
        conn.send(msg.val());
        msg.val("");
        return false
    });
    if (window["WebSocket"]) {
        conn = new WebSocket("ws://{{$}}/ws");
        conn.onclose = function(evt) {
            appendLog($("<div><b>通信が切断されました</b></div>"))
        }
        conn.onmessage = function(evt) {
            appendLog($("<div/>").text(evt.data))
        }
    } else {
        appendLog($("<div><b>あなたのブラウザはwebsocketをサポートしていません</b></div>"))
    }
    });
</script>
<style type="text/css">
html {
    overflow: hidden;
}
body {
    overflow: hidden;
    padding: 0;
    margin: 0;
    width: 100%;
    height: 100%;
    background: black;
}
#log {
    background: white;
    margin: 0;
    padding: 0.5em 0.5em 0.5em 0.5em;
    position: absolute;
    top: 0.5em;
    left: 0.5em;
    right: 0.5em;
    bottom: 3em;
    overflow: auto;
}
#form {
    padding: 0 0.5em 0 0.5em;
    margin: 0;
    position: absolute;
    bottom: 1em;
    left: 0px;
    width: 300%;
    overflow: hidden;
}
</style>
</head>
<body>
<div id="log"></div>
<form id="form">
    <input type="submit" value="Send" />
    <input type="text" id="msg" size="64"/>
</form>
</body>
</html>　```

こんな感じですね。ここではwebsocketからのメッセージをやりとりしているだけなんで、特に解説することもないと思います、
通信が切断されたらconn.oncloseが呼ばれるようになっています。

##WebSocketを使う（サーバー編）
さぁ実際にサーバー側を組み立てていきましょい！
