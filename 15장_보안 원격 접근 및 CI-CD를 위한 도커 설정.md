이번 장에서는 도커 API를 안전하게 외부로 공개하는 방법을 익히고, 이를 활용해 로컬 컴퓨터 혹은 지속적 통합-지속적 배포 파이프라인에서 도커 엔진에 접근해 보는 방법을 배운다.

# 15.1 도커 API 엔드포인트 형태
이번 절을 끝까지 읽고 나면 도커 엔진 원격 접근이 어떤 식으로 동작하는지 이해할 수 있을 것이므로 비보안 HTTP 접근 대신 더 안전한 수단을 택하게 될 것이다.<br>

[실습] 원격 접근은 도커 엔진 설정에서 허용할 수 있다. 윈도 10이나 맥에서 도커 데스크톱을 사용 중이라면 고래 아이콘을 클릭해 메뉴에서 Settings를 선택한 다음,<br>
      Expose Daemon on tcp://localhost:2375 Without TLS 항목을 체크하라. 그림 15-1에 해당 설정 화면을 실었다. 설정을 저장하면 도커 엔진이 재시작 한다.<br>

<br>
도커 API로 HTTP 요청을 보내 원격 접근 허용 설정이 잘 적용됐는지 확인해 보자. 명령행 도구에 TCP 호스트 주소를 인자로 주어 사용해 봐도 알 수 있다.<br>

[실습] 도커 명령행 도구로도 host 인잣값을 지정해 원격 도커 엔진에 접근할 수 있다. 로컬 호스트에 원격 접근도 가능하다. 그러나 이 경우에는 TCP대신 로컬 채널을 사용한다.<br>

> 로컬 도커 엔진에 TCP 프로토콜을 통해 접근<br>
> docker --host tcp://localhost:2375 container ls<br>
> 이번에는 HTTP를 통해 REST API로 접근<br>
> curl http://localhost:2375/containers/json<br>

<br>

<img src="https://github.com/user-attachments/assets/c875c338-fd4b-4666-8614-2bb79779f3e5" style="width:50%">

<br>
그럼 이제 우리가 도커 서버를 원격으ㅏ로 접근하게 해 달라고 요청했을 때 운영팀이 느꼈을 공포를 한번 상상해 보자. 이것은 다시말해, 해당 서버의 도커를 누구든지 다룰 수 있게 해 달라는 것과 다름 없다.<br>
그것은 아무 보안 수단도 없이 흔적도 남기지 않을 방법으로 말이다. 도커 엔진에 대한 원격 접근이 얼마나 위험해질 수 있는지 간과해서는 안 된다.

[실습] 도커 엔진에 대한 비보안 원격 접근을 허용하는 것이 왜 위험한지 직접 체험해 보자. 먼저 도커 엔진을 실행 중인 컴퓨터의 디스크를 마운트한 컨테이너를 실행한다.<br>
      이 컨테이너를 통해 호스트의 파일 시스템을 마음대로 뒤질 수 있다.<br>
<br>

> 리눅스 컨테이너 환경의경우<br>
> docker --host tcp://localhost:2375 container run -it -v /:host-drive diamol/base<br>
> 컨테이너 안에서 호스트 컴퓨터의 파일 시스템을 뒤질 수 있다.<br>
> ls<br>
> ls host-drive<br>

<br>

