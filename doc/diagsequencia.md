# Diagrames de seqüència

## Diagrama Init Database
```mermaid
sequenceDiagram
    autonumber
    actor Usuari
    participant CLI as Flask CLI
    participant App as Flask App (flaskr)
    participant DB as Base de Dades (SQLite)
    participant Schema as schema.sql

    Usuari->>CLI: Executa "$ flask --app flaskr init-db"
    CLI->>App: Cerca el comandament registrat "init-db"
    App->>App: Crida la funció init_db()
    App->>DB: Obre connexió amb get_db()
    DB-->>App: Retorna la connexió
    App->>Schema: Llegeix el fitxer schema.sql
    Schema-->>App: Retorna les sentències SQL (DROP / CREATE TABLE)
    App->>DB: Executa el script SQL amb db.executescript()
    DB-->>App: Confirma la creació de les taules
    App-->>CLI: Finalitza l'execució de la funció
    CLI-->>Usuari: Mostra missatge d'èxit ("Initialized the database.")
```
## Diagrama d'arrencada de l'aplicació
```mermaid
sequenceDiagram
    autonumber
    participant CLI as Flask CLI / Terminal
    participant Init as __init__.py (App Factory)
    participant DB as db.py (Mòdul DB)
    participant FlaskApp as Instància App Flask

    Note over Usuari, FlaskApp: Procés 1: Inicialització de l'Aplicació (flask run)
    Usuari->>CLI: Executa `flask run` (o `--app flaskr`)
    CLI->>Init: Cerca i crida `create_app()`
    activate Init
    Init->>FlaskApp: Crea la instància `Flask(__name__)`
    Init->>FlaskApp: Configura App (DATABASE, SECRET_KEY)
    Init->>DB: Crida `db.init_app(app)`
    activate DB
    DB->>FlaskApp: Enregistra `app.teardown_appcontext(close_db)`
    DB->>FlaskApp: Enregistra comanda CLI (`app.cli.add_command(init_db_command)`)
    deactivate DB
    Init->>FlaskApp: Enregistra Blueprint `auth` (`app.register_blueprint(auth.bp)`)
    Init->>FlaskApp: Enregistra Blueprint `blog` (`app.register_blueprint(blog.bp)`)
    Init-->>CLI: Retorna la instància `app` configurada
    deactivate Init
    CLI-->>Usuari: Servidor local arrancat i escoltant peticions
```
## Login

```mermaid
sequenceDiagram
    autonumber
    actor Usuari
    participant Navegador
    participant Auth as auth.py (Blueprint)
    participant DB as db.py (SQLite)
    participant Session as Session (Cookie)

    Note over Usuari, Session: 1. Sol·licitud de la pantalla de Login (GET)
    Usuari->>Navegador: Accedeix a /auth/login
    Navegador->>Auth: GET /auth/login
    activate Auth
    Auth-->>Navegador: Retorna plantilla HTML (login.html)
    deactivate Auth
    Navegador-->>Usuari: Mostra formulari de Login

    Note over Usuari, Session: 2. Enviament de credencials (POST)
    Usuari->>Navegador: Introdueix usuari, contrasenya i prem "Log In"
    Navegador->>Auth: POST /auth/login (username, password)
    activate Auth

    Auth->>DB: get_db().execute("SELECT * FROM user WHERE username = ?", (username,))
    activate DB
    DB-->>Auth: Retorna objecte 'user' (o None)
    deactivate DB

    alt L'usuari no existeix
        Auth-->>Navegador: Mostra error "Incorrect username."
    else Contrasenya incorrecta
        Auth->>Auth: check_password_hash(user['password'], password)
        Auth-->>Navegador: Mostra error "Incorrect password."
    else Autenticació correcta
        Auth->>Auth: check_password_hash() == True
        Auth->>Session: session.clear()
        Auth->>Session: session['user_id'] = user['id']
        Auth-->>Navegador: Redirigeix a / (index)
    end
    deactivate Auth

    Note over Usuari, Session: 3. Verificació de sessió en peticions posteriors
    Navegador->>Auth: Petició HTTP a qualsevol ruta
    activate Auth
    Auth->>Auth: @bp.before_app_request (load_logged_in_user)
    alt Si existeix session['user_id']
        Auth->>DB: SELECT * FROM user WHERE id = session['user_id']
        DB-->>Auth: Retorna dades de g.user
    else No hi ha sessió
        Auth->>Auth: g.user = None
    end
    deactivate Auth
```