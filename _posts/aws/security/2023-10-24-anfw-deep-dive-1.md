---
title: "[ANFW] 파헤치기 1"

categories:
    - AWS Security
tags:
    - anfw, theory

toc: true
toc_sticky: true
toc_label: "anfw deep dive"
toc_icon: "bookmark"

date: "2023-10-24"
last_modified_at: 2023-10-25
---

![https://awstip.com/network-security-with-amazon-network-firewall-80b84347fec3](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/d079d37e-8c8d-4dee-b6e4-de6b572fd4fc){: .align-center}

## [01] ANFW 소개

경험 + AWS 공식 문서를 토대로 내가 기억하기 쉽게 AWS Network Firewall (ANFW) 를 정리하려고 한다.

ANFW는 이름에서 유추할 수 있듯이 AWS Cloud Native 방화벽 서비스이다.

![anfw-theory-1-drawio](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/6a14e12b-fdcf-47a8-902c-05df766e8a72){: .align-center}

- *그림에서는 편의상 모든 서브넷을 퍼블릭 서브넷으로 구현하지만 실습에서는 EC2 서버의 경우 프라이빗 서브넷으로 구현할 것이다.*

- *나중에 실습에서 확인할 수 있겠지만 그림에서와 같이 ANFW 를 생성하면 자동으로 선택한 서브넷에 GWLB 엔드포인트가 생성된다.*

- *GWLB 엔드포인트를 생성하면서 대상이 되는 트래픽의 라우팅은 전부 GWLB 엔드포인트 id 로 지정해주어야 한다.*

- *GWLB 뒤에서 ANFW 가 동작하는 로직이기 때문에 ANFW 실제 동작 서버는 구경할 수 없다.*

ANFW를 사용하면 VPC 경계의 트래픽을 필터링할 수 있다.

- *여기서 얘기하는 트래픽은 **IGW, NAT GW, VPN, DX 로부터 들어오고 나가는 트래픽**을 의미한다.*

{: .notice--warning}
ANFW는 **ANFW 의 엔드포인트 (GWLB 엔드포인트) 가 존재하는 서브넷을 제외**한 VPC 의 모든 서브넷을 보호할 수 있다.

## [02] ANFW 의 3가지 리소스

### 1. Firewall

VPC 의 서브넷에 대한 트래픽 필터링 논리를 제공하는 AWS 리소스이다.

Firewall 과 Firewall Policy 는 1:1 구조이다.

- *즉, 1개의 Firewall 에는 오직 1개의 Firewall Policy 만 부착할 수 있다.*

### 2. Firewall Policy

VPC에서 들어오고 나가는 트래픽을 필터링할 때 사용할 방화벽의 규칙 및 기타 설정을 정의하는 AWS 리소스이다.

재사용이 가능한 1개 이상의 Stateful, Stateless Rule Group 을 정의한다.

1개의 Firewall Policy 를 여러 개의 Firewall 에 부착할 수 있다.

- *즉, 동일한 Firewall Policy 는 다른 Firewall 에 부착하여 사용할 수 있음을 의미한다.*

### 3. Rule Group

VPC 트래픽과 일치시킬 규칙 세트와 네트워크 방화벽이 일치 항목을 찾을 때 수행할 조치를 정의하는 AWS 리소스이다.

Firewall Policy 와 마찬가지로 재사용이 가능하며 트래픽을 검사하고 검사 기준과 일치하는 패킷 및 트래픽 흐름을 처리하는 기준들의 모음이다.

Stateful, Stateless 총 2가지의 종류가 있다.

{: .notice--warning}
AWS 공식 문서에는 오직 Rule Group 만 ARN 이 있다고 얘기하는 것 같지만 직접 확인해보면 **위 3가지 모두 AWS 리소스이기 때문에 ARN 이 존재**한다.

## [03] Stateless & Stateful Rule Group

![https://docs.aws.amazon.com/network-firewall/latest/developerguide/firewall-rules-engines.html](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/0f569876-8e0f-47a9-acc7-f8c39811e066){: .align-center}

### 1. Stateless Rule Group

상태를 저장하지 않는 규칙 그룹이고 AWS NACL 과 비슷한 개념으로 동작한다.

- *여기서 얘기하는 상태는 **트래픽 방향, 기존에 연결된 패킷**을 의미한다.*

상태를 저장하지 않는다는 말은 기존에 연결되었던 패킷이더라도 들어올 때와 나갈 때를 각각 검사하겠다는 것을 의미한다.

그래서 **5-tuple (source ip, destination ip, source port, destination port, protocol)** 로 작성한 규칙을 생성했다면 이와 정확하게 반대되는 규칙을 생성해야 정상적으로 동작한다는 의미이다.

Stateless Rule Group 은 우선순위를 부여할 수 있으며 우선순위의 숫자가 낮을수록 제일 먼저 검사를 시작하는 규칙이 된다.

### 2. Stateful Rule Group

상태를 저장하고 있으므로 규칙을 생성했다면 반대되는 규칙을 생성하지 않아도 되며 AWS Security Group 과 비슷한 개념으로 동작한다.

- *AWS Security Group 과 비슷한 개념이지만 디테일은 다르다.*

- *Stateful Rule Group 의 default traffic: **pass traffic***

- *AWS Security Group 의 default traffic: **deny traffic***

2가지의 Action 방식이 존재한다.

1. Strict order

- 우선순위를 지정하며 일치하는 규칙을 적용하고 나면 나머지 규칙은 무시한다.

- **AWS 자체에서 권장하는 옵션이다.**

2. Action order

- 우선순위를 지정하지 않고 오로지 Action 순으로 동작한다.

- 오로지 Action 순으로 동작한다는 것은 규칙이 하나의 Action 에 만족한다고 하더라도 나머지 Action 을 끝까지 본다는 뜻이다.

- 동작하는 Action 순은 **Pass -> Drop -> Alert** 으로 처리된다.

## [04] ANFW Traffic Flow

![anfw-theory-2 drawio](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/a5e03adb-f6d5-4840-aca2-836bb77ea9be){: .align-center}

그렇다면 ANFW 는 어떻게 동작하는 걸까?

쉽게 요약하면 Stateless Rule Group -> Stateful Rule Group 순으로 패킷을 검사한다.

각 Rule Group 에 속하는 규칙들을 검사하기 위해 필요한 것이 Engine 이다.

### 1. Stateless Rules Engine

ANFW 는 일치하는 항목을 찾거나 모든 Stateless Rule Group 을 소진할 때까지 방화벽 정책의 Stateless Rule 에 대해 각 패킷들을 검사한다.

우선순위의 숫자가 제일 낮은 규칙부터 검사한다.

### 2. Default Stateless Rule Actions

패킷이 Stateless Rule 과 일치하지 않는 경우 ANFW 는 패킷 유형에 따라 전체 패킷 또는 UDP 패킷 조각에 대해 ANFW 정책의 Default Stateless Rule 작업을 수행한다.

### 3. Stateful Rules Engine

ANFW 는 검사를 위해 Stateful Rules Engine 에 패킷을 전달할 때 패킷의 트래픽 흐름 컨택스트에서 Stateful Rule Group 에 대해 패킷을 검사한다.

---

피드백은 환영입니다.

잘못된 내용이 있다면 댓글로 알려주시면 감사하겠습니다! 😉
