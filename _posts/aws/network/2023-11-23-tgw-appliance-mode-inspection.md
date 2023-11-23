---
title: '[TGW Appliance Mode] Inspection VPC 구성해보기'

categories:
  - AWS Network
tags:
  - tgw appliance mode, inspection

toc: true
toc_sticky: true
toc_label: 'tgw appliance mode - inspection'
toc_icon: 'bookmark'

date: 2023-11-23
last_modified_at: 2023-11-23
---

![IMG_8012](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/6f3e562a-0d29-4f42-a032-293dfd44e2d1){: .align-center}

## [00] 사전 지식

1. 방화벽 특징

- 방화벽은 들어온 패킷을 검사하고 나가는 패킷을 검사한다는 아주 중요한 특징이 있다.

2. AZ 친화성

- AWS 는 같은 가용영역끼리 통신을 먼저 주고 받으려는 경향이 아주 강하게 구성되어 있다.

3. TGW Appliance Mode

- AWS 상에서 TGW 를 통해 방화벽 장비를 거쳐 서로 다른 AZ 와의 통신이 필요로 할 때 AZ 친화성을 무너뜨려주는 TGW 의 기능 중 하나이다.

말이 조금 어렵지만 밑에 그림과 실제 테스트를 보면 이해가 빠를 것이다.

## [01] 구성도

![tgw-appliance-mode-inspection drawio](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/ec8b8945-987b-4be1-bba7-c119bf32d564){: .align-center}

구성할 최종 아키텍처의 모습은 위와 같다.

- _NAT Gateway 는 일부러 생성하지 않았고, 서버는 미리 AMI 를 생성하여 간단하게 Golang 으로 웹 서버를 만들어서 테스트를 진행하였다._

## [02] TGW Appliance Mode Off

![스크린샷 2023-11-22 16 18 51](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/f45593ef-77a5-4b87-8bd9-16be77913d28){: .align-center}

우선, TGW Appliance Mode 가 꺼져있을 경우이다.

![tgw-appliance-mode-inspection-mode-off drawio](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/a4f38038-c045-45e5-89f3-7bf4047700ea){: .align-center}

방화벽은 기본적으로 들어온 패킷과 나가는 패킷을 검사하는데 그림을 자세히 보면 파란색 선과 빨간색 선이 서로 다른 `Firewall Endpoint` 를 향한다는 것을 알 수 있다.

이럴 경우 가용영역 c 에 위치한 방화벽 입장에서는 **"나한테 왜 return 패킷을 보내지? 내가 언제 너 (빨간색 선의 EC2) 한테 패킷을 줬었나??"** 라며 혼란에 빠지게 된다.

![스크린샷 2023-11-22 16 18 27](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/cd581bc8-af26-4ef8-91af-29935af0808c){: .align-center}

그렇게 되면 당연히 통신은 되지 않고 위 사진과 같이 Time Out 이 발생하게 된다.

## [03] TGW Appliance Mode On

그렇다면 어떻게 하면 TGW 와 방화벽을 거쳐 서로 다른 AZ 끼리의 통신을 가능하게 할 수 있을까?

![스크린샷 2023-11-22 16 02 01](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/9da72c4a-733e-4866-acd2-7d8b9f2c0bf7){: .align-center}

위와 같이 TGW Appliance Mode 를 켜준다.

![스크린샷 2023-11-22 16 01 45](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/90b48cc2-8b7e-4d11-aa4a-40b82ce0542b){: .align-center}

그렇게 되면 위 사진처럼 통신은 원활하게 될 것이다.

![tgw-appliance-mode-inspection-mode-on drawio](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/beab6d9a-2ace-48b3-b1a0-b09ffa5fef0a){: .align-center}

아키텍처의 흐름도는 위 사진과 같다.

## [04] Bonus: 만약 방화벽이 없는 상태에서는 어떻게 동작하는가?

![스크린샷 2023-11-22 17 05 34](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/3dae0833-f142-4030-90c0-49f986ad1b18){: .align-center}

![스크린샷 2023-11-22 17 04 42](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/ecb1c51e-20d3-4128-9b78-a0cfa7f5ba76){: .align-center}

정답은 위 사진처럼 통신이 아주 잘 된다.

방화벽이 없을 떄는 트래픽이 단순히 TGW 를 타고 흐르는 것이기 때문에 정상적으로 작동한다.

이를 `비대칭 통신` 이라고 부르며 AWS 에서는 라우팅 테이블의 기본적으로 들어가져 있는 `local` 이라는 경로 때문에 가능하다.

AWS 는 라우팅 테이블을 생성하면 항상 기본적으로 자기 자신의 vpc cidr 를 local 이라는 이름으로 사전에 등록을 시켜서 생성해준다.

이로 인해 서버의 보안그룹만 알맞게 설정해준다면 동일 vpc 의 az-a <-> az-c 의 통신이 가능하다.

---

피드백은 환영입니다.

잘못된 내용이 있다면 댓글로 알려주시면 감사하겠습니다! 😉
