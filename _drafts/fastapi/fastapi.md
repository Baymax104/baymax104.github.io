---
layout: post
title: FastAPI基础
categories:
- Python
tags:
- Python
- FastAPI
---

## 开始

FastAPI是一个用于构建API的现代、快速（高性能）的 web 框架

FastAPI中有两个核心组件

-   Starlette：负责web处理，基于AnyIO实现异步处理
-   Pydantic：负责执行数据校验

### Python异步

Python通过`asyncio`模块实现异步代码，底层只存在一个线程，asyncio是"多任务合作"模式，允许异步任务交出执行权给其他任务，等到其他任务完成，再收回执行权继续往下执行

asyncio模块在单线程上启动一个事件循环（event loop），时刻监听新进入循环的事件，加以处理，并不断重复这个过程，直到异步任务结束

![bg2019112005](assets/bg2019112005-1721662637466-2.jpg)

### asyncio模块

asyncio模块配合`async`和`await`关键字使用

使用`async`关键字修饰一个函数，表示该函数是一个异步任务函数，使用`await`关键字调用一个异步任务函数，`await`只能在`async`函数中调用

```python
import asyncio
async def run_task():
    await asyncio.sleep(1)  # 休眠1秒
```

通过`asyncio.run()`函数来调用一个异步任务函数

```python
asyncio.run(run_task())
```

使用`asyncio.gather()`函数可以组合多个异步任务

```python
async def multi_task():
    await asyncio.gather(run_task(), run_task(), run_task())

asyncio.run(multi_task())
```

### 创建简单的FastAPI

```Python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```

使用`uvicorn main:app --reload`命令启动uvicorn服务器，默认在8000端口启动

访问`127.0.0.1:8000/docs`可以看到SwaggerUI生成的交互式API文档，访问`127.0.0.1:8000/redoc`可以看到ReDoc生成的可选的API文档

代码分析

1.   导入`FastAPI`类，`FastAPI`类提供了API的所有功能
2.   创建`FastAPI`的实例`app`，应用的所有API对通过`app`来创建和管理
3.   创建路径操作函数，使用`@<app_name>.<operation>(path)`装饰器来定义一个路径操作函数，满足该路径的请求会通过该函数处理
     -   `<app_name>`为创建的FastAPI实例的变量名
     -   `<operation>`为请求操作，同HTTP操作相同，如`@app.get`、`@app.post`、`@app.put`、`@app.delete`
     -   path为路径字符串，以`/`开头，`/`表示根路径

## 请求处理

### API参数

路径参数

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

-   路径参数用`{}`包围
-   声明路径参数对应的形参，使用类型标记可以自动转换为声明的类型，默认类型为str

通过自定义枚举类并继承str，可以实现枚举映射

```python
from enum import Enum

from fastapi import FastAPI

# 定义枚举类，继承str，可将str参数自动映射到对应的枚举值
class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"

app = FastAPI()

@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    return {"model_name": model_name, "message": "Have some residuals"}
```

当路径参数也表示一个路径，此时需要标记`:path`，参数类型为str

```python
from fastapi import FastAPI

app = FastAPI()

# file_path也需要以'/'开头
@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    # /files//home/johndoe/myfile.txt
    return {"file_path": file_path}
```

---

查询参数

对于未匹配到路径参数的形参，FastAPI会自动解释为查询参数，即URL中`?`后的键值对

```python
from fastapi import FastAPI

app = FastAPI()

# 支持默认值、可选参数
# 对于bool值，当参数不为空时，为True，否则为False
@app.get("/items")
async def read_item(item_id: int = 1, q: str | None = None, short: bool = False):
    item = {"item_id": item_id}
    return item
```

### 请求体

对于POST方法，数据通过请求体传输，此时需要通过Pydantic定义请求数据

```python
from fastapi import FastAPI
from pydantic import BaseModel

# 继承BaseModel类定义数据模型，并标记属性数据类型
"""
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2
}
"""
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

app = FastAPI()

@app.post("/items/")
async def create_item(item: Item):
    return item
```

一个BaseModel表示一个请求体参数，可以同时传输多个请求体参数

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

"""
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    },
    "user": {
        "username": "dave",
        "full_name": "Dave Grohl"
    }
}
"""
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

class User(BaseModel):
    username: str
    full_name: str | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, user: User):
    results = {"item_id": item_id, "item": item, "user": user}
    return results
