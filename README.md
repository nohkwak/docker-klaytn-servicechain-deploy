# 4 SCN on Docker 
[Klaytn Service Chain](https://ko.docs.klaytn.com/node/service-chain) 노드 4개를 Docker에 띄우기 위한 스크립트 및 설정 파일

<img width="416" alt="스크린샷 2022-03-09 오후 9 30 37" src="https://user-images.githubusercontent.com/2010763/157442455-c0eefcba-05a7-4a3a-b37c-bf26238b5dfb.png">


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

$ ./homi setup local --cn-num 4 --test-num 1 --servicechain --chainID 1002 --serviceChainID 1012 --p2p-port 22323 -o homi-output
```


## SCN 4개 노드 구성

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
        "kni://...@172.16.239.11:22321?discport=0\u0026ntype=cn",
        "kni://...@172.16.239.12:22322?discport=0\u0026ntype=cn",
        "kni://...@172.16.239.13:22323?discport=0\u0026ntype=cn",
        "kni://...@172.16.239.14:22324?discport=0\u0026ntype=cn"
]
```


### Docker 실행 및 확인
```
$ docker-compose up

$ docker ps
CONTAINER ID   IMAGE                  COMMAND     CREATED         STATUS         PORTS          NAMES
001650532e00   klaytn/klaytn:latest   "/bin/sh"   6 minutes ago   Up 6 minutes   0.0.0.0:22323->22323/tcp, :::22323->22323/tcp, 32323/tcp, 32323/udp, 0.0.0.0:8553->8551/tcp, :::8553->8551/tcp, 0.0.0.0:8557->8552/tcp, :::8557->8552/tcp, 0.0.0.0:50503->50505/tcp, :::50503->50505/tcp, 0.0.0.0:50507->50506/tcp, :::50507->50506/tcp, 0.0.0.0:61003->61001/tcp, :::61003->61001/tcp   kscn4_SCN-L2-03_1
a51f28cdebbb   klaytn/klaytn:latest   "/bin/sh"   6 minutes ago   Up 6 minutes   32323/tcp, 32323/udp, 0.0.0.0:8554->8551/tcp, :::8554->8551/tcp, 0.0.0.0:8558->8552/tcp, :::8558->8552/tcp, 0.0.0.0:22324->22323/tcp, :::22324->22323/tcp, 0.0.0.0:50504->50505/tcp, :::50504->50505/tcp, 0.0.0.0:50508->50506/tcp, :::50508->50506/tcp, 0.0.0.0:61004->61001/tcp, :::61004->61001/tcp   kscn4_SCN-L2-04_1
552b205dd008   klaytn/klaytn:latest   "/bin/sh"   6 minutes ago   Up 6 minutes   0.0.0.0:8551->8551/tcp, :::8551->8551/tcp, 0.0.0.0:61001->61001/tcp, :::61001->61001/tcp, 32323/tcp, 32323/udp, 0.0.0.0:8555->8552/tcp, :::8555->8552/tcp, 0.0.0.0:22321->22323/tcp, :::22321->22323/tcp, 0.0.0.0:50501->50505/tcp, :::50501->50505/tcp, 0.0.0.0:50505->50506/tcp, :::50505->50506/tcp   kscn4_SCN-L2-01_1
ac9d164cbb87   klaytn/klaytn:latest   "/bin/sh"   6 minutes ago   Up 6 minutes   32323/tcp, 32323/udp, 0.0.0.0:50506->50506/tcp, :::50506->50506/tcp, 0.0.0.0:8552->8551/tcp, :::8552->8551/tcp, 0.0.0.0:8556->8552/tcp, :::8556->8552/tcp, 0.0.0.0:22322->22323/tcp, :::22322->22323/tcp, 0.0.0.0:50502->50505/tcp, :::50502->50505/tcp, 0.0.0.0:61002->61001/tcp, :::61002->61001/tcp   kscn4_SCN-L2-02_1
```


### 접속 방법
```
$ docker exec -it kscn4_SCN-L2-01_1 bash
$ docker exec -it kscn4_SCN-L2-02_1 bash
$ docker exec -it kscn4_SCN-L2-03_1 bash
$ docker exec -it kscn4_SCN-L2-04_1 bash
```


### 초기화
각각의 노드를 초기화 한다. genesis.json에서 test account의 잔고를 충분히 큰 값으로 변경해 놓으면 테스트 시 용이하다.  
```
$ kscn   --datadir  /data   init  /homi-output/scripts/genesis.json
$ ls /data 
keystore  klay      kscn
```


### 설정 파일 복사
각각의 노드[1..4]에 static-nodes.json 파일과 nodekey[1..4]파일를 각각 복사한다. 
```
$ cp /homi-output/scripts/static-nodes.json /data/
$ cp /homi-output/keys/nodekey[1..4] /data/klay/nodekey
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



### AMI 저장
추후 반복잡업을 없애고자 여기까지 진행한 이미지([ami-05f533c07efc6c532](https://ap-northeast-2.console.aws.amazon.com/ec2/v2/home?region=ap-northeast-2#ImageDetails:imageId=ami-05f533c07efc6c532))를 AWS에 저장하였다.  
