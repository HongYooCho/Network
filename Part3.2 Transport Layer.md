# Transport Layer Part 2
* 데이터를 보낸다고 해서 무조건 받는 것이 아니므로 어플리케이션 층에 있는 프로세스의 데이터가 어떻게 목적지까지 신뢰성 있게 도달하는 것을 보장할 것인가가 중요하다.
### Reliable Data Transfer
* 어떤 상태에서 다른 상태로 넘어가기 위해서는 액션 (이벤트)이 필요하다.
* RDT를 시작하면서…
- Sender 측
 rdt_send(): 어플리케이션 층에서 콜하는 것. 데이터를 전송 층으로 옮기기
<br> udt_send(): 전송층에서 네트워크 레이어로 보내는 것. (Unreliable channel로)
- Receiver 측
rdt_rcv(): 네트워크 층에서 전송 층으로 데이터를 받는 것
<br> deliver_data(): 데이터를 어플리케이션 레이어로 올리는 것
* 어떤 것들이 Unreliable channel로 만들까?
1. 비트에러: 도착했는데 보낸 데이터와 다름
2. 패킷 드랍, 로스: 도착도 못함
3. 중복된 패킷의 도착
- 위 3가지가 rdt 프로토콜의 복잡성을 결정한다.
### RDT 1.0
* 전제 조건 1) 비트에러가 없고 2) 패킷 로스도 없다
- 즉 걍 보내면 받고 잃어버리지도 않는 완벽 그 자체인 링크 상태
![q]( https://raw.githubusercontent.com/HongYooCho/Network/master/Photo/rdt1.0.PNG)

* 돌아가는 순서
1. 초기 상태에서 rdt_send(data)라는 이벤트가 발생.
2. 어플리케이션 층에서 트랜스포트 층으로 데이터가 넘어가게 되고 패킷을 만들고 네트워크 계층으로 넘긴다.
3. 여기 조건에서는 걍 바로 받으므로 패킷에서 데이터를 추출하고 데이터를 어플리케이션 층으로 올린다.
### RDT 2.0
* bit error가 있는 채널. 이걸 어떻게 리커버리할 것인가?
* 리커버리 하는 방법: 고치는 게 아니라 retransmission을 한다.
* Ack과 NACK이 존재
* 이 두가지 메시지를 통해 다시 전송할지 안 할지 결정
![q]( https://raw.githubusercontent.com/HongYooCho/Network/master/Photo/rdt2.0.PNG)
1. 1과는 다르게 패킷에 첵섬을 같이 넣음
2. 보낸 후에 액과 낵을 기다리는 상태로 넘어감(stop and wait)
3. 리시버 측에서 잘 받았고 오류가 없으면 어플리케이션 층에 딜리버 하고 액을 보낸다.
4. 만약 둘중 하나라도 만족 시키지 못한다면 낵을 보낸다
5. 다시 발신자 측 기다리는 상태에서 낵이면 재전송 액이면 대기 상태에 있는다.
- 여기서 stop and wati이란 발신자가 한 패킷을 보내고 응답을 기다리는 상황을 뜻함
- checksum을 보내주기만 하지 checksum 자체의 오류 여부는 알수 없다.
### RDT 2.0의 오류
* 액과 낵 자체의 오류를 해결할 방법과 수신자 측의 상황을 알 수가 없다.
* 그러므로 수신자 측이 보낸 메시지가 액인지 낵인지 모르는 상황 발생(옵션 2가지)
1. 발신자 측에서는 아무 행동을 하지 않는다.
- 원래 메시지가 낵이면 데이터를 아예 받지 못하는 상황 -> 큰일나는 상황
2. 걍 뭐든 간에 재전송을 해주는 방식
- 액일 경우 중복되는 데이터를 받게 된다.
* 그러므로 옵션 2를 사용하는 데 Sequence Number를 붙혀서 보냄으로써 해결하자
 ![q]( https://raw.githubusercontent.com/HongYooCho/Network/master/Photo/rdt2.1.PNG)
![q]( https://raw.githubusercontent.com/HongYooCho/Network/master/Photo/rdt2.1.1.PNG)

1. 2와 다른 점: 패킷에 시퀀스 넘버도 추가 됨
2. 패킷을 보내 놓고 발신자 측은 대기
3. 송신자 측은 패킷이 왔나, 데이터에 오류가 없나에 추가로 시퀀스 넘버가 있는지를 확인한다.
3.1 맞으면 액과 첵섬을 같이 보내고 다음 번호를 기다리는 상태.
3.2 틀리면 낵과 첵섬을 같이 보내고 다음 번호를 기다리는 상태.
4. 첵섬을 보고 메시지에 문제가 있는 지 확인한다
5. 재전송을 하고 다시 대기상태.
6. 수신자 측에서 중복된 것을 받은 것을 알고 액과 책섬을 다시 보낸다.
7. 수신자 측은 계속 다음 넘버를 기다리고 있는 상태
8. 발신자 측은 액을 받고 다음 넘버를 보낼 준비를 한다.
- 근데 꼭 액과 낵 이 두 메시지가 필요할까? -> 시퀀스 넘버로도 해결이 가능하다.
### RDT 2.2
* 2.1에서 낵을 없애자!
* 낵 대신에 리시버는 지난 패킷을 잘받았다는 액을 보내자
- 예) 0번을 보내고 0번 액을 기다리는 데 1번 액이 온다 –> 이건 낵을 의미
* 수신자에게서 중복 액이 오면 낵으로 처리한다가 핵심
![q]( https://raw.githubusercontent.com/HongYooCho/Network/master/Photo/rdt2.2.PNG)
1. 1번에 대한 액을 보낸다.
2. 발신자 측에 도착하고 0번 패킷을 만들고 보낸다.
3. 근데 0번 패킷에서 오류가 났다.
4. 수신자는 오류가 난 것을 확인한다.
5. 1번 패킷에 대한 액을 보낸다.
6. 발신자 측은 0번을 기다리는 상태인데 1번이 오는 것을 보고 중복액이 왔으니 낵으로 처리
7. 아까 만들었던 0번 패킷을 다시 재전송한다.
8. 수신자 측에 0번 패킷이 도착하고 수신자는 0번 액을 보내고 1번을 기다리는 상태가 된다.
- 근데 RDT 2에서는 로스에 대해서는 고려하지 않았다. 단지 첵섬을 추가해서 비트에러만 확인했을 뿐……
### RDT 3.0
* 기존 2버전 + Loss가 나는 채널에 대해서도 고려하자.
* Loss가 생기는 경우: 데이터를 보냈는 데 액 메시지가 오는건지 안오는건지….
* 계속 기다릴 수 없는 일. 타이머를 두어서 그 안에 못들어오면 Loss로 간주하고 재전송하자.
* 중복 데이터 전송은 이미 시퀀스 넘버로 처리가 가능하므로 문제 ㄴㄴ
![q]( https://raw.githubusercontent.com/HongYooCho/Network/master/Photo/rdt3.0.PNG)
* 2.2와는 다르게 잘못된 액이오거나 커럽트가 발생하면 재전송하지 않고 대기
* 재전송은 타이머가 끝난 후에 보내준다. (타이머도 이때 초기화)
* 제대로 된 액이 왔을 경우, 타이머를 멈추고 다음 시퀀스 데이터로 넘어감
* 일반적인 로스는 걍 타임아웃 끝나고 재전송하면 끝. 하지만 긴 여정을 하고 온 액 메시지가 늦게 온거라면??
![q]( https://raw.githubusercontent.com/HongYooCho/Network/master/Photo/rdtp3.0.PNG)
- 이것이 바로 premature timeout.
- 이럴 경우 위에 써있는 것처럼 잘못된 액이 오면 걍 무시하고 대기 한다. 또한 늦게 온 액은 결국 재전송된 패킷의 액이 빨리 온것으로 처리된 것이고 다음 패킷을 보낼 때 이전 재전송 패킷에 대한 액이 오는데 무시하면 Ok
### Utilization
* 근데 rdt가 신뢰성있게 보내주는 건 좋은데 하나씩 보내고 기다리고 또 보내고 기다리고….. 효율성은 그지다.
* 그러므로 데이터를 보낼 때 하나씩이 아닌 여러 개의 데이터를 동시에 보내주자 (비록 해당 패킷에 대한 액을 받지 못하더라도) -> Pipelining
* 파이프라이닝 프로토콜: 고백엔, Selective repeat
### Pipelining
* 데이터를 한번 보낼 때 여러 개를 동시에 보내므로 효율성은 증가하게된다.
* 하지만 데이터 양을 계속해서 증가시키게 되면 라우터에 있는 버퍼의 큐가 넘치게 되고 로스가 발생할 확률도 높아진다. 또한 데이터들이 무사히 도착했다고 해도 리시버 측에서 데이터 전부를 수용 할수 없어 로스가 발생할 수 있다.
- 즉 데이터의 양에 따라서 효율성과 로스는 비례관계. 둘을 잘 컨트롤 해야한다.
### GO –BACK –N
* 발신자는 N만큼의 데이터를 액을 받지 않아도 보낼 수 있다.
* 수신자는 오직 Cumulative ack만 보낸다. -> 어디까지 받았다 라고 알려주는 액
* 발신자는 가장 늦게 보낸 패킷에 대한 타이머만 고려해준다.
- 타이머가 끝나면 액을 받지 못한 모든 패킷들을 다시 재전송
* Sliding window 프로토콜을 사용
* 순서대로 액이와야한다. 리시버측에서는 이상한 순서로 온 데이터에 대해선 버려버리고 다시 재전송을 대기한다.
### Selective repeat
* 고백엔과는 달리 리시버측에서 어떤 패킷들을 정확히 받았는 지 알고 있다.
* 발신자는 액을 받지 못한 데이터만 보낼 수 있다.
- 액을 받지 못한 패킷에 대해 타이머가 있다.
* 고백엔이였으면 중간에 못받으면 거기서부터 쭉 다시 받아와야하는데 얘는 못받은 데이터만 다시 받으면 된다. 대신 SR은 이 정보를 저장해줄 버퍼가 필요하다.
- eg) 5개의 패킷을 보내고 3번에서 로스. 고백엔 -> 3 4 5 다시 전송, SR -> 3만 재전송
