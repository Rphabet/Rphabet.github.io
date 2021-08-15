---
title: MongoDB MAC OS 설치부터 실행까지
date: 2021-08-14 23:00:00 +0900
categories: [DBMS, MongoDB]
tags: [DBMS, SQL, MongoDB] 
math: true
comments: true
image:
  src: img/mongoDB_StartSever.png
  width: 1000   # in pixels
  height: 400   # in pixels
  alt: image alternative text

---

# MongDB Mac OS 설치부터 실행까지

---

관계형 DBMS가 아닌 NoSQL DBMS인 mongoDB의 특징까지 하나하나 열거하며 스스로 배웠던 부분에 대한 복습을 하고 싶지만, 우선 시간 부족이라는 좋은 핑계를 내세우며 간단하게 **맥 개발 환경 구축**정도만 설명하고 오늘의 포스팅을 마무리 하겠다.

## 맥 개발 환경 구축

Pre-requisite:

- mac OS 10.11 버전 이상이어야함
- homebrew를 통해 `terminal`로 설치할 것이기에 `brew` 가 깔려 있어야한다.

```bash
$ brew install node
$ brew install mongodb-community@5.0  # community 에디션 5.0 버전을 설치하라는 소리 (글을 쓴 시점의 최신 버전)
```

이렇게 커맨드를 치고 엔터를 **빵!** 치면 아래와 같은 결과를 볼 수 있을것이다.

```bash
(base) Bans-MacBook-Pro:~ bangirimben$ brew install mongodb-community@5.0
==> Installing mongodb-community from mongodb/brew
==> Downloading https://fastdl.mongodb.org/tools/db/mongodb-database-tools-macos-x86_64-100.
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/macos-term-size/manifests/1.0.0
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/macos-term-size/blobs/sha256:a19d9785c6b4d8
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sha256:a19d978
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/node/14/manifests/14.17.5
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/node/14/blobs/sha256:d5a953dc4cb682a7e5c9a0
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sha256:d5a953d
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/mongosh/manifests/1.0.5
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/mongosh/blobs/sha256:36c31f20e685f007af3812
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sha256:36c31f2
######################################################################## 100.0%
==> Downloading https://fastdl.mongodb.org/osx/mongodb-macos-x86_64-5.0.2.tgz
######################################################################## 100.0%
==> Installing dependencies for mongodb/brew/mongodb-community: mongodb-database-tools, macos-term-size, node@14 and mongosh
==> Installing mongodb/brew/mongodb-community dependency: mongodb-database-tools
🍺  /usr/local/Cellar/mongodb-database-tools/100.5.0: 13 files, 115.6MB, built in 6 seconds
==> Installing mongodb/brew/mongodb-community dependency: macos-term-size
==> Pouring macos-term-size--1.0.0.big_sur.bottle.tar.gz
🍺  /usr/local/Cellar/macos-term-size/1.0.0: 5 files, 52.6KB
==> Installing mongodb/brew/mongodb-community dependency: node@14
==> Pouring node@14--14.17.5.big_sur.bottle.tar.gz
🍺  /usr/local/Cellar/node@14/14.17.5: 4,377 files, 62MB
==> Installing mongodb/brew/mongodb-community dependency: mongosh
==> Pouring mongosh--1.0.5.big_sur.bottle.tar.gz
🍺  /usr/local/Cellar/mongosh/1.0.5: 5,217 files, 30.3MB
==> Installing mongodb/brew/mongodb-community
==> Caveats
To have launchd start mongodb/brew/mongodb-community now and restart at login:
  brew services start mongodb/brew/mongodb-community
Or, if you don't want/need a background service you can just run:
  mongod --config /usr/local/etc/mongod.conf
==> Summary
🍺  /usr/local/Cellar/mongodb-community/5.0.2: 11 files, 180MB, built in 7 seconds
==> Caveats
==> mongodb-community
To have launchd start mongodb/brew/mongodb-community now and restart at login:
  brew services start mongodb/brew/mongodb-community
Or, if you don't want/need a background service you can just run:
  mongod --config /usr/local/etc/mongod.conf
```

