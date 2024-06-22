# docker
001 도커 컨테이너 사용법<br>
Container 관련 커맨드<br>
'docker container'<br>
start  컨테이너 실행   -i<br>
stop   컨테이너 정지<br>
create 컨테이너 생성   --name, -e, -p, -v<br>
run    이미지를 내려받고 컨테이너를 생성 및 실행    --name, -e, -p, -v, -d, -i, -t<br>
rm     컨테이너 삭제  -f,-v<br>
exec   컨테이너에서 프로그램 실행   -i, -t<br>
ls     컨테이너에서 목록 출력    -a<br>
cp     컨테이너와 호스트 간 파일 복사<br>
commit 컨테이너를 이미지로 변환<br>
# image
pull 이미지 내려받음<br>
rm 이미지 삭제<br>
ls 가지고 있는 이미지 목록을 출력<br>
build 이미지 생성<br>
# option
--name  컨테이너 이름<br>
-p 포트 번호 지정<br>
-v 불륨 설정<br>
-e 환경 변수 설정<br>
-d 백그라운드 실행<br>
-i 컨테이너에 터미널 연결<br>
-t 특수 키를 사용가능하게 설정<br>

# 도커 컨테이너와 통신하기
-p ${host_port}:${container_port}
![image](https://github.com/TeackjinLee/docker/assets/85720454/3c8bcec5-ffec-44c2-a96c-ac7a2dd92e2a)

# Dockerfile 이란?
dockerfile은 도커 이미지를 생성하기 위한 스크립트 파일<br>
여러 키워드를 사용하여 dockerfile을 작성하여 빌드를 보다 쉽게 수행할 수 있음<br>

<li>FROM   - FROM키워드를 사용하여 base가 되는 image를 지정 주로 OS이미지나 런타임 이미지를 지정함</li>
<li>RUN    - 이미지를 빌드할 때 사용하는 커맨드를 설정할 때 사용</li>
<li>ADD    - 이미지에 호스트의 파일이나 폴더를 추가하기 위해 사용</li>
<li>COPY   - 호스트 환경의 파일이나 폴더를 이미지 안으로 복사하기 위해 사용 'ADD'와 동일하게 동작하지만 가장 확실한 차이점은 URL을 지정하거나, 압축파일을 자동으로 풀지 않음</li>
<li>EXPOSE - 이미지가 통신에 사용할 포트를 지정할 때 사용</li>
<li>ENV    - 환경 변수를 지정할때 사용, 여기서 설정한 변수는 $name, ${name} 의 형태로 사용할 수 있음<br>
              추가로 ${name:-else} : name이 정의가 안되어 있다면 'else'가 사용됨</li>
<li>CMD    - 도커 컨테이너가 실행될 때 실행할 커맨드를 지정 'RUN'과 비슷하지만 CMD는 도커 이미지를 빌드할 때 실행되는 것이 아니라 컨테이너를 시작할 때 실행된다는 것이 다름</li>
<li>ENTRYPOINT - 도커 이미지가 실행될 때 사용되는 기본 커맨드를 지정(강제)</li>
<li>WORKDIR - RUN,CMD,ENTRYPOINT등을 사용한 커맨드를 실행하는 디렉토리를 지정 -w 옵션으로 오버라이딩 할 수 있음</li>
<li>VOLUME  - 퍼시스턴스 데이터를 저장할 경로를 지정할 때 사용<br>
              호스트의 디렉토리를 도커 컨테이너에 연결<br>
              주로 휘발성으로 사용되면 안되는 데이터를 저장할 때 사용</li>

docker build 커맨드<br>
docker build ${option} ${dockerfile direcotry}<br>
예) docker build -t test .<br>

