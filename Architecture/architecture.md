## Architecture

```mermaid
flowchart TD

    A[PostgreSQL 18<br/>AWS EC2]

    B[Logical Replication<br/>WAL]

    C[PeerDB<br/>Snapshot + CDC]

    D[MinIO]

    E[ClickHouse 26<br/>AWS EC2]

    A --> B
    B --> C
    C --> D
    D --> E
```
