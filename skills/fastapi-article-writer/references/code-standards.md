# 代码示例规范

**必须提供可直接运行的完整代码**。

## 1. 完整的文件结构

在文章中展示项目结构：

```
项目结构：
main.py          # 入口文件
models.py        # 数据模型
requirements.txt # 依赖
```

## 2. 完整的依赖列表

```txt
fastapi==0.104.1
uvicorn[standard]==0.24.0
pydantic==2.5.0
# 其他必要依赖
```

## 3. 代码编写规范

### 类型注解必须完整

```python
from typing import Optional, List
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str
    email: Optional[str] = None

def get_user(user_id: int) -> Optional[User]:
    """根据ID获取用户信息"""
    ...
```

### 渐进式展示（错误 → 正确 → 优化）

每段代码必须展示演进过程：

**错误写法（问题代码）：**
```python
@app.get("/items")
def get_items():  # 缺少类型注解
    items = fetch_items()  # 没有错误处理
    return items
```

**正确写法（基础版本）：**
```python
@app.get("/items")
def get_items() -> List[Item]:
    try:
        items = fetch_items()
        return items
    except Exception as e:
        raise HTTPException(500, str(e))
```

**优化写法（生产版本）：**
```python
@app.get("/items", response_model=StandardResponse[List[Item]])
async def get_items(
    page: int = Query(1, ge=1),
    size: int = Query(20, ge=1, le=100)
) -> StandardResponse[List[Item]]:
    """
    获取物品列表

    Args:
        page: 页码，从1开始
        size: 每页数量，1-100

    Returns:
        StandardResponse: 标准响应格式
    """
    try:
        items = await fetch_items(page=page, size=size)
        return StandardResponse(data=items)
    except DatabaseError as e:
        logger.error(f"获取物品列表失败: {e}")
        raise HTTPException(500, "数据库查询失败")
```

### 必含错误处理

所有代码必须包含 try-except 块：

```python
try:
    result = await some_operation()
except SpecificException as e:
    logger.error(f"具体错误: {e}")
    raise HTTPException(status_code=400, detail="具体错误信息")
except Exception as e:
    logger.error(f"意外错误: {e}")
    raise HTTPException(status_code=500, detail="服务器内部错误")
```

### 配置参数集中定义

```python
# config.py 或代码开头
class Config:
    DATABASE_URL = "postgresql://user:pass@localhost/db"
    REDIS_URL = "redis://localhost:6379/0"
    SECRET_KEY = os.getenv("SECRET_KEY", "default-key")
    MAX_RETRY = 3
    TIMEOUT = 30
```

## 4. 真实场景示例

- 用实际的业务场景：用户注册、订单查询、文章发布
- 避免 foo、bar 这类无意义命名
- 数据用真实格式：手机号、邮箱、日期等

## 5. 注释规范

函数注释需包含功能、参数、返回值、异常：

```python
async def create_order(
    order: OrderCreate,
    db: AsyncSession = Depends(get_db)
) -> Order:
    """
    创建新订单

    Args:
        order: 订单创建请求体
        db: 数据库会话

    Returns:
        Order: 创建的订单对象

    Raises:
        HTTPException: 库存不足或参数错误时抛出
    """
    # 检查库存
    stock = await check_stock(order.product_id)
    if stock < order.quantity:
        raise HTTPException(400, "库存不足")

    # 创建订单
    new_order = Order(**order.dict())
    db.add(new_order)
    await db.commit()

    return new_order
```
