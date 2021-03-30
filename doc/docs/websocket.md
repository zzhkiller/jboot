# Websocket 

Jboot 内置了 Undertow 服务器，完美支持 websocket 功能， tomcat8 也是内置了对 websocket 的支持。

在使用 websocket 之前，需要添加如下配置：

```
jboot.web.webSocketEndpoint=your-endpoint-class-name
```

例如：

```
//多个 endpoint 用英文逗号（,） 隔开。
jboot.web.webSocketEndpoint=io.jboot.test.websocket.WebsocketDemo
```

WebsocketDemo 的代码如下：

```java
package io.jboot.test.websocket;

import javax.websocket.OnMessage;
import javax.websocket.Session;
import javax.websocket.server.ServerEndpoint;


@ServerEndpoint("/myapp.ws")
public class WebsocketDemo {

    @OnMessage
    public void message(String message, Session session) {
        for (Session s : session.getOpenSessions()) {
            System.out.println("receive : " + message);
            s.getAsyncRemote().sendText(message);
        }
    }
}
```
这里要注意：`@ServerEndpoint("/myapp.ws")` 中的内容 `/myapp.ws` 必须有后缀，后缀可以自定义，否则会被 JFinal 的拦截。

我们还需要写一个 Controller，来渲染 html 页面。

```java
@RequestMapping("/websocket")
public class WebsocketController extends JbootController {

    public void index() {
        render("/websocket.html");
    }

}
```

websocket.html 的内容如下：

```html
<html>
<head><title>Undertow Chat</title>
    <script>
        var socket;
        if (window.WebSocket) {
            // 这里的 /myapp.ws 必须和以上 @ServerEndpoint("/myapp.ws") 
            // 中配置的内容保持一致
            socket = new WebSocket("ws://127.0.0.1:8080/myapp.ws");
            socket.onmessage = function (event) {
                var chat = document.getElementById('chat');
                chat.innerHTML = chat.innerHTML + event.data + "<br />";
            };
        } else {
            alert("Your browser does not support Websockets. (Use Chrome)");
        }

        function send(message) {
            if (!window.WebSocket) {
                return false;
            }
            if (socket.readyState == WebSocket.OPEN) {
                socket.send(message);
            } else {
                alert("The socket is not open.");
            }
            return false;
        }
    </script>
    <style type="text/css">
        html,body {width:100%;height:100%;}
        html,body,ul,ol,dl,li,dt,dd,p,blockquote,fieldset,legend,img,form,h1,h2,h3,h4,h5,h6 {margin:0;padding:0;}
        body {
            font:normal 12px/1.5 Arial,Helvetica,'Bitstream Vera Sans',sans-serif;
            background: #c5deea; /* Old browsers */
            background: -moz-linear-gradient(top, #c5deea 0%, #066dab 100%); /* FF3.6+ */
            background: -webkit-gradient(linear, left top, left bottom, color-stop(0%, #c5deea), color-stop(100%, #066dab)); /* Chrome,Safari4+ */
            background: -webkit-linear-gradient(top, #c5deea 0%, #066dab 100%); /* Chrome10+,Safari5.1+ */
            background: -o-linear-gradient(top, #c5deea 0%, #066dab 100%); /* Opera 11.10+ */
            background: -ms-linear-gradient(top, #c5deea 0%, #066dab 100%); /* IE10+ */
            background: linear-gradient(to bottom, #c5deea 0%, #066dab 100%); /* W3C */
            height: 90%;
        }

        .center {
            margin-left: auto;
            margin-right: auto;
            width: 70%;
            background: white;
        }

        .chatform {
            margin-left: auto;
            margin-right: auto;
            margin-bottom: 0;
            width: 70%;
        }
        form{
            width: 100%;
        }
        label{
            display: inline;
            width: 100px;
        }
        #msg{
            display: inline;
            width: 100%;
        }
1
    </style>
</head>
<body>
<div class="page">
    <div class="center" >
        <h1>JSR-356 Web Socket Chat</h1>
        <div id="chat" style="height:100%;width: 100%; overflow: scroll;">


        </div>

        <form onsubmit="return false;" class="chatform" action="">
            <label for="msg">Message</label>
            <input type="text" name="message" id="msg"  
            onkeypress="if(event.keyCode==13) { send(this.form.message.value); this.value='' } " />
        </form>
    </div>
</div>
</body>
</html>
```
