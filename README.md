# General

## Deployment type

Docker

## Image

Based on official image on Docker Hub: https://hub.docker.com/_/redis/

## Licence

BSD-3-Clause

## Version

6.2.4

## Description

Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache, and message broker. Redis provides data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs, geospatial indexes, and streams. Redis has built-in replication, Lua scripting, LRU eviction, transactions, and different levels of on-disk persistence, and provides high availability via Redis Sentinel and automatic partitioning with Redis Cluster.

# Deployment

General example:

```sh
docker run -d --rm \
        --name redis \
        -p 6379:6379 \
        -v $HOME/redis/data:/data \
        -v $HOME/redis/conf:/usr/local/etc/redis \
        redis:6.0.10 \
        redis-server --appendonly yes /usr/local/etc/redis/redis.conf
```

__Note__: "Redis is designed to be accessed by trusted clients inside trusted environments ... Redis only replies to queries from the loopback" - by default.

## Parameters

|Name|Value|Description|
|-|-|-|
|Ports| `-p 6379:6379` | Redis port |
|Volumes| `-v $HOME/redis/data:/data` | Persist Redis data |
|CLI ARG| `--appendonly yes` | Persist redis data to the disk |
|CLI ARG| `/usr/local/etc/redis/redis.conf` | Location of the Redis config |

<br/>
For additional configuration edit `/usr/local/etc/redis/redis.conf`. See [1] for more details.

## Testing

```sh
docker exec -it redis redis-cli

localhost:6379> quit
```


# Authentication

Set password in redis.conf:

```sh
mkdir -p redis/conf
cat > redis/conf/redis.conf <<EOF
requirepass myredispassword
appendonly yes
EOF
```

See [2] for more information about the security.

## ACL

The Redis ACL, short for Access Control List, is the feature that allows certain connections to be limited in terms of the commands that can be executed and the keys that can be accessed.

Default ACL file location: `/etc/redis/users.acl`

See [3] for more information about the ACL feature.

## Testing

```sh
docker exec -it redis redis-cli

localhost:6379> AUTH myredispassword

OK
```

# TLS

By default, Redis uses mutual TLS and requires clients to authenticate with a valid certificate (authenticated against trusted root CAs specified by ca-cert-file or ca-cert-dir).

You may use `tls-auth-clients no` to disable client authentication. [4]

Run gen-test-certs.sh [5] or follow the example below to generate a root CA and a server certificate. Files: `ca.crt`, `ca.key`, `redis.crt`, `redis.dh`, `redis.key` will be generated in `./certs` directory.

To create server certificates (`ca.crt`, `server.key`, `server.crt`), execute the following, replace the subject (value of `-subj` parameter) as needed:

```sh
openssl genrsa -out certs/ca.key 4096
openssl req -x509 -new -nodes -sha256 \
        -key certs/ca.key \
        -days 3650 \
        -subj '/O=Redis Test/CN=Certificate Authority' \
        -out certs/ca.crt

openssl genrsa -out certs/redis.key 2048
openssl req -new -sha256 \
        -key certs/redis.key \
        -subj '/O=Redis Test/CN=Server' openssl x509 -req -sha256 \
        -CA certs/ca.crt \
        -CAkey certs/ca.key \
        -CAserial certs/ca.txt \
        -CAcreateserial \
        -days 365 \
        -out certs/redis.crt
openssl dhparam -out certs/redis.dh 2048
```

```sh
cat > $HOME/redis/conf/redis.conf <<EOF
tls-auth-clients no
tls-port 6379
port 0
tls-cert-file /certs/redis.crt
tls-key-file /certs/redis.key
tls-ca-cert-file /certs/ca.crt
requirepass myredispassword
appendonly yes
EOF
```

## Deployment

```sh
docker run -d --rm \
        --name redis \
        -v $HOME/certs/:/certs \
        -v $HOME/redisconf:/usr/local/etc/redis \
        redis:6.0.10 \
        redis-server /usr/local/etc/redis/redis.conf
```

## Testing

```sh
docker exec -it redis redis-cli --tls --cacert /certs/ca.crt

127.0.0.1:6379> AUTH myredispassword

OK
```
# TLS mutual athentication

See TLS above, without line: `tls-auth-clients no` in config file (by default, Redis uses mutual TLS).


## Testing

```sh
docker exec -it redis \
        redis-cli --tls --cert /certs/redis.crt --key /certs/redis.key --cacert /certs/ca.crt

127.0.0.1:6379> AUTH myredispassword

OK
```

# References

[1] https://raw.githubusercontent.com/redis/redis/6.0/redis.conf

[2] https://redis.io/topics/security

[3] https://redis.io/topics/acl

[4] https://redis.io/topics/encryption

[5] https://github.com/redis/redis/blob/6.2/utils/gen-test-certs.sh