설치가 완료되었다면 `$ mongo -version` 을 입력하여 설치 완료 및 버전 확인을 해주자.

```bash
$ mongo -version 

MongoDB shell version v5.0.2
Build Info: {
    "version": "5.0.2",
    "gitVersion": "6d9ec525e78465dcecadcff99cce953d380fedc8",
    "modules": [],
    "allocator": "system",
    "environment": {
        "distarch": "x86_64",
        "target_arch": "x86_64"
    }
}
```

---

## MongoDB 실행

개발 환경을 구축했으니 MongoDB의 셸로 접속해야 앞으로 데이터를 조작하는 방법에 대해 배울 수 있다. 

셸에 접속하기 위해서는 우선 'mongod 인스턴스'가 실행되어야 한다.

지금 단계에서는 간단하게 `mongod`를 켜면 데이터베이스에 접속할 수 있다는 정도로 이해하고 넘어가자.

```bash
$ mongod
```

처음 MongoDB를 설치하면 문제가 발생했다면서 다음과 같은 문장이 등장한다.

```bash
exception in initAndListen: NonExistentPath: Data directory not found
```

MongoDB는 데이터를 기본적으로 `/data/db` 디렉토리에 저장하는데, 그 디렉토리가 생성되어 있지 않아서 나오는 에러 메시지다.

에러메시지에 나온 경로에 가서 디렉토리를 생성해주고 다시 한 번 실행을 해보자.

이제 `mongod` 명령어를 입력한 뒤 터미널 창을 하나 더 켜서 다음의 명령어를 입력하면 몽고 셸에 접속하게 된다.

```bash
$ mongo
```

필자는 처음에 `$ mongo` 를 입력했을 때 `connection failed` 에 직면하였는데, 

```bash
MongoDB shell version v5.0.2
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Error: couldn't connect to server 127.0.0.1:27017, connection attempt failed: SocketException: Error connecting to 127.0.0.1:27017 :: caused by :: Connection refused :
connect@src/mongo/shell/mongo.js:372:17
@(connect):2:6
exception: connect failed
exiting with code 1

```

음.. 간단하게 구글링해보니깐 macbook 사용자들 중에서 MongoDB 버전 업 즉 v4 --> v5 로 올렸을 때 생기는 흔하게 경험하는 에러라고 알려져있다. 

혹자는 지우고, 경로 재설정에 어마무시한(?) 작업들이 필요했다고 하는데.. 필자 또한 체념하고 지우고 다시 깔고 경로 재설정까지 고려하고 있었으나 귀신같이 `brew` 를 통해 실행시킬수 있었다.

```bash
(base) Bans-MacBook-Pro:db bangirimben$ brew services start mongodb-community@5.0

==> Successfully started `mongodb-community` (label: homebrew.mxcl.mongodb-com

(base) Bans-MacBook-Pro:db bangirimben$ mongo
MongoDB shell version v5.0.2
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("-----") }
MongoDB server version: 5.0.2
================
Warning: the "mongo" shell has been superseded by "mongosh",
which delivers improved usability and compatibility.The "mongo" shell has been deprecated and will be removed in
an upcoming release.
We recommend you begin using "mongosh".
For installation instructions, see
https://docs.mongodb.com/mongodb-shell/install/
================
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	https://docs.mongodb.com/
Questions? Try the MongoDB Developer Community Forums
	https://community.mongodb.com
---
The server generated these startup warnings when booting: 
        2021-08-14T22:55:38.707+09:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
---
---
        Enable MongoDB's free cloud-based monitoring service, which will then receive and display
        metrics about your deployment (disk utilization, CPU, operation statistics, etc).

        The monitoring data will be available on a MongoDB website with a unique URL accessible to you
        and anyone you share the URL with. MongoDB may use this information to make product
        improvements and to suggest MongoDB products and deployment options to you.

        To enable free monitoring, run the following command: db.enableFreeMonitoring()
        To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---
> 

```

다음장부터는 이렇게 mongo shell에 접속한 상태에서 진행해보자.

<img src="/Users/bangirimben/Rphabet.github.io/_posts/img/mongoDB_StartSever.png" alt="">

