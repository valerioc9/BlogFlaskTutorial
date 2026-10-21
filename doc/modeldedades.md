# Model de dades

## Model Entitat Relació E/R

```mermaid

erDiagram
    USER ||--o{ POST : writes
    USER {
        INTEGER id PK
        TEXT username
        TEXT password
    }
    POST {
        INTEGER id PK
        INTEGER author_id FK
        TIMESTAMP created
        TEXT title
        TEXT body
    }
```

Quin tipus de Base de dades utilitza l´aplicació?