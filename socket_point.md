## Websocket
Server side
```python
from fastapi import FastAPI, WebSocket,  Request, Form, WebSocketDisconnect

async def send_wsmsg(msg: str):
    for client in clients:
        await client.send_text(msg)

clients = []
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    # need await
    await websocket.accept()
    clients.append(websocket)
    try:
        while True:
            msg = await websocket.receive_text()
            print(msg)
            for client in clients:
                await client.receive_text(msg)
    
    # browser close
    except WebSocketDisconnect:
        print("client disconnected")
    finally:
        clients.remove(websocket)
        print("clients remove from connection list")

cnt = 0
@app.get("/countup")
async def countup():
    global cnt
    cnt += 1
    await send_wsmsg(str(cnt))
    return 200

```

Client side
```python
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>リアルタイム更新</title>
</head>
<body>
    <h1 id="message">サーバーからのメッセージがここに表示されます</h1>

    <script>
        const messageElement = document.getElementById("message");
        const ws = new WebSocket("ws://{{request.ip}}:{{request.port}}/ws");

        ws.onmessage = function(event) {
            // サーバーからのメッセージを表示
            messageElement.innerText = event.data;
        };

        ws.onclose = function() {
            console.log("WebSocket connection closed.");
        };
    </script>
</body>
</html>
```


## jinja2

install
```bash
pip install jinja2
```

Server side
```python
from fastapi.templating import Jinja2Templates

@app.get("/")
async def get():
    request = {"ip": "127.0.0.1", "port": 8001}
    return templates.TemplateResponse("index.html", {"request": request})
```

Client side
```html
function connect_ws() {
    const ws = new WebSocket("ws://{{request.ip}}:{{request.port}}/ws");
    ・・・・
}
```