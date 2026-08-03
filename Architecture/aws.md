```mermaid
flowchart LR

    PG[EC2<br/>PostgreSQL]

    PB[EC2<br/>PeerDB]

    CH[EC2<br/>ClickHouse]

    PG -- Private VPC --> PB
    PB -- CDC --> CH
```