![image](https://github.com/user-attachments/assets/dfe2441d-b320-4ce9-87bb-d309b77367c4)

<br>
절대 도커 엔진에 비보안 원격 접근을 허용해서는 안 된다.<br>

# 15.2 보안 원격 접근을 위한 도커 엔진 설정

도커에는 API가 요청을 받아들일 수 있는 채널이 두 가지 더 있다. 이 두 가지 채널 모두 보안 채널이다.<br>
첫 번째 채널은 전송 계층 보안으로, HTTPS 프로토콜의 디지털 인증서와 같은 방식의 암호화를 사용한다. 도커 API는 상호 TLS를 사용하므로 서버와 클라이언트 각각 인증서를 갖는다.<br>
서버의 인증서는 자신을 증명하고 전송되는 내용을 암호화하는 데 사용되며, 클라이언트 인증서는 자신을 증명하는 데 사용된다.<br>
두 번쨰 채널은 보안 셸(Secure Shell, SSH) 프로토콜이다. 이 프로토콜은 리눅스 서버에 원격 접속 하는 표준 프로토콜이지만, 윈도에서도 사용 가능하다. SSH로 원격 서버에 접근하려면 사용자명과 패스워드 혹은 피밀키가 필요하다.<br>
<br>

![image](https://github.com/user-attachments/assets/972ecd5b-1070-4195-a851-66941bb8d1c5)

<br>
우선 상호 TLS를 이용해 도커 엔진의 보안 원격 접근을 설정해 보자. 상호 TLS를 사용하려면 먼저 인증서와 키 파일 쌍을 두개 만들어야 한다.<br>
키 파일은 인증서의 패스워드 역할을 한다. 하나는 도커 API가 사용하고, 다른 하나는 클라이언트에서 사용된다.<br>

[실습] Play with Docker 웹 사이트에 로그인한 후 노드 하나를 생성하라. 그리고 같은 세션에서 인증서를 배포할 컨테이너를 실행한다. 그다음에는 도커 엔진이 이 인증서를 사용하도록 설정한 후 도커를 재시작하라.<br>

> 인증서를 둘 디렉터리를 생성하라<br>
> mkdir -p /diamol-certs<br>
> 인증서 및 설정값을 적용할 컨테이너를 실행한다.<br>
> docker container run -v /diamol-certs:/certs -v /etc/docker:/docker diamol/pwd-tls:server<br>
> 새로운 설정을 적용해 도커를 재시작한다.<br>
> pkill dockerd<br>
> dockerd &>/docker.log &<br>

<br>

![image](https://github.com/user-attachments/assets/7689cd39-37c2-4e32-857e-85b94b147478)

<br>

![image](https://github.com/user-attachments/assets/7a0eb19a-f233-4a7f-995f-28a11b98ee8a)

<br>
TLS를 통해 도커 엔진에 접근하려면 인증 기관과 한 싸으이 인증서(클라이언트/서버 인증서)가 필요하다.
<br>
이제 도커 엔진 원격 접근에 보안이 적용됐다. 지금부터 인증 기관 인증서, 클라이언트 인증서및 키 없이 curl로 REST API를 호출하거나 도커 명령행 도구로 이 원격 도커 엔진에 명령을 내릴수 없다.<br>
클라이언트 TLS없이 API사용을 시도하면 도커 엔진이 접근을 차단한다.<br>

[실습] 실행 중인 도커 엔진에 접근할 떄는 항상 2376번 포트를 사용해야 한다. 앞서 2376번 포트를 개방할 때 복사한 현재 세션의 도메인을 사용한다. 이제 원격 도커 엔진에 접근해 보자.<br>

> address 항목에서 현재 세션의 도메인을 복사한다 #대략 ip172-18-0-62-b09pj8nad2eg008a76e0-6379.direct.Labs.play-with-docker.com과 # 비슷한 값이다<br>
> 도메인을 환경 변수로 설정한다(윈도)<br>
> SpwdDomain="<나의_현재세션_pwd_도메인>"<br>
> 도메인을 환경 변수로 설정한다(리눅스)<br>
> pwdDomain="<나의_현재세션_pwd_도메인>"<br>
> 도커 API에 직접 접근을 시도한다<br>
> curl "http://$pwdDomain/containers/json"<br>
> 명령행 도구로도 직접 접근을 시도한다<br>
> docker --host tcp://sowdDomain" container ts<br>
> 클라이언트 인증서를 로컬 컴퓨터로 추출한다<br>
> mkdir -p /tmp/pwd-certs<br>
> cd ./ch15/exercises<br>
> tar -xvf pwd-client-certs -C /tmp/pwd-certs<br>
> 클라이언트를 사용해 도커 엔진에 접근을 시도한다<br>
> docker --host "tcp://$pwdDomain" --tlsverify --tlscacert /tmp/pwd-certs/ca.pem --tlscert /tmp/pwd-certs/client-cert.pem --tlskey /tmp/pwd-certs/client-key.pem container ls<br>
> 도커 명령행 도구로 명령을 내릴 수 있다<br>
> docker --host "tcp://$pwdDomain" --tlsverify --tlscacert /tmp/pwd-certs/ca.pem --tlscert /tmp/pwd-certs/client-cert.pem --tlskey /tmp/pwd-certs/client-key.pem container run -d -P diamol/apache<br>

<br>

[실습] Play with Docker 웹 사이트의 현재 세션으로 돌아가자. node1의 IP주소를 옮겨 적은 뒤, 다른 노드를 하나 더 생성한다. 다음 명령을 입력해 node2에서 SSH를 통해 node1에서 실행 중인 도커 엔진에 명려을 내려 보자

> node1의 IP 주소를 환경 변수로 정의한다<br>
> node1ip="<node1-ip-address-goes-here>"<br>
> 접속 테스트를 위해 SSH로 접속을 시도한다<br>
> ssh root@node1ip<br>
> exit<br>
> node2에서 해당 노드에서 실행 중인 컨테이너의 목록을 확인한다<br>
> docker container ls<br>
> node1에서 원격으로 접근한 도커 엔진에서 실행 중인 컨테이너의 목록을 확인한다<br>
> docker -H ssh://root@node1ip container ls<br>

Play With Docker를 사용하면 이 과정이 매우 간단해 진다.<br>

![image](https://github.com/user-attachments/assets/89c6bc9f-4bd2-4690-828f-523434c6b953)


# 15.3 도커 컨텍스트를 사용해 원격 엔진에서 작업하기
<br>
[실습] TLS 보안이 설정된 Play with Docker 내 도커 엔진의 도메인과 인증서로 컨텍스트를 생성한다. 도커 컨텍스트를 사용하면 원격으로 접근할 도커 엔진을 편리하게 전환 할 수 있다.
<br>

> Play with Docker 내 도커 엔진의 도메인과 인증서로 컨텍스트를 생성한다<br>
> docker context create pwd-tls --docker "host=tcp://$pwdDomain,ca=/tmp/pwd-certs/ca.pem,cert=/tmp/pwd-certs/client-cert.pem,key=/tmp/pwd-certs/client-key.pem"<br>
> SSH 보안을 적용한 경우<br>
> docker context create local-tls --docker "host=ssh://user@server"<br>
> 컨텍스트 목록을 확인한다<br>
> docker context ls<br>

<br>

![image](https://github.com/user-attachments/assets/fa26f5af-0737-4849-aa1d-c34f75da9877)

<br>
context는 로컬 엔진이나 원격 엔진 간에 대상을 전환하기 위해 필요한 모든 정보가 들어간다.
context는 로컬 네트워크의 컴퓨터나 인터넷상의 컴퓨터도 가리킬 수 있다.<br>
context를 전환하는 방법은 두가지다.<br>
한가지는 해당 터미널 세션에만 적용되는 임시적인 전환,
다른 한가지는 이후 다른 컨텍스트로 다시 전환할 때까지 다른 터미널 세션에도 모두 적용되는 영구적인 전환이다.<br>

[실습] context가 전환되면 그때부터 호스트명을 지정하지 않고 입력된 도커 명령은 현재 컨텍스트가 가리키는 도커 엔진으로 전달된다.<br> 
컨텍스트는 환경 변수를 사용해 임지로 전환할 수도 있고, context use명령을 사용해 영구적으로 전환할 수도 있다.<br>

> 환경 변수를 사용해 이름이 있는 컨텍스트로 전환하는 방법<br>
> 해당 터미널 세션에서만 적용되지만, 권장되는 컨텍스트 전환 방법이다.<br>
> 윈도 환경에서<br>
> $env:DOCKER_CONTEXT='pwd-tls'<br>
> 리눅스 환경에서<br>
> export DOCKER_CONTEXT='pwd-tls'<br>
> 현재 선택된 컨텍스트를 확인한다<br>
> docker context ls<br>
> 기본 컨텍스트로 복귀한다. 이 방법으로 컨텍스트를 전환하면 다른 터미널 세션까지 영향을 미치기 때문에 바람직하지 않다.<br>
> docker context use default<br>
> 현재 컨텍스트가 가리키는 도커 엔진의 컨테이너 목록을 확인한다<br>
> docker container ls<br>

<br>
예상과 다른 결과가 출력된다.

docker context use 명령으로 컨텍스트를 전환하면 시스템 전체에 영향이 미친다.
새 터미널 창을 열거나 도커 명령을 사용하는 모든 배치 프로세스가 이 컨텍스트를 사용한다. 이 컨텍스트 설정은 환경 변수 DOCKER_CONTEXT를 정의해 오버라이드 할 수 있다.<br>
이 환경 변수의 값은 docker context use 명령으로 설정한 컨텍스트에 우선하지만, 현재 터미널 세션에만 적용된다. 만약 주기적으로 컨텍스트를 전환해 가며 도커를 사용한다면,<br>
기본 컨텍스트는 로컬 도커 엔진을 가리키도록 그대로 두고 환경 변수만을 사용해 컨텍스트를 전환하는 것이 좋다.<br>
그렇지 않으면, 어제 퇴근하기 전에 운영 서버 컨텍스트로 전환한 것을 까맣게 잊고 다음 날 아침 시원하게 모든 컨테이너를 삭제하는 실수를 저지르게 된다.<br>
<br>

![image](https://github.com/user-attachments/assets/1b0e644a-084a-47fb-8c74-e8c0feb6d8db)

<br>
물론 운영 서버의 도커 엔진에 주기적으로 접근할 일이 없을 수도 있다. 하지만 컨테이너를 배우면 배울수록 도커의 장점 중 하나인 쉬운 자동화의 편리함을 누리게 되고,<br>
도커 엔진에 직접 접근하는 사용자는 최고 관리자와 지속적 통합/배포 파이프라인에 사용되는 시스템 계정 외에는 필요치 않다는 것을 알게 된다.<br>
<br>

# 15.4 지속적 통합 파이프라인에 지속적 배포 추가하기









# 15.5 도커 리소스 접근 모델









# 15.6 연습 문제
