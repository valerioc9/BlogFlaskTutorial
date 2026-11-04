# Diagrama base de dades
sequenceDiagram
    autonumber
    actor Usuario
    participant CLI as Flask CLI
    participant App as Flask App (flaskr)
    participant DB as Base de Datos (SQLite)
    participant Schema as schema.sql

    Usuario->>CLI: Ejecuta "$ flask --app flaskr init-db"
    CLI->>App: Busca el comando registrado "init-db"
    App->>App: Llama a la función init_db()
    App->>DB: Abre conexión con get_db()
    DB-->>App: Retorna la conexión
    App->>Schema: Lee el archivo schema.sql
    Schema-->>App: Retorna las sentencias SQL (DROP / CREATE TABLE)
    App->>DB: Ejecuta el script SQL con db.executescript()
    DB-->>App: Confirma la creación de las tablas
    App-->>CLI: Finaliza la ejecución de la función
    CLI-->>Usuario: Muestra mensaje de éxito (ej. "Initialized the database.")