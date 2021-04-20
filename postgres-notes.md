# Postgres notes

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
