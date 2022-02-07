# 4 SCN on Docker 
Docker에 4개의 SCN을 설정하기 위한 스크립트 및 설정 파일

## Prerequisites
Followings are required.

1. [Docker](https://docs.docker.com/get-docker/)
2. [Docker-compose](https://docs.docker.com/compose/install/)
3. [kscn, homi](https://ko.docs.klaytn.com/node/download)

## 4개 노드 환경 구성
### 노드 실행 
```
$ sudo docker-compose up
```

### 확인 방법
```
$ sudo docker ps
CONTAINER ID        IMAGE                                      COMMAND                  CREATED             STATUS              PORTS                                                                                                                       NAMES
433da604b56b        klaytn/klaytn:latest                       "/bin/sh"                22 seconds ago      Up 17 seconds       8552/tcp, 32323/udp, 0.0.0.0:8552->8551/tcp, 0.0.0.0:32322->32323/tcp, 0.0.0.0:50502->50506/tcp, 0.0.0.0:61002->61001/tcp   kscn4_SCN-2_1
a50f5ae56d17        klaytn/klaytn:latest                       "/bin/sh"                22 seconds ago      Up 16 seconds       8552/tcp, 32323/udp, 0.0.0.0:8550->8551/tcp, 0.0.0.0:32320->32323/tcp, 0.0.0.0:50500->50506/tcp, 0.0.0.0:61000->61001/tcp   kscn4_SCN-0_1
66b64edd71bb        klaytn/klaytn:latest                       "/bin/sh"                22 seconds ago      Up 16 seconds       0.0.0.0:8551->8551/tcp, 0.0.0.0:61001->61001/tcp, 8552/tcp, 32323/udp, 0.0.0.0:32321->32323/tcp, 0.0.0.0:50501->50506/tcp   kscn4_SCN-1_1
28d5e6c3b54c        klaytn/klaytn:latest                       "/bin/sh"                22 seconds ago      Up 16 seconds       8552/tcp, 32323/udp, 0.0.0.0:32323->32323/tcp, 0.0.0.0:8553->8551/tcp, 0.0.0.0:50503->50506/tcp, 0.0.0.0:61003->61001/tcp   kscn4_SCN-3_1
```

### 접속 방법
```
$ sudo docker exec -it kscn4_SCN-0_1 bash
$ sudo docker exec -it kscn4_SCN-1_1 bash
$ sudo docker exec -it kscn4_SCN-2_1 bash
$ sudo docker exec -it kscn4_SCN-3_1 bash
```

## SCN 구성 
### homi 실행으로 설정 파일 생성
```
$ ./homi setup local --cn-num 4 --test-num 1 --servicechain --p2p-port 30000 -o homi-output
$ scp -r path/to/homi-output/ user@address:~/ 
```

### 초기화
```
$ kscn --datadir /data init /klaytn/scripts/genesis.json
$ ls /data 
keystore  klay      kscn
```

### 설정값 업데이트 
kni 정보와 key 정보 nodekey[1~4]를 각각 업데이트한다. 
```
$ cp /klaytn/scripts/static-nodes.json /data/
$ cp /klaytn/keys/nodekey[1~4] /data/klay/nodekey
```

kscn 설치 폴더(/klaytn-docker-pkg)로 이동하여 conf/kscnd.conf를 다음과 같이 업데이트한다. 
```
...
PORT=30000
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

### 접속 방법
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
