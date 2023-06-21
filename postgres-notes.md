# Postgres notes

### Dynamic list of values to a query placeholder

Passing query parameters to raw SQL queries in Python using SQLAlchemy is quite trivial, but what about when you have to pass a dynamic list of values for `WHERE foo IN (...)`?

```python
import sqlalchemy
from sqlalchemy.sql import text

baz_vals = [1, 2, 3]

engine = sqlalchemy.create_engine('postgresql://myuser:mypassword@localhost/mydatabase')
con = engine.connect()
sql = text('SELECT foo FROM bar WHERE baz = ANY (:baz_vals)')
stmt = sql.bindparams(baz_vals=baz_vals)
con.execute(stmt)
result = con.fetchall()
```

Effectively, this query was executed in Postgres:

```sql
SELECT foo FROM bar WHERE baz = ANY (ARRAY[1,2,3])
```
  
### Stop a PG cluster in a service from another service

In docker-compose file:

```
command: |
  sh -c "
    psql -v ON_ERROR_STOP=on -f /data/import.sql
    status=\"$$?\"
    psql -c \"COPY (SELECT 1) TO PROGRAM 'pg_ctl stop --mode=smart --no-wait';\"
    exit \"$${status}\"
  "
```

See original idea (not by me) in [sujuikoDB](https://github.com/datarttu/sujuikoDB/blob/6c0e9e1bee4a5464864c014d0bf26439781f32b2/docker-compose.test.yml).
