# memo

##### FastAPI

同期、非同期のパフォーマンスの違いがよくわかる例  
同期  
```
import time
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    time.sleep(5)
    return {"message": "Hello World"}
```
非同期  
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

