---
title: "GoWebsocket"
date: 2020-03-14T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- gin
- webSocket
---

# html页面

>测试使用

```html
<!DOCTYPE html>
<html>
<head>
    <title>abc</title>
</head>
<body>
<script type="text/javascript">
    var sock = null;
  	// 端口修改
    var wsuri = "ws://localhost:8080/a";

    window.onload = function() {
        sock = new WebSocket(wsuri);
        //建立连接后触发
        sock.onopen = function() {
            console.log(" 建立连接后触发 connected to " + wsuri);
        }
        // 关闭连接时候触发
        sock.onclose = function(e) {
            console.log("关闭连接时候触发 connection closed (" + e.code + ")");
        }
        // 收到消息后触发
        sock.onmessage = function(e) {
            console.log("收到消息后触发 message received: " + e.data);
        }
        //发生错误的时候触发
        sock.onerror=function (e) {
            console.log("发生错误时候触发"+wsuri)
        }
    };
    //如果sock被关闭掉了这里也会报错的
    function send() {
        var msg = document.getElementById('message').value;
        sock.send(msg);
    };
</script>
<h1>GoWebSocketDemo</h1>
<form>
    <p>
        Message: <input id="message" type="text" value="aa">
    </p>
</form>
<button onclick="send();">给服务器发送消息</button>
</body>
</html>
```



# go的http包实现

```go
package main

import (
	"fmt"
	"golang.org/x/net/websocket"
	"log"
	"net/http"
)

func main() {
	http.Handle("/", websocket.Handler(Echo))
	if err := http.ListenAndServe("127.0.0.1:9999", nil); err != nil {
		log.Fatal(err)
	}
}

func Echo(w *websocket.Conn) {
	for {
		var reply string
		if err := websocket.Message.Receive(w, &reply); err != nil {
			fmt.Println("err:", err)
			break
		}
		msg := "get:" + reply
		fmt.Println(msg)
		if err := websocket.Message.Send(w, msg); err != nil {
			fmt.Println("err:", err)
			break
		}
	}
}
```

# Gin实现

```go
package main

import (
	"github.com/gin-gonic/gin"
	"golang.org/x/net/websocket"
)

func websocketHandler(handler websocket.Handler) gin.HandlerFunc {
	return func(context *gin.Context) {
		if context.IsWebsocket() {
			handler.ServeHTTP(context.Writer, context.Request)
		} else {
			context.JSON(200, gin.H{
				"msg": "err",
			})
			context.Abort()
		}
	}
}

func websocketConn(conn *websocket.Conn) {
	websocket.Message.Send(conn, "aaa")
}

func main() {
	app := gin.Default()

	app.GET("/ws", websocketHandler(websocketConn))

	app.Run("80")
}
```

