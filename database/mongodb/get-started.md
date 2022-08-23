# 시작하기

## Docker로 MongoDB 설치하기

```shell
docker run \
    --name mongodb \
    -d \
    -p 27017:27017 \
    -e MONGO_INITDB_ROOT_USERNAME=root \
    -e MONGO_INITDB_ROOT_PASSWORD=root \
    mongo
```

## MongoDB 접속하기

```shell
docker exec -it mongodb /bin/bash
```

root 계정으로 접속한다.

```shell
mongo -u root -p root
```