```

当一个请求体中包含嵌套请求体和单个键值对，需要使用Body对象指明单个键值对参数

```python
from typing import Annotated

from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()

"""
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    },
    "user": {
        "username": "dave",
        "full_name": "Dave Grohl"
    },
    "importance": 5
}
"""
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

class User(BaseModel):
    username: str
    full_name: str | None = None

# importance参数使用Body对象指明
@app.put("/items/{item_id}")
async def update_item(
    item_id: int, item: Item, user: User, importance: Annotated[int, Body()]
):
    results = {"item_id": item_id, "item": item, "user": user, "importance": importance}
    return results
```

在请求体中只包含一个请求体参数时，需要指定Body对象的`embed=True`

```python
from typing import Annotated

from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()

"""
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    }
}
"""
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Annotated[Item, Body(embed=True)]):
    results = {"item_id": item_id, "item": item}
    return results
```

### 数据校验

查询参数与字符串校验

查询参数还支持更多的校验，通过Query对象实现，此时参数显式地声明为查询参数

```python
from typing import Union

from fastapi import FastAPI, Query

app = FastAPI()

# q为可选参数，default设置默认值为None
# q最大长度为50，最小长度为3，匹配正则表达式
@app.get("/items/")
async def read_items(
    q: Union[str, None] = Query(
        default=None, 
        min_length=3, 
        max_length=50, 
        pattern="^fixedquery$"
    ),
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

声明为必需参数

-   `q: str = Query(min_length=3)`：类型标记不包含None
-   `q: str = Query(default=..., min_length=3)`：使用省略号表示必需参数
-   `q: Union[str, None] = Query(default=..., min_length=3)`：表示该参数必须传递，即使它是None
-   `q: str = Query(default=Required, min_length=3)`：使用`pydantic.Required`声明必需参数

通过将查询参数标记为列表，可以使参数接收多个值

```python
from typing import List, Union

from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Union[List[str], None] = Query(default=None)):
    query_items = {"q": q}
    return query_items
```

Query的其他参数

-   Query可以接收自定义的键值对，通过参数获取
-   设置`alias`参数可以设置URL参数的别名
-   设置`deprecated=True`表示该参数将被弃用

---

路径参数与数值校验

路径参数通过Path对象实现校验，Path对象具有和Query对象相同的参数，Path参数总是必需的

```python
from typing import Annotated

from fastapi import FastAPI, Path, Query

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="The ID of the item to get")],
    q: Annotated[str | None, Query(alias="item-query")] = None,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

数值校验

Path和Query中包含指定数值范围的参数

-   `ge=<value>`：大于等于
-   `gt`、`le`、`lt`

### Cookie参数

定义Cookie参数的方式与定义Query和Path参数相同

```python
from typing import Annotated

from fastapi import Cookie, FastAPI

app = FastAPI()

@app.get("/items/")
async def read_items(ads_id: Annotated[str | None, Cookie()] = None):
    return {"ads_id": ads_id}
```

### Header参数

定义Header参数的方式与定义Query、Path、Cookie参数相同

FastAPI的Header参数会自动将请求头中`User-Agent`形式的参数自动转换为`user_agent`形式

```python
from typing import Annotated

from fastapi import FastAPI, Header

app = FastAPI()

@app.get("/items/")
async def read_items(user_agent: Annotated[str | None, Header()] = None):
    return {"User-Agent": user_agent}
```

使用list标记可以以列表形式接收多个Header参数

```python
@app.get("/items/")
async def read_items(x_token: Annotated[list[str] | None, Header()] = None):
    return {"X-Token values": x_token}
```

## 响应处理

### 响应模型

使用Pydantic定义响应的数据模型，设置到路径装饰器的`response_model`参数，或直接标记为函数返回值类型

```python
from typing import Any
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: list[str] = []

# response_model参数接收响应模型
@app.post("/items/", response_model=Item)
async def create_item(item: Item) -> Any:
    return item

# response_model参数也接收模型的列表
@app.get("/items/", response_model=list[Item])
async def read_items() -> Any:
    return [
        {"name": "Portal Gun", "price": 42.0},
        {"name": "Plumbus", "price": 32.0},
    ]
    
# 接口返回值类型指明返回模型
@app.post("/items/")
async def create_item(item: Item) -> Item:
    return item
```

fastapi中内置了`Response`类及其子类，可以表示不同种类的响应，常用的有`JSONResponse`和`RedirectResponse`等，fastapi使用Pydantic创建并验证返回类型，因此不支持Union类型
