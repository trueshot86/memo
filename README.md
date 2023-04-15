# memo

### FastAPI

同期、非同期のパフォーマンスの違いがよくわかる例  
##### 同期  
```
import time
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    time.sleep(5)
    return {"message": "Hello World"}
```
##### 非同期  
```
import asyncio
from fastapi import FastAPI

app = FastAPI()

async def square(number: int):
    await asyncio.sleep(5)
    return {"result": number ** 2}

@app.get("/async/{number}")
async def async_endpoint(number: int):
    result = await square(number)
    return result
```

例えば、3人が同時に同時にアクセスすると  
同期のパターンだと３人目のページが表示されるのは約15秒後  
非同期のパターンだと3人目のページが表示されるのは約5秒後  

##### 起動コマンド
```
uvicorn main:app --host 0.0.0.0 --port 8000
```

##### Jinja2テンプレート利用版
main.py
```
from fastapi import FastAPI, Request
from fastapi.templating import Jinja2Templates

app = FastAPI()
templates = Jinja2Templates(directory="templates")

from fastapi.responses import HTMLResponse

@app.get("/", response_class=HTMLResponse)
async def read_item(request: Request):
    return templates.TemplateResponse("index.html", {"request": request, "message": "Hello, world!"})
```

index.html
```
<!DOCTYPE html>
<html>
    <head>
        <title>{{ message }}</title>
    </head>
    <body>
        <h1>{{ message }}</h1>
        <p>Request URL: {{ request.url.path }}</p>
    </body>
</html>
```

#### get(path parameter,query parameter)
オプショナルにしたければ、デフォルトをNoneにする。  
必須にしたい場合は、デフォルト値を宣言しない。
```
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/users/{user_id}/items/{item_id}")
async def read_user_item(
    user_id: int, item_id: str, q: Union[str, None] = None, short: bool = False
):
    item = {"item_id": item_id, "owner_id": user_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item
```

##### post
```
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None


app = FastAPI()


@app.post("/items/")
async def create_item(item: Item):
    return item
```

form
```
from fastapi import FastAPI, Request, Form, Depends
from fastapi.templating import Jinja2Templates
from fastapi.middleware.csrf import CSRFMiddleware

app = FastAPI()
templates = Jinja2Templates(directory="templates")
app.add_middleware(CSRFMiddleware, secret_key="random_secret_key")

@app.get("/")
def index(request: Request):
    return templates.TemplateResponse("index.html", {"request": request})

@app.post("/form")
def form_post(request: Request, name: str = Form(...), age: int = Form(...), token: str = Depends()):
    return templates.TemplateResponse("result.html", {"request": request, "name": name, "age": age})
```

html
```
<form method="post">
  <input type="hidden" name="csrf_token" value="{{ csrf_token() }}">
  <input type="text" name="name">
  <input type="number" name="age">
  <button type="submit">Submit</button>
</form>
```
### Linux
/etc/systemd/system/xxxxx.service
##### systemd
```
[Unit]
Description=xxxxx.sh

[Service]
Type=simple
ExecStart=/bin/bash /root/xxxxx.sh

[Install]
WantedBy=multi-user.target
```
systemctl daemon-reload  

