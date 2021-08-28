# fastAPI-Notes
Notas del framework fastAPI

# Menú

- [dependencias Windows](#dependencias-Windows)
- [Ejecutar servidor de manera sencilla](#Ejecutar-servidor-de-manera-sencilla)
- [Rutas](#Rutas)
- [BBDD](#BBDD)
    - [Modelos](#Modelos)
    - [Validaciones](#Validaciones)
    - [Get con id](#Get-con-id)
    - [Delete](#Delete)


## dependencias Windows:

```txt
asgiref==3.4.1
click==8.0.1
colorama==0.4.4
fastapi==0.68.1
h11==0.12.0
httptools==0.2.0
mysqlclient==2.0.3
peewee==3.14.4
pydantic==1.8.2
python-dotenv==0.19.0
PyYAML==5.4.1
starlette==0.14.2
typing-extensions==3.10.0.0
uvicorn==0.15.0
watchgod==0.7
websockets==9.1
```

---

## Ejecutar servidor de manera sencilla

craer un modulo de ejecución, en esta caso `main.py`

```python
# main.py

from fastapi import FastAPI

app = FastAPI(title="Hola Mundo",
              desciption="Estamos en un prueba",
              version="1.0.1")
```

**Terminal**

> levantar el servidor

`.fastAPI>uvicorn main:app`

Cada vez que el servidor tenga cambios se recarga de manera automatica

`.fastAPI>uvicorn main:app --reload`

---

## Rutas

```python
# registro de primera ruta
# responde al HTTP
# sirve con todos los métodos para el protocolo HTTP: app.put(), app.post(), etc...
@app.get("/")
def index():
    return "Hola Mundo"

@app.get("/about")
def about():
    return "Estamos en la sección about de la web"
```

> Ejecutar los métodos de manera asíncrona

```python
@app.get("/")
async def index():
    return "Hola Mundo"
```

> Apartado de documentación

`http://localhost:8000/docs`

![docs](https://user-images.githubusercontent.com/62488915/131228432-d65f55b4-dc82-4a31-8046-a0c608a6341a.png)

---

## BBDD

**Ejemplo de CRUD**

se crea un nuevo módulo `database.py`

```python
# database.py

from peewee import *

database = MySQLDatabase(
    'fastapi',
    user='root', password='pass',
    host='localhost', port=3306
)
```

```python
# main.py

from fastapi import FastAPI

from database import database as connection

app = FastAPI(title="Hola Mundo",
              desciption="Estamos en un prueba",
              version="1.0.1")

@app.on_event('startup')
def startup():
    if connection.is_closed():
        connection.connect()

@app.on_event('shutdown')
def shutdown():
    if not connection.is_closed():
        connection.close()

# registro de primera ruta
# responde al HTTP
@app.get("/")
async def index():
    return "Hola Mundo"

@app.get("/about")
async def about():
    return "Estamos en la sección about de la web"
```

### Modelos

```python
# database.py

class User(Model):
    
    username = CharField(max_length=30)
    email = CharField(max_length=30) 
    
    def __str__(self) -> str:
        return self.username
    
    class Meta:
        database = database
        table_name = 'users'
```

**Usando el modelo**

```python
# main.py

from fastapi import FastAPI

from database import database as connection
from database import User

app = FastAPI(title="Hola Mundo",
              desciption="Estamos en un prueba",
              version="1.0.1")

@app.on_event('startup')
def startup():
    if connection.is_closed():
        connection.connect()
        
    connection.create_tables([User])

@app.on_event('shutdown')
def shutdown():
    if not connection.is_closed():
        connection.close()

# registro de primera ruta
# responde al HTTP
@app.get("/")
async def index():
    return "Hola Mundo"

@app.get("/about")
async def about():
    return "Estamos en la sección about de la web"

@app.post("/users")
async def create_user():
    return "Nuevo Usuario"
```

## Validaciones

```python
# schemas.py

from pydantic import BaseModel
from typing import Optional

class UserRequestModel(BaseModel):
    username: str
    email: Optional[str] = None # un campo opcional
```

```python
# main.py

from fastapi import FastAPI

from database import database as connection
from database import User

from schemas import UserRequestModel

app = FastAPI(title="Hola Mundo",
              desciption="Estamos en un prueba",
              version="1.0.1")

@app.on_event('startup')
def startup():
    if connection.is_closed():
        connection.connect()
        
    connection.create_tables([User])

@app.on_event('shutdown')
def shutdown():
    if not connection.is_closed():
        connection.close()

# registro de primera ruta
# responde al HTTP
@app.get("/")
async def index():
    return "Hola Mundo"

@app.get("/about")
async def about():
    return "Estamos en la sección about de la web"

@app.post("/users")
async def create_user(user_request: UserRequestModel):
    user = User.create(
        username=user_request.username,
        email=user_request.email
    )
    
    return user_request
```

**Posteando un nuevo usuario**

![post](https://user-images.githubusercontent.com/62488915/131231218-1e82fba9-57f7-4650-8caa-13a41a69d4f4.png)

---

## Get con id

```python
# main.py

@app.get("/users/{user_id}")
async def get_user(user_id):
    user = User.select().where(
        User.id == user_id
    ).first()
    
    if user:
        return UserResponseModel(
            id=user.id,
            username=user.username,
            email=user.email
        )
    else:
        return HTTPException(404, "User not found") 
```

## Delete

```python
# main.py

@app.delete("/users/{user_id}")
async def delete_user(user_id):
    user = User.select().where(
        User.id == user_id
    ).first()
    
    if user:
        user.delete_instance()
        return True
    else:
        return HTTPException(404, "User not found") 
```
