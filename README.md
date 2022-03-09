# 4 SCN on Docker 
[Klaytn Service Chain](https://ko.docs.klaytn.com/node/service-chain) 노드 4개를 Docker에 띄우기 위한 스크립트 및 설정 파일


## Prerequisites
Followings are required.

1. [Docker](https://docs.docker.com/get-docker/)
2. [Docker-compose](https://docs.docker.com/compose/install/)
3. [homi](https://ko.docs.klaytn.com/node/download)


## homi 실행으로 설정 파일 생성
```
$ tar xvf homi-vX.X.X-XXXXX-amd64.tar.gz
homi-linux-amd64/
homi-linux-amd64/bin/
homi-linux-amd64/bin/homi

$ cd homi-linux-amd64/bin

$ ./homi setup local --cn-num 4 --test-num 1 --servicechain --chainID 1002 --p2p-port 22323 -o homi-output
```


## SCN 4개 노드 구성
![무제](https://user-images.githubusercontent.com/2010763/154245501-c30d87ce-ec8f-4ce4-b049-8388b7cd9430.png)
### 작업 디렉토리 생성 및 파일 이동 
```
$ mkdir ~/kscn4
$ cd ~/kscn4
$ mv docker-compose.yaml  ./
$ mv homi-output/         ./
```


### kni 정보 업데이트 
homi-output/scripts/static-nodes.json 파일에서 각 노드 IP 및 Port 정보를 업데이트한다. 
```
[
        "kni://...@172.16.239.10:22323?discport=0\u0026ntype=cn",
        "kni://...@172.16.239.11:22323?discport=0\u0026ntype=cn",
        "kni://...@172.16.239.12:22323?discport=0\u0026ntype=cn",
        "kni://...@172.16.239.13:22323?discport=0\u0026ntype=cn"
]
```


### Docker 실행 및 확인
```
$ docker-compose up

$ docker ps
CONTAINER ID        IMAGE                                      COMMAND                  CREATED             STATUS              PORTS                                                                                                                                                NAMES
30b7500b8f20        klaytn/klaytn:latest                       "/bin/sh"                16 minutes ago      Up 16 minutes       0.0.0.0:22323->22323/tcp, 32323/tcp, 32323/udp, 0.0.0.0:8553->8551/tcp, 0.0.0.0:8558->8552/tcp, 0.0.0.0:50503->50506/tcp, 0.0.0.0:61003->61001/tcp   kscn4_SCN-3_1
214ef7a6c3a4        klaytn/klaytn:latest                       "/bin/sh"                16 minutes ago      Up 16 minutes       32323/tcp, 32323/udp, 0.0.0.0:8552->8551/tcp, 0.0.0.0:8557->8552/tcp, 0.0.0.0:22322->22323/tcp, 0.0.0.0:50502->50506/tcp, 0.0.0.0:61002->61001/tcp   kscn4_SCN-2_1
c38630f7b0e0        klaytn/klaytn:latest                       "/bin/sh"                16 minutes ago      Up 16 minutes       32323/tcp, 32323/udp, 0.0.0.0:8550->8551/tcp, 0.0.0.0:8555->8552/tcp, 0.0.0.0:22320->22323/tcp, 0.0.0.0:50500->50506/tcp, 0.0.0.0:61000->61001/tcp   kscn4_SCN-0_1
6563416ad2c5        klaytn/klaytn:latest                       "/bin/sh"                16 minutes ago      Up 16 minutes       0.0.0.0:8551->8551/tcp, 0.0.0.0:61001->61001/tcp, 32323/tcp, 32323/udp, 0.0.0.0:8556->8552/tcp, 0.0.0.0:22321->22323/tcp, 0.0.0.0:50501->50506/tcp   kscn4_SCN-1_1
```


### 접속 방법
```
$ sudo docker exec -it kscn4_SCN-0_1 bash
$ sudo docker exec -it kscn4_SCN-1_1 bash
$ sudo docker exec -it kscn4_SCN-2_1 bash
$ sudo docker exec -it kscn4_SCN-3_1 bash
```


### 초기화
각각의 노드를 초기화 한다. 
```
$ kscn   --datadir  /data   init  /klaytn/scripts/genesis.json
$ ls /data 
keystore  klay      kscn
```


### 설정 파일 복사
각각의 노드[1..4]에 static-nodes.json 파일과 nodekey[1..4]파일를 각각 복사한다. 
```
$ cp /klaytn/scripts/static-nodes.json /data/
$ cp /klaytn/keys/nodekey[1..4] /data/klay/nodekey
```

/klaytn-docker-pkg/conf/kscnd.conf 파일에서 주요 정보를 업데이트한다. 
```
...
PORT=22323
...
SC_SUB_BRIDGE=0
...
DATA_DIR=/data
...
```


### 실행
```
$ kscnd start
Starting kscnd: OK
```


### 실행 상태 확인 방법
klay.blockNumber로 블록 생성 상태를 확인할 수 있으며, 이 숫자가 0이 아니면 노드가 제대로 동작하는 것이다.
```
$ kscn attach --datadir /data
> klay.blockNumber
10
```


### 중지
```
$ kscnd stop
```
