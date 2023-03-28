# tiflash-config

## Start Minio

```shell
brew install minio/stable/minio
minio server /tmp/minio_data --address :9222
```

Create a bucket named `tiflash-s3`:

```shell
mkdir -p /tmp/minio_data/tiflash-s3
```

## Start a Cluster with TiFlash Write Node

```shell
git clone https://github.com/breezewish/tiflash-cloud-native-tiup-config.git /tmp/tiflash-config

tiup playground nightly \
  --db.config /tmp/tiflash-config/tidb.toml \
  --tiflash.config /tmp/tiflash-config/tiflash-wn.toml
```

## Start a TiFlash Read Node

```shell
tiup tiflash:nightly server \
  --config-file /tmp/tiflash-config/tiflash-rn.toml
```

## Play!

```shell
mysql --host 127.0.0.1 --port 4000 -u root test
```

```sql
create table orders (oid int key, uid int);
insert into orders values (1, 1);
insert into orders values (2, 1);

alter table orders set tiflash replica 1;

select * from information_schema.tiflash_replica;

set @@session.tidb_isolation_read_engines='tiflash';

select count(*) from orders;
```

## Cleanup

You must do clean-up if you want to repeat these steps again, because TiUP will only clean-up data of the TiDB cluster:

```shell
# Only need to clean TiFlash data.
# It's fine to leave minio data not cleaned.
rm -rf /tmp/tiflash
```
