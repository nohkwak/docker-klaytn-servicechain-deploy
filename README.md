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
kni 정보와 key 정보를 업데이트한다. 
```
$ cp /klaytn/scripts/static-nodes.json /data/
$ cp /klaytn/keys/nodekey2 /data/klay/nodekey
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
