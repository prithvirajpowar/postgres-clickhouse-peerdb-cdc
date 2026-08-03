```mermaid
flowchart TD

A[Application]

B[PostgreSQL]

C[WAL]

D[PeerDB]

E[ClickHouse]

F[Analytics Dashboard]

A --> B
B --> C
C --> D
D --> E
E --> F
```
