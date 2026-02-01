# FastAPI - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **FastAPI版本**：0.115+
- **最新稳定版**：0.115.x
- **推荐版本**：0.115.0+（2024-2025最新版本）

### 学习难度
- **难度等级**：⭐⭐⭐ (中等)
- **预计学习时间**：20-30小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- Python 3.9+基础
- 异步编程（async/await）
- HTTP协议基础
- RESTful API概念
- Pydantic基础

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 理解FastAPI的核心概念和优势
- [ ] 掌握路由和请求处理
- [ ] 熟练使用Pydantic进行数据验证
- [ ] 掌握依赖注入系统
- [ ] 能够实现身份认证和授权
- [ ] 掌握数据库集成（SQLAlchemy）
- [ ] 能够处理文件上传和WebSocket
- [ ] 掌握中间件和CORS配置
- [ ] 能够编写测试和部署应用

## 📖 目录

1. [FastAPI简介](#1-fastapi简介)
2. [环境搭建](#2-环境搭建)
3. [基础路由](#3-基础路由)
4. [请求参数](#4-请求参数)
5. [请求体和响应模型](#5-请求体和响应模型)
6. [依赖注入](#6-依赖注入)
7. [身份认证和安全](#7-身份认证和安全)
8. [数据库集成](#8-数据库集成)
9. [中间件和CORS](#9-中间件和cors)
10. [文件上传和WebSocket](#10-文件上传和websocket)
11. [后台任务](#11-后台任务)
12. [测试](#12-测试)
13. [部署](#13-部署)
14. [最佳实践](#14-最佳实践)
15. [实战案例](#15-实战案例)

---

## 1. FastAPI简介

### 1.1 什么是FastAPI

FastAPI是一个现代、快速（高性能）的Web框架，用于基于标准Python类型提示构建API。

**核心特性**：
- 🔥 **极快的性能**：与NodeJS和Go相当的性能
- 🔥 **快速开发**：开发速度提升200-300%
- 🔥 **更少的bug**：减少约40%的人为错误
- 🔥 **直观**：强大的编辑器支持，自动补全
- 🔥 **简单**：易于学习和使用
- 🔥 **简短**：减少代码重复
- 🔥 **健壮**：生产级代码，自动交互式文档
- 🔥 **基于标准**：基于OpenAPI和JSON Schema

### 1.2 为什么选择FastAPI

**与其他框架对比**：

| 特性 | FastAPI | Flask | Django |
|------|---------|-------|--------|
| 性能 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| 异步支持 | ✅ 原生 | ⭕ 部分 | ⭕ 部分 |
| 类型提示 | ✅ 完整 | ❌ 无 | ❌ 无 |
| 自动文档 | ✅ 自动 | ❌ 手动 | ❌ 手动 |
| 数据验证 | ✅ Pydantic | ❌ 手动 | ✅ Forms |
| 学习曲线 | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |

### 1.3 应用场景

- **AI/ML API服务**：模型推理API、RAG应用后端
- **微服务架构**：高性能微服务
- **实时应用**：WebSocket、SSE
- **数据API**：RESTful API、GraphQL
- **企业应用**：后台管理系统API

---

## 2. 环境搭建

### 2.1 安装FastAPI

```bash
# 🔥 安装FastAPI和ASGI服务器
pip install "fastapi[standard]"

# 或分别安装
pip install fastapi
pip install "uvicorn[standard]"

# 可选：安装其他依赖
pip install python-multipart  # 文件上传
pip install python-jose[cryptography]  # JWT
pip install passlib[bcrypt]  # 密码哈希
pip install sqlalchemy  # 数据库ORM
pip install alembic  # 数据库迁移
```

### 2.2 创建第一个应用

```python
# main.py
from fastapi import FastAPI

# 🔥 创建FastAPI实例
app = FastAPI(
    title="我的API",
    description="这是一个示例API",
    version="1.0.0"
)

@app.get("/")
async def root():
    """根路径"""
    return {"message": "Hello World"}

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    """读取单个项目"""
    return {"item_id": item_id}
```

### 2.3 运行应用

```bash
# 🔥 使用uvicorn运行
uvicorn main:app --reload

# 参数说明：
# main: 文件名（main.py）
# app: FastAPI实例名称
# --reload: 开发模式，代码改动自动重载

# 访问：
# API: http://127.0.0.1:8000
# 交互式文档: http://127.0.0.1:8000/docs
# 替代文档: http://127.0.0.1:8000/redoc
```

---

## 3. 基础路由

### 3.1 HTTP方法

```python
from fastapi import FastAPI

app = FastAPI()

# 🔥 GET请求
@app.get("/items/")
async def read_items():
    return {"message": "GET请求"}

# 🔥 POST请求
@app.post("/items/")
async def create_item():
    return {"message": "POST请求"}

# 🔥 PUT请求
@app.put("/items/{item_id}")
async def update_item(item_id: int):
    return {"message": f"PUT请求，更新项目{item_id}"}

# 🔥 DELETE请求
@app.delete("/items/{item_id}")
async def delete_item(item_id: int):
    return {"message": f"DELETE请求，删除项目{item_id}"}

# 🔥 PATCH请求
@app.patch("/items/{item_id}")
async def patch_item(item_id: int):
    return {"message": f"PATCH请求，部分更新项目{item_id}"}
```

### 3.2 路径操作配置

```python
from fastapi import FastAPI, status
from typing import Set

app = FastAPI()

# 🔥 配置路径操作
@app.post(
    "/items/",
    response_model=dict,
    status_code=status.HTTP_201_CREATED,
    tags=["items"],
    summary="创建项目",
    description="创建一个新的项目",
    response_description="创建成功的项目"
)
async def create_item():
    """
    创建项目的详细说明：
    
    - **name**: 项目名称
    - **price**: 项目价格
    """
    return {"name": "新项目", "price": 100}
```

---

## 4. 请求参数

### 4.1 路径参数

```python
from fastapi import FastAPI, Path
from typing import Annotated

app = FastAPI()

# 🔥 基础路径参数
@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}

# 🔥 带验证的路径参数
@app.get("/items/{item_id}")
async def read_item(
    item_id: Annotated[int, Path(
        title="项目ID",
        description="要获取的项目ID",
        ge=1,  # 大于等于1
        le=1000  # 小于等于1000
    )]
):
    return {"item_id": item_id}

# 🔥 枚举路径参数
from enum import Enum

class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"

@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name == ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}
    return {"model_name": model_name, "message": "Have some residuals"}
```

### 4.2 查询参数

```python
from fastapi import FastAPI, Query
from typing import Annotated

app = FastAPI()

# 🔥 基础查询参数
@app.get("/items/")
async def read_items(skip: int = 0, limit: int = 10):
    return {"skip": skip, "limit": limit}

# 🔥 可选查询参数
@app.get("/items/{item_id}")
async def read_item(item_id: int, q: str | None = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}

# 🔥 带验证的查询参数
@app.get("/items/")
async def read_items(
    q: Annotated[str | None, Query(
        title="查询字符串",
        description="用于搜索的查询字符串",
        min_length=3,
        max_length=50,
        pattern="^[a-zA-Z0-9]+$"
    )] = None
):
    return {"q": q}

# 🔥 查询参数列表
@app.get("/items/")
async def read_items(
    q: Annotated[list[str] | None, Query()] = None
):
    return {"q": q}
```

### 4.3 使用Pydantic模型定义查询参数

```python
from fastapi import FastAPI, Query
from pydantic import BaseModel, Field
from typing import Annotated, Literal

app = FastAPI()

# 🔥 查询参数模型（推荐方式）
class FilterParams(BaseModel):
    limit: int = Field(100, gt=0, le=100, description="每页数量")
    offset: int = Field(0, ge=0, description="偏移量")
    order_by: Literal["created_at", "updated_at"] = "created_at"
    tags: list[str] = []

@app.get("/items/")
async def read_items(
    filter_query: Annotated[FilterParams, Query()]
):
    return {
        "limit": filter_query.limit,
        "offset": filter_query.offset,
        "order_by": filter_query.order_by,
        "tags": filter_query.tags
    }
```

---

## 5. 请求体和响应模型

### 5.1 请求体（Pydantic模型）

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field, EmailStr
from typing import Annotated

app = FastAPI()

# 🔥 定义Pydantic模型
class Item(BaseModel):
    name: str = Field(..., min_length=1, max_length=100, description="项目名称")
    description: str | None = Field(None, max_length=500)
    price: float = Field(..., gt=0, description="价格必须大于0")
    tax: float | None = None
    tags: list[str] = []

# 🔥 使用请求体
@app.post("/items/")
async def create_item(item: Item):
    return item

# 🔥 混合路径、查询和请求体参数
@app.put("/items/{item_id}")
async def update_item(
    item_id: int,
    item: Item,
    q: str | None = None
):
    result = {"item_id": item_id, **item.model_dump()}
    if q:
        result.update({"q": q})
    return result
```


### 5.2 响应模型

```python
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()

class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: str | None = None

class UserOut(BaseModel):
    username: str
    email: EmailStr
    full_name: str | None = None

# 🔥 使用响应模型（自动过滤敏感信息）
@app.post("/users/", response_model=UserOut)
async def create_user(user: UserIn):
    return user  # password不会被返回

# 🔥 响应模型列表
@app.get("/users/", response_model=list[UserOut])
async def read_users():
    return [
        {"username": "user1", "email": "user1@example.com"},
        {"username": "user2", "email": "user2@example.com"}
    ]

# 🔥 响应状态码
from fastapi import status

@app.post("/items/", status_code=status.HTTP_201_CREATED)
async def create_item(name: str):
    return {"name": name}
```

---

## 6. 依赖注入

### 6.1 基础依赖

```python
from fastapi import FastAPI, Depends
from typing import Annotated

app = FastAPI()

# 🔥 定义依赖函数
async def common_parameters(
    skip: int = 0,
    limit: int = 100
):
    return {"skip": skip, "limit": limit}

# 🔥 使用依赖
@app.get("/items/")
async def read_items(
    commons: Annotated[dict, Depends(common_parameters)]
):
    return commons

@app.get("/users/")
async def read_users(
    commons: Annotated[dict, Depends(common_parameters)]
):
    return commons
```

### 6.2 类作为依赖

```python
from fastapi import FastAPI, Depends
from typing import Annotated

app = FastAPI()

# 🔥 类依赖
class CommonQueryParams:
    def __init__(self, skip: int = 0, limit: int = 100):
        self.skip = skip
        self.limit = limit

@app.get("/items/")
async def read_items(
    commons: Annotated[CommonQueryParams, Depends()]
):
    return {
        "skip": commons.skip,
        "limit": commons.limit
    }
```

### 6.3 子依赖

```python
from fastapi import FastAPI, Depends, Header, HTTPException
from typing import Annotated

app = FastAPI()

# 🔥 子依赖
async def verify_token(x_token: Annotated[str, Header()]):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")

async def verify_key(x_key: Annotated[str, Header()]):
    if x_key != "fake-super-secret-key":
        raise HTTPException(status_code=400, detail="X-Key header invalid")
    return x_key

# 🔥 使用多个依赖
@app.get("/items/", dependencies=[Depends(verify_token), Depends(verify_key)])
async def read_items():
    return [{"item": "Portal Gun"}, {"item": "Plumbus"}]
```

---

## 7. 身份认证和安全

### 7.1 OAuth2密码流程

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel
from typing import Annotated

app = FastAPI()

# 🔥 OAuth2配置
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None

# 模拟数据库
fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "fakehashedsecret",
        "disabled": False,
    }
}

def fake_hash_password(password: str):
    return "fakehashed" + password

# 🔥 获取当前用户依赖
async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    user = fake_users_db.get(token)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials",
            headers={"WWW-Authenticate": "Bearer"},
        )
    return User(**user)

# 🔥 登录端点
@app.post("/token")
async def login(
    form_data: Annotated[OAuth2PasswordRequestForm, Depends()]
):
    user_dict = fake_users_db.get(form_data.username)
    if not user_dict:
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    
    hashed_password = fake_hash_password(form_data.password)
    if not hashed_password == user_dict["hashed_password"]:
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    
    return {"access_token": user_dict["username"], "token_type": "bearer"}

# 🔥 受保护的端点
@app.get("/users/me")
async def read_users_me(
    current_user: Annotated[User, Depends(get_current_user)]
):
    return current_user
```

### 7.2 JWT认证

```python
from datetime import datetime, timedelta, timezone
from typing import Annotated
import jwt
from jwt.exceptions import InvalidTokenError
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from passlib.context import CryptContext
from pydantic import BaseModel

# 🔥 JWT配置
SECRET_KEY = "your-secret-key-keep-it-secret"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

app = FastAPI()

# 🔥 密码哈希
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    username: str | None = None

class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None

class UserInDB(User):
    hashed_password: str

# 模拟数据库
fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "$2b$12$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6Lruj3vjPGga31lW",
        "disabled": False,
    }
}

def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password):
    return pwd_context.hash(password)

def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)

def authenticate_user(fake_db, username: str, password: str):
    user = get_user(fake_db, username)
    if not user:
        return False
    if not verify_password(password, user.hashed_password):
        return False
    return user

# 🔥 创建JWT token
def create_access_token(data: dict, expires_delta: timedelta | None = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.now(timezone.utc) + expires_delta
    else:
        expire = datetime.now(timezone.utc) + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

# 🔥 获取当前用户
async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
        token_data = TokenData(username=username)
    except InvalidTokenError:
        raise credentials_exception
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception
    return user

# 🔥 登录端点
@app.post("/token")
async def login_for_access_token(
    form_data: Annotated[OAuth2PasswordRequestForm, Depends()]
) -> Token:
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )
    return Token(access_token=access_token, token_type="bearer")

# 🔥 受保护的端点
@app.get("/users/me/", response_model=User)
async def read_users_me(
    current_user: Annotated[User, Depends(get_current_user)]
):
    return current_user
```


---

## 8. 数据库集成

### 8.1 SQLAlchemy配置

```python
# database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

# 🔥 数据库URL
SQLALCHEMY_DATABASE_URL = "sqlite:///./sql_app.db"
# SQLALCHEMY_DATABASE_URL = "postgresql://user:password@postgresserver/db"

# 🔥 创建引擎
engine = create_engine(
    SQLALCHEMY_DATABASE_URL,
    connect_args={"check_same_thread": False}  # 仅SQLite需要
)

# 🔥 会话工厂
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# 🔥 基类
Base = declarative_base()

# 🔥 依赖函数
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### 8.2 定义模型

```python
# models.py
from sqlalchemy import Boolean, Column, ForeignKey, Integer, String
from sqlalchemy.orm import relationship
from database import Base

# 🔥 用户模型
class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True)
    hashed_password = Column(String)
    is_active = Column(Boolean, default=True)

    items = relationship("Item", back_populates="owner")

# 🔥 项目模型
class Item(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, index=True)
    description = Column(String, index=True)
    owner_id = Column(Integer, ForeignKey("users.id"))

    owner = relationship("User", back_populates="items")
```

### 8.3 Pydantic Schemas

```python
# schemas.py
from pydantic import BaseModel

# 🔥 Item Schemas
class ItemBase(BaseModel):
    title: str
    description: str | None = None

class ItemCreate(ItemBase):
    pass

class Item(ItemBase):
    id: int
    owner_id: int

    class Config:
        from_attributes = True

# 🔥 User Schemas
class UserBase(BaseModel):
    email: str

class UserCreate(UserBase):
    password: str

class User(UserBase):
    id: int
    is_active: bool
    items: list[Item] = []

    class Config:
        from_attributes = True
```

### 8.4 CRUD操作

```python
# crud.py
from sqlalchemy.orm import Session
import models, schemas

# 🔥 获取用户
def get_user(db: Session, user_id: int):
    return db.query(models.User).filter(models.User.id == user_id).first()

def get_user_by_email(db: Session, email: str):
    return db.query(models.User).filter(models.User.email == email).first()

def get_users(db: Session, skip: int = 0, limit: int = 100):
    return db.query(models.User).offset(skip).limit(limit).all()

# 🔥 创建用户
def create_user(db: Session, user: schemas.UserCreate):
    fake_hashed_password = user.password + "notreallyhashed"
    db_user = models.User(email=user.email, hashed_password=fake_hashed_password)
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user

# 🔥 获取项目
def get_items(db: Session, skip: int = 0, limit: int = 100):
    return db.query(models.Item).offset(skip).limit(limit).all()

# 🔥 创建项目
def create_user_item(db: Session, item: schemas.ItemCreate, user_id: int):
    db_item = models.Item(**item.model_dump(), owner_id=user_id)
    db.add(db_item)
    db.commit()
    db.refresh(db_item)
    return db_item
```

### 8.5 API端点

```python
# main.py
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.orm import Session
from typing import Annotated
import crud, models, schemas
from database import SessionLocal, engine, get_db

# 🔥 创建数据库表
models.Base.metadata.create_all(bind=engine)

app = FastAPI()

# 🔥 创建用户
@app.post("/users/", response_model=schemas.User)
def create_user(
    user: schemas.UserCreate,
    db: Annotated[Session, Depends(get_db)]
):
    db_user = crud.get_user_by_email(db, email=user.email)
    if db_user:
        raise HTTPException(status_code=400, detail="Email already registered")
    return crud.create_user(db=db, user=user)

# 🔥 获取用户列表
@app.get("/users/", response_model=list[schemas.User])
def read_users(
    skip: int = 0,
    limit: int = 100,
    db: Annotated[Session, Depends(get_db)]
):
    users = crud.get_users(db, skip=skip, limit=limit)
    return users

# 🔥 获取单个用户
@app.get("/users/{user_id}", response_model=schemas.User)
def read_user(
    user_id: int,
    db: Annotated[Session, Depends(get_db)]
):
    db_user = crud.get_user(db, user_id=user_id)
    if db_user is None:
        raise HTTPException(status_code=404, detail="User not found")
    return db_user

# 🔥 为用户创建项目
@app.post("/users/{user_id}/items/", response_model=schemas.Item)
def create_item_for_user(
    user_id: int,
    item: schemas.ItemCreate,
    db: Annotated[Session, Depends(get_db)]
):
    return crud.create_user_item(db=db, item=item, user_id=user_id)

# 🔥 获取项目列表
@app.get("/items/", response_model=list[schemas.Item])
def read_items(
    skip: int = 0,
    limit: int = 100,
    db: Annotated[Session, Depends(get_db)]
):
    items = crud.get_items(db, skip=skip, limit=limit)
    return items
```

---

## 9. 中间件和CORS

### 9.1 CORS配置

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# 🔥 配置CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000", "https://example.com"],  # 允许的源
    allow_credentials=True,  # 允许携带凭证
    allow_methods=["*"],  # 允许的HTTP方法
    allow_headers=["*"],  # 允许的HTTP头
)

@app.get("/")
async def main():
    return {"message": "Hello World"}
```

### 9.2 自定义中间件

```python
from fastapi import FastAPI, Request
import time

app = FastAPI()

# 🔥 自定义中间件
@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

---

## 10. 文件上传和WebSocket

### 10.1 文件上传

```python
from fastapi import FastAPI, File, UploadFile
from fastapi.responses import HTMLResponse

app = FastAPI()

# 🔥 单文件上传
@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile):
    contents = await file.read()
    return {
        "filename": file.filename,
        "content_type": file.content_type,
        "size": len(contents)
    }

# 🔥 多文件上传
@app.post("/uploadfiles/")
async def create_upload_files(files: list[UploadFile]):
    return [{"filename": file.filename} for file in files]

# 🔥 保存文件
@app.post("/upload/")
async def upload_file(file: UploadFile):
    with open(f"uploads/{file.filename}", "wb") as buffer:
        content = await file.read()
        buffer.write(content)
    return {"filename": file.filename, "message": "File uploaded successfully"}
```

### 10.2 WebSocket

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from fastapi.responses import HTMLResponse

app = FastAPI()

html = """
<!DOCTYPE html>
<html>
    <head>
        <title>Chat</title>
    </head>
    <body>
        <h1>WebSocket Chat</h1>
        <form action="" onsubmit="sendMessage(event)">
            <input type="text" id="messageText" autocomplete="off"/>
            <button>Send</button>
        </form>
        <ul id='messages'>
        </ul>
        <script>
            var ws = new WebSocket("ws://localhost:8000/ws");
            ws.onmessage = function(event) {
                var messages = document.getElementById('messages')
                var message = document.createElement('li')
                var content = document.createTextNode(event.data)
                message.appendChild(content)
                messages.appendChild(message)
            };
            function sendMessage(event) {
                var input = document.getElementById("messageText")
                ws.send(input.value)
                input.value = ''
                event.preventDefault()
            }
        </script>
    </body>
</html>
"""

@app.get("/")
async def get():
    return HTMLResponse(html)

# 🔥 WebSocket端点
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    try:
        while True:
            data = await websocket.receive_text()
            await websocket.send_text(f"Message text was: {data}")
    except WebSocketDisconnect:
        print("Client disconnected")
```


---

## 11. 后台任务

### 11.1 基础后台任务

```python
from fastapi import FastAPI, BackgroundTasks
import time

app = FastAPI()

# 🔥 定义后台任务
def write_log(message: str):
    time.sleep(5)  # 模拟耗时操作
    with open("log.txt", "a") as log:
        log.write(f"{message}\n")

# 🔥 使用后台任务
@app.post("/send-notification/{email}")
async def send_notification(
    email: str,
    background_tasks: BackgroundTasks
):
    background_tasks.add_task(write_log, f"Notification sent to {email}")
    return {"message": "Notification sent in the background"}

# 🔥 多个后台任务
@app.post("/process/")
async def process_data(
    background_tasks: BackgroundTasks
):
    background_tasks.add_task(write_log, "Task 1")
    background_tasks.add_task(write_log, "Task 2")
    background_tasks.add_task(write_log, "Task 3")
    return {"message": "Processing started"}
```

---

## 12. 测试

### 12.1 测试配置

```python
# test_main.py
from fastapi.testclient import TestClient
from main import app

# 🔥 创建测试客户端
client = TestClient(app)

# 🔥 测试GET请求
def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello World"}

# 🔥 测试POST请求
def test_create_item():
    response = client.post(
        "/items/",
        json={"name": "Test Item", "price": 10.5}
    )
    assert response.status_code == 200
    assert response.json()["name"] == "Test Item"

# 🔥 测试认证
def test_read_items_with_auth():
    response = client.get(
        "/items/",
        headers={"Authorization": "Bearer fake-token"}
    )
    assert response.status_code == 200
```

### 12.2 数据库测试

```python
# test_database.py
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from database import Base, get_db
from main import app

# 🔥 测试数据库
SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False})
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base.metadata.create_all(bind=engine)

def override_get_db():
    try:
        db = TestingSessionLocal()
        yield db
    finally:
        db.close()

app.dependency_overrides[get_db] = override_get_db

client = TestClient(app)

def test_create_user():
    response = client.post(
        "/users/",
        json={"email": "test@example.com", "password": "testpass"}
    )
    assert response.status_code == 200
    assert response.json()["email"] == "test@example.com"
```

---

## 13. 部署

### 13.1 使用Uvicorn部署

```bash
# 🔥 生产环境运行
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4

# 使用Gunicorn + Uvicorn Workers
gunicorn main:app --workers 4 --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

### 13.2 Docker部署

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# 🔥 安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY . .

# 暴露端口
EXPOSE 8000

# 🔥 启动应用
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/dbname
    depends_on:
      - db
  
  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=dbname
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### 13.3 环境变量配置

```python
# config.py
from pydantic_settings import BaseSettings

# 🔥 使用Pydantic Settings
class Settings(BaseSettings):
    app_name: str = "My API"
    admin_email: str
    database_url: str
    secret_key: str
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 30

    class Config:
        env_file = ".env"

settings = Settings()
```

```bash
# .env
ADMIN_EMAIL=admin@example.com
DATABASE_URL=postgresql://user:password@localhost/dbname
SECRET_KEY=your-secret-key-here
```

---

## 14. 最佳实践

### 14.1 项目结构

```
my_fastapi_project/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI应用入口
│   ├── config.py            # 配置文件
│   ├── dependencies.py      # 全局依赖
│   ├── models/              # SQLAlchemy模型
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── item.py
│   ├── schemas/             # Pydantic schemas
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── item.py
│   ├── crud/                # CRUD操作
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── item.py
│   ├── api/                 # API路由
│   │   ├── __init__.py
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   ├── users.py
│   │   │   └── items.py
│   │   └── deps.py          # API依赖
│   ├── core/                # 核心功能
│   │   ├── __init__.py
│   │   ├── security.py      # 安全相关
│   │   └── config.py
│   └── db/                  # 数据库
│       ├── __init__.py
│       ├── base.py
│       └── session.py
├── tests/                   # 测试
│   ├── __init__.py
│   ├── test_users.py
│   └── test_items.py
├── alembic/                 # 数据库迁移
├── .env                     # 环境变量
├── .gitignore
├── requirements.txt
├── Dockerfile
└── docker-compose.yml
```

### 14.2 错误处理

```python
from fastapi import FastAPI, HTTPException, Request
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError

app = FastAPI()

# 🔥 自定义异常
class ItemNotFoundException(Exception):
    def __init__(self, item_id: int):
        self.item_id = item_id

# 🔥 异常处理器
@app.exception_handler(ItemNotFoundException)
async def item_not_found_exception_handler(
    request: Request,
    exc: ItemNotFoundException
):
    return JSONResponse(
        status_code=404,
        content={"message": f"Item {exc.item_id} not found"}
    )

# 🔥 验证错误处理
@app.exception_handler(RequestValidationError)
async def validation_exception_handler(
    request: Request,
    exc: RequestValidationError
):
    return JSONResponse(
        status_code=422,
        content={"detail": exc.errors()}
    )
```

### 14.3 日志配置

```python
import logging
from fastapi import FastAPI, Request
import time

# 🔥 配置日志
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

app = FastAPI()

# 🔥 请求日志中间件
@app.middleware("http")
async def log_requests(request: Request, call_next):
    start_time = time.time()
    
    logger.info(f"Request: {request.method} {request.url}")
    
    response = await call_next(request)
    
    process_time = time.time() - start_time
    logger.info(f"Response: {response.status_code} - {process_time:.2f}s")
    
    return response
```

### 14.4 性能优化

```python
from fastapi import FastAPI
from fastapi_cache import FastAPICache
from fastapi_cache.backends.redis import RedisBackend
from fastapi_cache.decorator import cache
from redis import asyncio as aioredis

app = FastAPI()

# 🔥 Redis缓存配置
@app.on_event("startup")
async def startup():
    redis = aioredis.from_url("redis://localhost")
    FastAPICache.init(RedisBackend(redis), prefix="fastapi-cache")

# 🔥 使用缓存
@app.get("/items/{item_id}")
@cache(expire=60)  # 缓存60秒
async def read_item(item_id: int):
    # 模拟耗时操作
    return {"item_id": item_id, "name": "Item"}
```

---

## 15. 实战案例

### 15.1 AI模型API服务

```python
from fastapi import FastAPI, File, UploadFile
from pydantic import BaseModel
import torch
from transformers import pipeline

app = FastAPI(title="AI Model API")

# 🔥 加载模型
classifier = pipeline("sentiment-analysis")

class PredictionResponse(BaseModel):
    label: str
    score: float

# 🔥 文本分类API
@app.post("/predict/", response_model=PredictionResponse)
async def predict_sentiment(text: str):
    result = classifier(text)[0]
    return PredictionResponse(
        label=result["label"],
        score=result["score"]
    )

# 🔥 图像分类API
@app.post("/classify-image/")
async def classify_image(file: UploadFile = File(...)):
    contents = await file.read()
    # 处理图像并进行分类
    return {"filename": file.filename, "prediction": "cat"}
```

### 15.2 RAG应用后端

```python
from fastapi import FastAPI, Depends
from pydantic import BaseModel
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_core.vectorstores import InMemoryVectorStore
from typing import Annotated

app = FastAPI(title="RAG API")

# 🔥 初始化组件
embeddings = OpenAIEmbeddings()
vectorstore = InMemoryVectorStore(embeddings)
llm = ChatOpenAI(model="gpt-4o-mini")

class QueryRequest(BaseModel):
    question: str

class QueryResponse(BaseModel):
    answer: str
    sources: list[str]

# 🔥 添加文档
@app.post("/documents/")
async def add_document(text: str):
    vectorstore.add_texts([text])
    return {"message": "Document added successfully"}

# 🔥 查询API
@app.post("/query/", response_model=QueryResponse)
async def query(request: QueryRequest):
    # 检索相关文档
    docs = vectorstore.similarity_search(request.question, k=3)
    
    # 构建上下文
    context = "\n".join([doc.page_content for doc in docs])
    
    # 生成回答
    prompt = f"基于以下上下文回答问题：\n\n{context}\n\n问题：{request.question}"
    answer = llm.invoke(prompt).content
    
    return QueryResponse(
        answer=answer,
        sources=[doc.page_content for doc in docs]
    )
```

---

## 📝 学习检查清单

- [ ] 理解FastAPI的核心概念和优势
- [ ] 能够创建基础的API端点
- [ ] 掌握路径参数、查询参数和请求体
- [ ] 熟练使用Pydantic进行数据验证
- [ ] 理解依赖注入系统
- [ ] 能够实现JWT认证
- [ ] 掌握SQLAlchemy数据库集成
- [ ] 了解中间件和CORS配置
- [ ] 能够处理文件上传和WebSocket
- [ ] 掌握测试和部署方法

---

## 🔗 相关资源

- [FastAPI官方文档](https://fastapi.tiangolo.com/)
- [FastAPI GitHub](https://github.com/tiangolo/fastapi)
- [Pydantic文档](https://docs.pydantic.dev/)
- [SQLAlchemy文档](https://docs.sqlalchemy.org/)
- [Uvicorn文档](https://www.uvicorn.org/)

---

@author erik.zhou
