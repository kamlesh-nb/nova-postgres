# nova-postgres

PostgreSQL wire-protocol driver for Nova (SCRAM auth, server-side prepared statements) on the async runtime. A Nova package — fetch with:

```sh
nova get https://github.com/kamlesh-nb/nova-postgres
```

```nova
import postgres;

let drv  = PgDriver();
let conn = await drv.connect("postgresql://user:pass@127.0.0.1:5432/mydb");
let rs   = await conn.query("SELECT id, name FROM users WHERE id = $1", params);
```

## Structure (SOLID)

The driver is split by responsibility rather than crammed into one file. Consumers only
touch the seam (`PgDriver` / `PgConnection`); the rest are internal modules.

| Module        | Responsibility                                                     |
|---------------|-------------------------------------------------------------------|
| `postgres`    | The seam: `PgConnection` (impl `Connection`) + `PgDriver` (impl `Driver`) + the connect/handshake orchestration. |
| `connection`      | Connection-string parsing (`ConnectionOptions`, `parse`).                   |
| `codec`    | The v3 wire codec — frame builders + response decoders + `PgCursor`. |
| `typemap`  | OID↔`DbType`, wire-text→`DbValue`, `DbValue`→SQL/bind text, `substituteParams`. |
| `proto`    | Async transport framing (`PgReader`, `PgFrame`, `readFrame`, `sendFrame`). |
| `stmt`     | Prepared-statement cache entry (`PgStmt`).                         |
| `auth`     | The SCRAM-SHA-256 / cleartext auth exchange.                       |

The pure halves (`codec` / `typemap`) are offline-gated (`tests/66_postgres_codec.nova`);
`connect`/`query` wire them to a socket and are live-verified against a running PostgreSQL.