# docker compose 파일
compose 파일은 도커 애플리케이션의 서비스, 네트워크, 볼륨 등의 설정을 yaml 형식으로 작성하는 파일<br>
- version<br>
- network<br>
- volume<br>
- config<br>
- secret<br>
주로  services만 사용, 여러 컨테이너를 정의하는데 사용됨<br>
![image](https://github.com/TeackjinLee/docker/assets/85720454/3d0bf068-57d7-4292-a2e1-d97645b94f4d)
<br>
컨테이너를 설정할 때 사용되는 키워드<br>
<img src="https://github.com/TeackjinLee/docker/assets/85720454/4ac14bd3-99f5-4074-ac3d-86c8da8116d6"/>
<br>
docker compopse 파일 실행<br>
작성된 docker-compose.yml파일을 실행하기 위해서는 아래와 같은 커맨드를 사용<br>
> docker-compose up<br>
옵션 설명<br>
-f 옵션<br>
docker-compose는 기본적으로 'docker-compose.yml' 또는 'docker-compose.yaml'의 이름을 사용<br>
만약 다른 이름으로 파일을 관리하고 사용한다면 아래와 같이 입력<br>
> docker-compose -f docker-compose-custom.yml up<br>

# 도커 이미지 생성하기
- 특정 이미지에 자주 사용하는 설정을 추가하여 편하게 사용하고 싶을 경우<br>
- 본인이 개발한 애플리케이션을 이미지로 생성하고 싶을 경우<br>
<br>
1. 첫번째 방법으로 준비된 컨테이너를 이미지로 변경하는 방법<br>
  - 아래 그림과 같이 설정이 반영되어 있는 컨테이너를 그대로 이미지로 생성<br>
<img src="https://github.com/TeackjinLee/docker/assets/85720454/1d29d062-cc7b-479b-ab48-670b36d62480"><br>
  - 이 작업을 수행하기 위해서는 컨테이너가 있는 상황에서 아래의 커맨드를 입력<br>
  - container_name : 이미지로 만들고자 하는 컨테이너의 이름<br>
  - image_name : 생성할 이미지의 이름<br>
  > docker commit {container_name} {image_name}<br>
2. Dockerfile로 이미지 생성하기<br>
  - Dockerfile에 추가하고자 하는 설정을 반영하고 그 파일로 이미지를 빌드<br>
> docker build ${option} ${dockerfile directory}<br>

3. 도커 이미지 파일로 저장
  - 이렇게 생성된 이미지는 파일로 저장 가능<br>
  - 많이 사용하지 않지만, 대체로 운영서버에서 이미지를 사용해야할 때 종종 사용되기도 함<br>
  3.1 save/load 커맨드<br>
    - save와 load 커맨드를 사용하면 아래와 같이 동작함<br>
    - save를 이용한 이미지 저장은 원본 이미지와 레이어를 동일하게 가져가는 형식으로 동작함<br>
    - save<br>
      도커 이미지를 tar 파일로 추출<br>
      > docker save -o test123.tar test123:latest<br>
    - load<br>
      추출된 tar파일을 이미지로 불러옴<br>
      docker load -i test123.tar<br>
  3.2 export/import 커맨드<br>
    - export와 import 커맨드를 사용하면 아래와 같이 동작함<br>
      export를 이용한 이미지 저장은 원본 이미지와 다르게 하나의 레이어로 통합되어 추출됨<br>
      이렇게 추출된 이미지는 다시 컨테이너로 가동하기 위해서는 별도의 작업이 필요함<br>
    - export<br>
      - 도커 컨테이너를 tar 파일로 추출<br>
        > docker export test123 > test123.tar<br>
    - import<br>
      - 추출된 tar파일을 이미지로 불러옴<br>
        > docker import test123.tar test123:version<br>
<br>
# 컨테이너 환경에서의 JVM 메모리 설정
1. 컨테이너 환경에서의 Heap Memory 설정<br>
  - JVM은 기본적으로 물리적인 메모리를 할당하게 설정<br>
  - 이후 컨테이너 환경에서의 운영환경이 발달함에 따라 컨테이너의 메모리를 기준으로 할당하는 기능이 추가<br>
  1.1 InitialRamPercentage<br>
    - Java 애플리케이션의 초기 힙 사이즈를 설정하기 위해 사용되는 매개변수 백분율을 사용하여 설정<br>
      Default: 1.5625%<br>
  1.2 MinRamPercentage<br>
    - 적은 메모리 사이즈에서 운영되는 애플리케이션에서의 최대 힙 사이즈를 설정하기 위해 사용<br>
      이런 경우 MaxRamPercentage의 설정은 무시됨<br>
      Default: 50%<br>
  1.3 MaxRamPercentage<br>
    - 충분한 양의 메모리에서 운영되는 애플리케이션의 최대 힙 사이즈를 설정하기 위해 사용
      Default: 25%<br>
<img src="https://github.com/TeackjinLee/docker/assets/85720454/f2908eff-f2d5-4221-9c5c-dea009c4e936"/><br>
<br>
2. 테스트를 위해 사용되는 주요 커맨드 조합<br>
  - Docker 커맨드를 활용하여 테스트를 수행 (테스트 환경 : Windows)<br>
  - Mac/Linux의 경우 find 커맨드를 grep으로 수정하여 사용 가능<br>
<img src="https://github.com/TeackjinLee/docker/assets/85720454/dfe95e79-229c-4cc3-ab7c-ab57c4410938"/><br>






