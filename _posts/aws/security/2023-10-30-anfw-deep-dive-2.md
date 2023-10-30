---
title: '[ANFW] 파헤치기 2'

categories:
  - AWS Security
tags:
  - anfw, hands on

toc: true
toc_sticky: true
toc_label: 'anfw hands on'
toc_icon: 'bookmark'

date: 2023-10-30
last_modified_at: 2023-10-30
---

![https://awstip.com/network-security-with-amazon-network-firewall-80b84347fec3](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/d079d37e-8c8d-4dee-b6e4-de6b572fd4fc){: .align-center}

## [00] ANFW 사전 구성

[ANFW 파헤치기 1](https://gyu-beom.github.io/aws%20security/anfw-deep-dive-1/) 에서 이론을 공부했다라면 이번에는 직접 테스트를 해보는 시간이다.

![anfw-deep-dive-2 drawio drawio](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/b4392268-0dcb-4558-9830-0eb792b6e609){: .align-center}

- _VPC, IGW, NAT GW 등 ANFW 가 아닌 다른 서비스들 생성을 다루기엔 글이 장황해지기에 구성은 Architecture 로 대체한다._

- _라우팅 테이블이 FW End 2a, FW End 2c 로 되어 있는 부분은 ANFW 를 생성하고 Ready 상태인 것을 확인한 후에 선택해야 한다._

- _[Edge Association IGW 에 적용해보기.](https://gyu-beom.github.io/aws%20network/igw-edge-association/)_

## [01] ANFW Egress Firewall Policy 생성

### 1. Describe firewall policy

![스크린샷 2023-10-30 14 06 41](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/ac3267ae-b70c-486c-a831-2de8d2d39185){: .align-center}

우선, Egress 를 생성할 것이라 egress firewall policy 를 생성한다.

### 2. Add rule groups

![스크린샷 2023-10-30 14 07 43](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/9080a2c3-d600-4b9e-a9ff-b1f043f29fd0){: .align-center}

![스크린샷 2023-10-30 14 08 03](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/22087664-d4b4-446f-a2e6-aee045659f97){: .align-center}

위 2가지 사진에서처럼 기본값으로 두고 생성한다.

사진을 보여주지 않는 부분은 넘어간다.

### 3. Add tags

![스크린샷 2023-10-30 14 09 08](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/30189292-a8a5-4a2a-9fd6-5ea0063f95ce){: .align-center}

**태그는 생명이다.**

**Name 태그라도 생성하는 습관을 만들어 놓는 것이 좋다.**

### 4. Review

![스크린샷 2023-10-30 14 09 29](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/cfb7c5b9-9c05-4681-a176-08fb100229f1){: .align-center}

잘 생성된 것을 확인할 수 있다.

Stateless, Stateful rule group 도 없는 빈 내용을 생성한 것이라 바로 다음에 Stateless rule group 부터 생성하도록 하자.

- _빈 내용의 Firewall Policy 를 생성한 이유는 Firewall 생성 시, Firewall Policy 를 그 자리에서 생성하거나 기존에 생성된 Firewall Policy 를 가져오라고 한다._

- _개인의 차이이긴 하겠지만 보통 이런 상황에서의 나는 빈 내용이라도 만들어서 기존에 생성되어있는 것을 붙이려고 하기 떄문에 미리 빈 내용의 Firewall Policy 를 생성하였다._

## [02] ANFW Egress Stateless Rule Group 생성

Stateless Rule Group 은 상태 비저장으로 허용하는 트래픽의 정방향과 역방향까지 모두 기입해줘야 하는 것이 큰 특징이다.

- _단, ANY - ANY 는 예외._

| Priority | Protocol |    Source    | Destination  | Source port range | Destination port range | Action  | Custom action | Masks | Flags |
| :------: | :------: | :----------: | :----------: | :---------------: | :--------------------: | :-----: | :-----------: | :---: | :---: |
|    1     |   ICMP   |  0.0.0.0/0   |  0.0.0.0/0   |         -         |           -            |  Pass   |       -       |   -   |   -   |
|    2     |   TCP    | 10.10.0.0/16 |  0.0.0.0/0   |    49152:65535    |       80<br>443        | Forward |       -       |   -   |   -   |
|    3     |   TCP    |  0.0.0.0/0   | 10.10.0.0/16 |     80<br>443     |      49152:65535       | Forward |       -       |   -   |   -   |
|    4     |   TCP    |  0.0.0.0/0   |  0.0.0.0/0   |      0:65535      |        0:65535         |  Drop   |       -       |   -   |   -   |

위 표대로 생성할 것이다.

Priority 2 의 Source port range 값인 49152:65535 은 동적 포트를 의미한다.

외부로 나갈 때 내부에서 사용하는 포트는 동적 포트라고 생각하면 된다.

### 1. Describe rule group

![스크린샷 2023-10-30 14 49 33](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/7e11f66b-deac-4f63-8f9c-865c10bb4b5b){: .align-center}

Egress firewall policy 를 생성했던 것과 마찬가지로 비슷하게 이름을 생성한다.

### 2. Configure rules

![스크린샷 2023-10-30 14 52 42](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/04366849-7cef-41d6-9721-b78854d0df9b){: .align-center}

다음과 같이 Priority 를 신경 써서 생성해주면 된다.

- _혹시라도 Priority 가 잘못되었다고 해서 삭제하지 말자. 먼저 생성시킨 뒤에 Priority 만 변경할 수 있다._

### 3. Add tags

![스크린샷 2023-10-30 14 54 09](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/8b8706fa-90ff-4aa6-be06-5a9543f64465){: .align-center}

**태그는 생명이다.**

### 4. Review

![스크린샷 2023-10-30 14 55 33](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/ba1ce0ca-48b5-44c2-b604-83f8c08b267e){: .align-center}

잘 생성된 것을 확인할 수 있다.

## [03] ANFW Egress Stateful Rule Group (Domain) 생성

### 1. Choose rule group type

![스크린샷 2023-10-30 14 59 36](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/561f249b-f492-4bdb-b46d-ea85b1e03664){: .align-center}

**Domain list** 을 클릭해준다.

### 2. Describe rule group

![스크린샷 2023-10-30 15 00 39](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/32680f19-8b33-4130-95eb-0dcd01d2840a){: .align-center}

### 3. Configure rules

![스크린샷 2023-10-30 15 01 02](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/1edfc3ae-55fc-471b-bdb4-ef865cd775de){: .align-center}

사진처럼 위 2가지의 도메인은 오픈을 해주어야 한다.

이유는 엔드포인트 등 기타 여러 가지 AWS 리소스들의 엔드포인트 URL 형태가 `.amazonaws.com` 혹은 `.amazon.com` 으로 되어 있기 때문이다.

### 4. Add tags

![스크린샷 2023-10-30 15 04 09](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/db98ea9d-b2cb-40d0-ad0c-7c0758de50c8){: .align-center}

**태그는 생명이다.**

### 5. Review

![스크린샷 2023-10-30 15 04 19](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/d1164a22-5bc2-428e-b958-75d82df993b6){: .align-center}

잘 생성된 것을 확인할 수 있다.

## [04] ANFW Egress Stateful Rule Group (5tuple) 생성

| Rule Variables Name |  Type  |             Values             |
| :-----------------: | :----: | :----------------------------: |
|   DEV_PRI_SERVER    | IP set | 10.10.91.0/24<br>10.10.93.0/24 |
|        PORT         | Ports  |           80<br>443            |

| Protocol |     Source      | Destination | Source port | Destination port | Direction | Action | Keyword |
| :------: | :-------------: | :---------: | :---------: | :--------------: | :-------: | :----: | :-----: |
|   HTTP   | $DEV_PRI_SERVER |     ANY     |     ANY     |      $PORT       |  Forward  | Reject |  sid:1  |

### 1. Choose rule group type

![스크린샷 2023-10-30 15 14 12](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/935562ff-7274-407d-b4f3-fc29be8a9a74){: .align-center}

**Standard stateful rule** 을 클릭해준다.

### 2. Describe rule group

![스크린샷 2023-10-30 15 15 31](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/18f3c85a-138f-4d56-878b-11dd1c863344){: .align-center}

이름은 알맞게 설정해준다.

### 3. Configure rules

![스크린샷 2023-10-30 15 15 54](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/63e581fe-6ef4-448a-82b7-df7d170c11ac){: .align-center}

사진에서 볼 수 있듯이 Stateful Rule Group 중 Standard stateful rule 은 변수를 지정할 수 있다.

![스크린샷 2023-10-30 15 16 19](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/7c98e49a-9a85-4773-ac1f-a6a9835737b4){: .align-center}

생성한 변수를 사용하기 위해서는 변수명 앞에 `$` 을 붙여준다.

### 4. Add tags

![스크린샷 2023-10-30 15 16 33](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/37b29ab8-7937-40b9-9fad-4e06b9869896){: .align-center}

**태그는 생명이다.**

### 5. Review

![스크린샷 2023-10-30 15 16 46](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/42e57cf8-11e8-4721-83f7-bcfec2e431fa){: .align-center}

잘 생성된 것을 확인할 수 있다.

## [05] ANFW Egress Firewall Policy - Egress Rule Group 연결

이제 생성한 Rule Group 과 Firewall Policy 를 연결해주어야 한다.

![스크린샷 2023-10-30 15 22 16](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/a5808e9e-cc8f-470f-bb8f-89ea4913573d){: .align-center}

Stateless rule group 부터 연결해준다.

![스크린샷 2023-10-30 15 24 06](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/272d864d-00ca-4ed4-b292-5bc6efd66e3a){: .align-center}

**Add stateless rule group** 버튼을 클릭하면 잘 연결된다.

![스크린샷 2023-10-30 15 26 19](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/1a85d808-fb23-4e5b-bfff-5b6c85b9c2e4){: .align-center}

![스크린샷 2023-10-30 15 26 38](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/59214784-ac21-44dc-957d-c54b64442214){: .align-center}

Stateless rule group 과 마찬가지로 Stateful rule group 또한 마찬가지로 동일한 작업을 해준다.

- _주의 사항으로는 Domain 이 맨 위에 있어야 한다._

- _이유 1: Stateful 의 Strict mode 는 Stateless 와 마찬가지로 우선순위가 부여되어 동작한다._

- _이유 2: 우선순위에 만족하면 나머지 Stateful Rule Group 들은 무시하게 된다._

## [06] ANFW Egress Firewall 생성

대망의 ANFW Firewall 생성이다.

### 1. Describe firewall

![스크린샷 2023-10-30 15 27 26](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/aec36be5-26c8-4a92-84c4-80d69d97960d){: .align-center}

ANFW 의 이름을 알맞게 생성해준다.

### 2. Configure VPC and subnets

![스크린샷 2023-10-30 15 30 37](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/f5f7e868-cfe5-4f86-96e8-cfe8fa44fcda){: .align-center}

Architecture 에서 본 것처럼 FW 서브넷을 선택한다.

### 3. Configure advanced settings

![스크린샷 2023-10-30 15 31 08](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/ee17103c-8f8c-4f51-bfb8-f67713f2a161){: .align-center}

혹시나 모를 사고를 대비하여 삭제를 방지하는 옵션이 기본적으로 켜져있다.

### 4. Associate firewall policy

![스크린샷 2023-10-30 15 31 14](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/8ce2386b-73e2-4ea4-8ea4-2e3859e1fb3f){: .align-center}

앞 단계에서 열심히 생성했던 firewall policy 를 선택한다.

### 5. Add tags

![스크린샷 2023-10-30 15 34 36](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/034da82c-63d9-48d6-8b85-d54164be82c2){: .align-center}

**태그는 생명이다.**

### 6. Review

![스크린샷 2023-10-30 15 51 33](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/9d038db1-1673-4cf8-9d12-ed8fff8e18c0){: .align-center}

ANFW 메인 화면이다.

![스크린샷 2023-10-30 15 51 49](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/51c37a21-eb20-4c5c-8169-3a1f166c89dc){: .align-center}

VPC - Endpoints 에 가보면 GWLB Endpoint 가 VPC 설정에서 설정한 갯수만큼 되어 있을 것이다.

- _참고로 엔드포인트 태그 이름의 경우, 직접 기입해준 것이다. (최초 생성 시에는 기입되어 있지 않음.)_

## [07] Egress 트래픽 테스트

테스트는 도메인으로 진행할 것이다.

| 반환 예상 |         테스트 URL         |
| :-------: | :------------------------: |
|   성공    | https://aws.amazon.com/ko/ |
|   실패    |  https://www.google.com/   |

테스트를 진행하기에 앞서서 Architecture 상의 라우팅 테이블 대로 구성을 했는지 확인해야 한다.

### 1. EC2 SSM 으로 접속

![스크린샷 2023-10-30 16 45 37](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/dbbe947f-7036-4e98-949b-e5b41c6fa580){: .align-center}

![스크린샷 2023-10-30 17 03 33](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/13a3d49b-8813-413c-902b-bf5d9bde5b08){: .align-center}

EC2 에서 테스트하기 위해서 SSM 으로 접근한다.

### 2. curl 명령 테스트

![스크린샷 2023-10-30 17 32 39](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/2d7b7500-b186-4df3-a1ae-e83c869da0da){: .align-center}

```bash
curl -v https://aws.amazon.com/ko/
```

![스크린샷 2023-10-30 17 57 48](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/5ffd8616-19ba-402b-b4b5-10971f82209b){: .align-center}

```bash
curl -v https://www.google.com/
```

예상한 것과 정확하게 동작하는 것을 볼 수 있다.

---

원래는 Ingress 까지 생성해서 Ingress, Egress 모두 테스트를 해보려고 했으나 블로그 포스팅의 길이가 너무 길어져서 Egress 까지만 소개한다.

Ingress 의 경우 **AWS Network** 카테고리에서 다른 서비스들과 함께 보도록 하겠다.

---

피드백은 환영입니다.

잘못된 내용이 있다면 댓글로 알려주시면 감사하겠습니다! 😉
