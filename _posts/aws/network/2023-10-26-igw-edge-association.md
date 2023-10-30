---
title: '[Edge Assocation] IGW 에 적용해보기'

categories:
  - AWS Network
tags:
  - edge assocation, hands on

toc: true
toc_sticky: true
toc_label: 'igw edge assocation'
toc_icon: 'bookmark'

date: 2023-10-26
last_modified_at: 2023-10-30
---

## [00] Edge Association 이란?

![anfw-deep-dive-2 drawio drawio](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/b4392268-0dcb-4558-9830-0eb792b6e609){: .align-center}

- _참고 Architecture_

Edge Assocation 은 VPC 로 들어오는 ingress 트래픽을 appliance 로 라우팅하는데 사용하는 라우팅 테이블이다.

Internet Gateway 혹은 Virtual Private Gateway 와 연결한 라우팅 테이블에 대상을 appliance 의 eni 로 지정하면 된다.

## [01] IGW 전용 라우팅 테이블 생성

![스크린샷 2023-10-26 12 21 09](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/2a457f2f-78f7-4934-96b2-80a0d30b5565){: .align-center}

일반적인 라우팅 테이블을 생성하는 것과 동일하다.

이름은 편하게 설정하면 된다.

## [02] IGW 전용 라우팅 테이블에 IGW 연결

![스크린샷 2023-10-26 12 19 45](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/d5be6e18-1b40-43d9-ab2b-c424030f8f73){: .align-center}

Destination: ingress 트래픽이 발생하는 VPC Cidr 를 적어준다.

- _ex) public nlb subnet cidr 혹은 public ec2 subnet cidr 등_

Target: FW 의 ENI 를 선택한다.

- _사진에서는 ANFW 의 GWLB Endpoint 이기 떄문에 vpce-X 로 표기되는 중이다._

## [03] IGW 전용 라우팅 테이블 라우팅 설정

![스크린샷 2023-10-26 12 27 27](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/d1c721dc-c1d1-4c10-92c2-73f9e1dd7e2f){: .align-center}

Route Table 을 선택하고 Edge Assocation 탭을 클릭한다.

![스크린샷 2023-10-26 12 29 14](https://github.com/gyu-beom/gyu-beom.github.io/assets/122728665/4f91820e-15bd-48da-9367-f90cfef0f237){: .align-center}

IGW 가 나오게 되고 연결하고자 하는 IGW 를 선택하여 저장해주면 된다.

## [04] 정리

3가지 단계를 거치게 되면 ingress 트래픽 중 public nlb cidr 로 들어오는 트래픽은 ANFW Firewall Policy 를 거치고 최종 목적지로 가게 된다.

---

피드백은 환영입니다.

잘못된 내용이 있다면 댓글로 알려주시면 감사하겠습니다! 😉
