# bb-redis
Building block for Redis

# IMAGE SOURCE

Official Redis image on __Docker Hub__: https://hub.docker.com/_/redis/


# DEPLOYMENT

> docker run --name redis -v $HOME/redisdata:/data -d redis:6.0.10 redis-server --appendonly yes

__Note__: "Redis is designed to be accessed by trusted clients inside trusted environments ... Redis only replies to queries from the loopback" - by default.

## TEST

> docker exec -it redis redis-cli
>
> localhost:6379> quit


# AUTHENTICATION


Set password in __refis.conf__:

> mkdir redisconf
>
> vi $HOME/redisconf/redis.conf
>
>> __requirepass myredispassword__
>>
>> appendonly yes

Run with conf:

> docker run --name redis -v $HOME/redisdata:/data __-v $HOME/redisconf:/usr/local/etc/redis__ -d redis:6.0.10 redis-server /usr/local/etc/redis/redis.conf

See: https://redis.io/topics/security

For username and password see: https://redis.io/topics/acl (aclfile: /etc/redis/users.acl)


## TEST

> docker exec -it redis redis-cli
>
> localhost:6379> AUTH myredispassword
>
> OK

# TLS

__Note__: "By default, Redis uses mutual TLS ... use tls-auth-clients no to disable client authentication" (see: https://redis.io/topics/encryption).

Run gen-test-certs.sh to generate a root CA and a server certificate (see: https://download.redis.io/redis-stable/utils/). Files: __ca.crt__, ca.key, __redis.crt__, redis.dh, __redis.key__ will be generated in ./certs directory.

> $ vi $HOME/redisconf/redis.conf
>
>> __tls-auth-clients no__
>>
>> __tls-port 6379__
>> 
>> __port 0__
>>
>> __tls-cert-file /certs/redis.crt__
>> 
>> __tls-key-file /certs/redis.key__
>> 
>> __tls-ca-cert-file /certs/ca.crt__
>> 
>> requirepass myredispassword
>>
>> appendonly yes



> docker run --name redis __-v $HOME/certs/:/certs__ -v $HOME/redisconf:/usr/local/etc/redis -d redis:6.0.10 redis-server /usr/local/etc/redis/redis.conf

## TEST

> docker exec -it redis redis-cli --tls --cacert /certs/ca.crt
>
> 127.0.0.1:6379> AUTH myredispassword
>
> OK




# TLS mutual athentication 

See HTTPS above, without line: "tls-auth-clients no" in config file (by default, Redis uses mutual TLS).


## TEST

> docker exec -it redis redis-cli --tls __--cert /certs/redis.crt__ __--key /certs/redis.key__ --cacert /certs/ca.crt
>
> 127.0.0.1:6379> AUTH myredispassword
>
> OK


