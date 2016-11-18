# DOCKER란?
개인적으로 docker의 정의를 딱 한줄로 요약하기는 어려운 일 같습니다. 그래서 [docker docs홈페이지](https://docs.docker.com/)의 정의를 빌려오자면..
*
docker는 안전하게 격리된 공간을 제공해주는 오픈소스입니다.
이 격리된 공간안에는 어떤 service를 수행하기 위한 모든 dependency와 libraries를 포함할 수 있습니다. 
*
라고 합니다. 

docker는 직접 사용해보기전까진 그 개념을 잡기가 좀 어려운데 위의 정의대로 격리되어 있으면서 목적을 위한 모든 수단이 갖춰져있는 공간을 제공해주는 오픈소스입니다.
격리된 공간을 제공한다는 점에서 vm을 떠올리시는 분들이 많을텐데 비슷합니다. 단지 훨씬 경량화되어 있고 용량도 작기 때문에 활용도가 더 큽니다. 

--image1 

위는 docker와 vm의 차이를 보여주는 그림입니다. 보시는 것처럼 hypervisor와 guest os가 제거되어 있고 대신 docker engine이 대신하고 있는 것을 볼 수 있습니다. 
vm은 hypervisor가 하드웨어 가상화를 먼저 수행하고 그 위에 guestos가 올라가는 형태가 되지만 docker는 커널위에 바로 격리된 공간을 구축합니다. guestos가 없기 때문에 완전한
vm으로 볼 순 없고 각종 실행에 필요한 binary와 lib만 구축이 된 형태입니다. 각각의 격리된 공간에서는 내가 원하는 어떤 서비스를 구동할 수도 있겠죠. 하지만 이 공간들은 
완전히 격리되어 있기 때문에 서로에 대해 영향을 받지 않습니다. 즉 하나의 서버에 여러개의 서비스를 띄우는 것이 부담스럽지 않을 수 있습니다. 

또 docker는 미리 격리 공간에 대한 템플릿을 준비해놓을 수 있습니다. 이 템플릿에는 격리된 공간에서의 서비스를 수행하기 위한 모든 사전준비를 넣어놓을 수 있습니다.
때문에 누가 실행하느냐, 어디서 실행하느냐에 상관없이 (docker만 돌아갈 수 있다면) 동일한 결과를 보여줄 수 있습니다. 

예를 들어 gitnsam 서비스 띄운다고 했을 때, 그냥 centos에서 gitnsam을 띄울려면 관련 dependency (ruby, bundle, rails, mysql 등등)을 미리 os에 설치한 후에 gitnsam을 
clone 받고 후 작업을 진행해야 합니다. 이것을 docker화 시킨다면 관련 dependency와 gitnsam이 들어있는 템플릿을 미리 만들어놓고 그것을 격리된 공간에서 수행하는 형식으로 
진행되겠죠. 후에 discourse를 같은 서버에서 띄운 다면 또다시 관련 dependency를 os에서 설치해야할텐데 이때부터 gitnsam dependency와 discourse dependency가 충돌할 수 있습니다.
비슷한 dependency를 사용하지만 버전이 다를 수 있으니까요. 하지만 docker화해서 구동한다면 어차피 격리된 공간이 보장되기 때문에 이런 문제점을 고민할 필요가 없습니다.

위에서 설명한 template와 격리된 공간을 docker용어로 치환하면 각각 image, container라고 부를 수 있습니다. 

# DOCKER INSTALL && SETTING
-- 문서 붙여넣기

# DOCKER ARCHITECTURE
--image2 

docker는 client-server architecture를 사용합니다. 일반적으로 하나의 linux에서 위의 과정을 거쳐 docker를 설치하게되면 client와 server가 한 os에 설치가 되는 형태이지만,
사실 docker는 docker client와 docker daemon이 따로 떨어져있을 수 있습니다. 

# DOCKER IMAGE 

그럼 image는 어디서 구할 수 있느냐.. 직접 만들거나 repository에서 가져올 수 있습니다. 여기서는 repository에 대해 설명할텐데요. 가장 유명한 곳은 [docker hub](hub.docker.com)가 있습니다. 사내에서는 redii가 있구요. 
누군가 어떤 목적으로 미리 만들어놓은 이미지를 사용할 수 있습니다. 예를들어 nginx를 띄운다고 해도 nginx에서 제공하는 가장 기본 nginx를 사용할 수 있고 누군가 reverse proxy 용으로 특화시켜놓은
nginx를 사용할 수도 있습니다. 개발할 때 오픈소스 가져다 쓰듯 검색을 좀 해보면 편하게 개발할 수도 있구요.

직접 만드는 방법은 두가지가 있습니다만 여기서는 Dockerfile에 대해서 먼저 설명하도록 하겠습니다. Dockerfile은 image를 만들 때 이러이러한 것들을 포함해 라고 미리 적어놓은 스크립트 같은 것입니다.
예를 들어 `nodejs`가 설치된 `ubuntu image`를 만들고 싶다면 다음과 같이 하면 되죠.

```
FROM ubuntu:latest

RUN apt-get install -qqy nodejs
```

from 은 여기서 부터 시작하라라는 뜻입니다. docker 진형에서는 이미 base가 될 수 있는 여러가지 이미지를 제공합니다. 위의 예제에서는 ubuntu의 최신버전에서부터 시작했습니다. 그 후 nodejs의 dependency를 
설치하는 스크립트를 실행하고 작업이 끝납니다. 만약 위의 image를 ubuntu-nodejs라는 이름으로 만든 상태에서 nodejs도 설치되어 있고 ruby도 설치된 ubuntu image를 만들고 싶다면

```
FROM ubuntu-nodejs:latest

RUN apt-get install -qqy ruby
```

처럼 이미 만든 이미지를 계속 활용할 수 있습니다. 

# DOCKER CONTAINER

위에서 만든 ubuntu-nodejs라는 이미지로 부터 격리된 공간, 즉 container를 만들고 싶다면 image를 run 시키면 됩니다.
이쯤에서 docker에서 가장 중요하다고 생각되는 명령어인 run에 대해 알아보겠습니다. 
```
docker run {options} {image_name or id}:{image version} {수행할 작업} 
```

docker run 명령어는 위와 같이 이루어져 있는데 주목하면 좋을 부분이 수행할 작업 부분입니다. docker container는 vm과 다르게 그냥 os하나 올리고 후 작업은 알아서 하는 형태라기보다는 어떤 목적을 가지고
격리된 공간을 가지는 형태입니다. 예를들어 gitnsam service를 수행하는 container나 hello-world라는 문구를 console에 찍는 container 같은 형태입니다. 그리고 그 격리된 공간은 이 목적을 다 수행하고 나면 
끝나게 되어 있습니다. 만약 그 작업이 계속되야한다면 (gitnsam service처럼) docker container로 계속 살아있는 형태가 되겠죠. 

예를 들어 위의 `ubuntu-nodejs`로 부터 예제를 들어보면
```
docker run -it ubuntu-nodejs /bin/bash
```

`ubuntu-nodejs image`에서부터 시작한 container를 bash명렁어를 수행시키는 명령어입니다. 이때 `/bin/bash` 명령어는 계속 interactive하도록 (-it option이 있어야합니다.) 유지되기 때문에 이 container는 사용자가 
종료하기 전엔 종료되지 않습니다. 반면

```
docker run ubnutu-nodejs echo 'aaa'
```
위의 명령어는 aaa라는 글을 콘솔에 찍고 container가 종료되게 됩니다. 즉 run될 때 수행하는 명령을 어떤식으로 정의하느냐에 따라 saas형태의 container가 될 수도 있고 개발환경을 세팅해주는 package가 될 수도 있으며
단순히 콘솔만 찍고 끝나는 container가 될 수도 있습니다. 

근데 이 명령어가 위의 예제에서야 매우 간단했지만 실제로 만들어보면 간단하게 딱 한줄로 떨어지는 경구는 드뭅니다. 그래서 image를 만들 당시에 이 image에서 시작한 container가 수행할 명령을 지정해놓을 수 있습니다.
바로 entrypoint입니다. 이 entrypoint에서 미리 image에 넣어놓은 script를 명령어로 지정할 수도 있죠. 따라서 꽤 복잡한 작업도 entrypoint로 지정가능합니다. 

# DOCKER COMMIT FOR IMAGE
이쯤에서 위에서 예제를 든
```
docker run -it ubuntu /bin/bash
```
명령어를 다시 상기시켜보겠습니다. 이 명령어를 실행시키면 (명령어를 ubuntu로 변경했으니 실제 실행되는 명령어입니다. 한번 해보세요) 마치 vm으로 들어온 것과 비슷한 동작을 합니다. 여기서 이런 생각을 해볼 수 있겠죠.
"Dockerfile로 image를 만들려면 script작업도 해야되고 그거 하나하나 하다가 오류나면 수정작업도 해야되고 귀찮네.. 그냥 container를 bash로 실행시킨다음에 내가 하고 싶은거 다 설치하고 서비스 띄우면 안되나?" 됩니다. 
심지어 그렇게 한 작업들을 다시 image화 시킬수도 있습니다. 재사용도 가능한거죠. 이 때 사용하는 기능이 docker commit입니다. 정말 git commit과 거의 유사하도록 돌아가고 commit 메세지나 log도 다 남습니다. 
따라서 image를 만들 수 있는 좋은 방법중 하나죠. 그럼 이런 생각이 들 수 있죠. 그럼 Dockerfile이 왜 필요하지.. image를 올릴 수 있는 repo도 있고 이력관리도 되고 이미 만들어진 image를 pull 받을 수도 있는데.. 라고 해도
실제로 docker image는 꽤 많은 작업들이 합쳐진 상태이기 때문에 이력만으로 이 image내부에 뭐뭐가 설치가 되어 있는지 확인하기가 어렵습니다. 마치 git 이력만 가지고 전체 project파일에 대해 파악할 수 없는 것과도 같죠. 
때문에 협업을 위해서 docker image를 만들 때는 꼭 Dockerfile을 생성해 놓길 추천드립니다. container로 띄울 때 에러가 나면 image에 정확히 어떤 내용이 있었는지를 모른채 디버깅하는 것이 매우 어렵습니다. 그 이미지를 내가 만들었다면
대충 내용을 알테니 어느정도 괜찮지만 그 이미지가 내가 만든것이 아닐 때는 Dockerfile이 없을 경우 매우 난감하죠.. 때문에 저는 개인적으로 commit으로 이미지를 만드는 것보다 /bin/bash에서 작업하는 내용을 잘 기록해뒀다가
Dockerfile로 image를 만드는 것을 추천하고 싶습니다. 


