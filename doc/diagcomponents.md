# Diagrama de Components

Descriure tots els components de l´aplicació

Components: db, _init_ , auth, blog

## db
Descripció: Funcions d´accés a BBDD
Mètodes:
- getbd() : Retorna connexió : Mètode que retorna la connexió a BBDD
- close_db() : Tanca la connexió BBDD

```mermaid

classDiagram
    class auth {
        <<Blueprint>>
        +register() View
        +login() View
        +logout() View
        +load_logged_in_user() Function
        +login_required(view) Decorator
    }

    class blog {
        <<Blueprint>>
        +index() View
        +create() View
        +update(id) View
        +delete(id) View
        +get_post(id, check_author) Post
    }

    class db {
        <<Mòdul>>
        +get_db() sqlite3.Connection
        +close_db(e) None
        +init_db() None
        +init_db_command() CLI_Command
    }

    class User {
        +INTEGER id PK
        +TEXT username UNIQUE
        +TEXT password
    }

    class Post {
        +INTEGER id PK
        +INTEGER author_id FK
    }

    auth --> db
    blog --> db
    User "1" <-- "*" Post : author
    blog ..> Post : utilitza
```