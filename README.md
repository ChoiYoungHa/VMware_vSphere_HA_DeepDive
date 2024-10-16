<div align="center">
<h1>🥇우리FISA 기술 세미나 우승팀🥇 - VMware vSphere HA Deep Dive</h1>
</div>

## 👨‍👨‍👧‍👦 개발 팀원
| <img src="https://avatars.githubusercontent.com/u/64997345?v=4" width="150" height="150"/> | <img src="https://avatars.githubusercontent.com/u/86272865?v=4" width="150" height="150"/> | <img src="https://avatars.githubusercontent.com/u/90691610?v=4" width="150" height="150"/> | <img src="https://avatars.githubusercontent.com/u/127727927?v=4" width="150" height="150"/> |
|:-------------------------------------------------------------------------------------------:|:------------------------------------------------------------------------------------------:|:-------------------------------------------------------------------------------------------:|:------------------------------------------------------------------------------------------:|
|               **최영하** <br/>[@ChoiYoungHa](https://github.com/ChoiYoungha)               |                  **최수연**<br/>[@lotuxsoo](https://github.com/lotuxsoo)                   |             **이석철**<br/>[@SeokCheol-Lee](https://github.com/SeokCheol-Lee)              |                **부준혁** <br/>[@BooJunhyuk](https://github.com/BooJunhyuk)                 |     |
<br>

## 📌 프로젝트 개요

Enterprise 환경에서 가장 중요하게 생각되는 High Availability 즉 고가용성을 보장하기 위해 필요한 개념 학습을 목표로 합니다.<br>
On-premise 환경에서 가장 많이 사용되는 솔루션인 VMWare vSphere의 핵심원리와 Hands-On-Lab 실습 내용을 정리하여 우리FIS 아카데미 기술세미나에서 발표하였습니다.

1. VMware [공식문서 vSphere Availability](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.avail.doc/GUID-33A65FF7-DA22-4DC5-8B18-5A7F97CCA536.html) 정독
2. VMware HOL - [Virtualization 101 (HOL-2535-01-VCF-L)](https://labs.hol.vmware.com/HOL/catalog/lab/13928) 실습
3. 기타문서 - VMware vSphere 67\_ Clustering Deep Dive, VCP-DCV for vSphere 8.x Official Cert Guide
   <br>

## 🌱 학습 목적

- VMware vSphere Cluster 작동 원리 이해
- VMware vSphere HA 기능 및 세부 옵션 이해
- 자료 수집 및 검증 후 지식 체계화
- 발표력 향상
  <br>

## 🛠 목차

- 주제 선정 이유 및 개념 설명
  - 고가용성을 보장하지 못한 사례
  - vSphere HA란?
- HA 작동 원리 및 장애 대응 방식
  - Primary Host 선정 및 Secondary Host 역할
  - 네트워크 격리, 파티셔닝, 응답옵션
  - NIC 카드 이중화
  - 데이터스토어 하트비트
  - 스플릿 브레인 현상 대응
  - PDL, APD 현상 대응
- HA 설정 및 최적화 방법
  - Priority
  - Admission Control
  - 데이터스토어 하트비트 정책
  - Advanced Setting
- 결론 및 고찰
  - 워크로드에 따른 Admission Control 정책 설정
  - vSphere Cluster 기능 적극적으로 활용하기
  - 3Tier 구조에서 HCI로 전환하기
    <br>

## 💡 주제 선정 이유 및 개념 설명

### 고가용성을 보장하지 못한 사례

![2024-09-22 17 40 43](https://github.com/user-attachments/assets/f30c8fa0-cae1-436a-9693-183e30e7dd27)

- DR 센터를 따로 두지 않아 신속한 복구가 어려웠다.

### vSphere HA

![2024-09-22 18 08 59](https://github.com/user-attachments/assets/2993f17d-b7d5-4921-a76e-e45e33f7d9ed)

- vSphere HA는 하나의 호스트가 죽으면 해당 호스트의 VM을 다른 Host에 failover 하여 고가용성을 보장

## 🎯 HA 작동 원리 및 장애 대응 방식

### Primary Host 선정 및 Secondary Host 역할

![2024-09-22 18 11 25](https://github.com/user-attachments/assets/28c135a3-17db-4b6f-b04e-fdf1ff70990b)

- vSphere를 통해 host를 cluster로 묶게 되면 FDA 에이전트가 설치된다.
  - FDA 에이전트는 Primary host를 선출할 때 데이터스토어를 가장 많이 가진 host를 후보군으로 둔다.
- primary host는 secondary host를 계속 모니터링하며 vCenter와 통신한다.
  - host가 죽게 되면 primary host는 이를 감지하여 다른 host에 VM을 failover 시킨다.
  - 만약에 primary host가 죽으면 데이터스토어 개수를 기반으로 primary host를 다시 선출한다.

### 네트워크 격리, 파티셔닝, 응답옵션

![2024-09-22 18 20 02](https://github.com/user-attachments/assets/13aeae30-7e57-4cc4-9470-186897692065)

- 각 호스트들은 Management Network를 통해 Network Heartbeat를 주고받는다. - 만약에 외부와 클러스터 내부 모두 네트워크 통신이 끊긴다면 네트워크 격리 상태가 된다. - 대표적으로 NIC 카드 장애, HOST down이 원인 - 특정 호스트끼리는 통신이 가능하지만 나머지 내부 클러스터 간에 통신이 안된다면 네트워크 파티셔닝 상태다. - 대표적으로 잘못된 네트워크 설정, 잘못된 방화벽 설정이 원인
  ![2024-09-22 18 25 28](https://github.com/user-attachments/assets/19f2a86c-4781-4f97-b7da-28fe1401835c)
- 네트워크 격리 또는 파티셔닝 상태가 됐을 때 vSphere에서 응답옵션을 선택할 수 있다.
  - Power off 옵션은 호스트를 강제종료하여 다른 host로 VM을 failover 한다.
  - Shutdown 옵션은 호스트를 정상 종료 시키고 다른 host로 VM을 failover 한다.

### NIC 카드 이중화

![2024-09-22 18 29 06](https://github.com/user-attachments/assets/1fa36c7d-deba-49e3-8acd-7def049b7210)

- 네트워크 격리, 파티셔닝 상황을 방지하고자 Management network를 NIC 이중화를 통해 고가용성을 강화한다.

### 데이터스토어 하트비트

![2024-09-22 18 30 07](https://github.com/user-attachments/assets/698bdcf5-cd76-4662-a4ff-040678206a50)

- 네트워크 하트비트를 주고받을 수 없는 상태일 때 데이터스토어 하트비트를 통해 해당 호스트의 네트워크가 격리 또는 파티셔닝 상태가 되었는지 확인할 수 있다.
  - 데이터스토어 하트비트는 스토리지 내부에 존재하는 LUN이라는 논리적인 공간 내에 생성된 데이터스토어에 IO를 보내 Host의 동작 여부를 확인한다.
  - 다만 LUN은 NFS에서는 지원하지 않으며 VMFS만 지원한다.

### 스플릿 브레인 현상 대응

![2024-09-22 18 34 29](https://github.com/user-attachments/assets/02ad7b75-1620-489c-8b53-a3294787b819)

- 스플릿 브레인이란 네트워크 격리, 파티셔닝 응답 옵션이 설정되어 있지 않았을 경우 vSphere는 해당 호스트를 종료하지 않고 다른 Host에 VM을 failover 한다
  - 때문에 네트워크가 분리, 파티셔닝된 host에도 VM이 살아있고 failover된 host에도 VM이 살아 있는 상황이 발생한다.
  - 같은 VM이 다른 host에 똑같이 올라간 이 상태를 두뇌가 분리된 것 같다 해서 스플릿 브레인 현상이라고 부른다.
- 스플릿 브레인 현상을 해결하기 위해서 VMCP를 사용할 수 있다.
  - 같은 데이터스토어에 Lock을 가지지 않은 Host가 데이터스토어에 접근한다면 해당 host를 종료시킴으로써 스플릿 브레인 현상을 해결할 수 있다.

### PDL, APD 현상대응

![2024-09-23 10 05 27](https://github.com/user-attachments/assets/673d98f9-eeae-4220-b364-c49969149ae0)

- PDL : 스토리지 하드웨어가 문제가 생긴 상황
  - 복구될 가능성이 없기에 바로 다른 호스트로 failover 하는 옵션
- APD : 스토리지와 통신할 수 있는 스토리지 네트워크가 모두 다운된 상황
  - 복구될 가능성이 있기 때문에 보수적인 접근(일정 시간 대기하다가 failover)
  - 상황에 따라 즉각적인 failover 옵션도 사용 가능

## ✨ HA 설정 및 최적화 방법

### Priority

![2024-09-23 15 57 19](https://github.com/user-attachments/assets/99c1ef9a-399c-437f-9587-0097594b3878)

- 클러스터 레벨에서 VM이 올라갈 때의 default 우선순위를 정한다.

### Admission Control

![2024-09-23 18 39 22](https://github.com/user-attachments/assets/34c5679c-acd1-4b0b-8459-768bdea8111e)

- Percentage AdmissionControl
  - 클러스터의 자원을 일정 비율 예약해두어 호스트가 죽었을 때 예약해둔 자원으로 VM을 정상적으로 failover 할 수 있도록 보장하는 옵션
  - 일정 비율을 예약해두어 사용하기 때문에 대규모 클러스터에 적합한 방식이다.
- Slot AdmissionControl
  - 가장 큰 VM의 리소스를 기준으로 슬롯을 만든다. 해당 슬롯 1개당 VM을 할당할 수 있다.
    - 예를 들어서 가장 큰 VM이 CPU 2GHz, Memory 2G라면 1슬롯당 2GHz, 2G를 할당할 수 있다.
    - 관리하기가 쉬운 대신에 다이나믹한 워크로드에는 적합하지 않다.
- Dedicated AdmissionControl
  - 하나의 호스트에 VM을 올리지 않고 순수히 VM의 failover를 보장하기 위해 사용하는 방식이다.

### 데이터스토어 하트비트 정책

![2024-09-23 18 50 31](https://github.com/user-attachments/assets/05f4b70d-9ef0-4bf3-afb8-3a9d720becf8)

- **Automatically select datastore accessible from the host**

  - vSphere가 자동으로 데이터스토어를 지정해주는 옵션

- **Use datastores only from specified list**

  - 사용자가 고성능 스토리지(SSD)와 같은 스토리지를 통한 데이터스토어 하트비트를 사용하고 싶을 때 임의로 스토리지 리스트를 지정해 줄 수 있다.

- **Use datastores form the specified list and complement automatically if needed**
  - 사용자가 지정한 스토리지가 장애가 났을 때 vSphere가 다른 스토리지의 데이터스토어를 자동으로 설정해준다.

### Advanced Setting

![2024-09-23 18 55 47](https://github.com/user-attachments/assets/f71832d1-655e-4200-b698-50677a8e0de0)

- **das.isolationaddress1**
  - host가 network heartbeat를 보내지 못할 때 네트워크 격리 또는 분리 상태가 된다.
    - 이는 management network에서 신호를 주고받는데 managerment network의 gateway가 문제가 생겼을 땐 모든 host가 네트워크 격리 상태가 된다.
    - 이를 방지하기 위해서 임의로 gateway ip 이외에 신뢰할 수 있는 ip를 추가로 등록하여 네트워크 격리 상태를 감지하는 기준을 확장한다.
- **das.usedefaultisolationaddress**
  - 위의 옵션을 사용할 것인지 사용하지 않을 것인지 결정하는 옵션이다.

## 💡 결론 및 고찰

### 대규모 인프라 서비스로 전환

![2024-09-23 19 09 31](https://github.com/user-attachments/assets/318586a5-be0e-4cd1-b88e-9f468e35224a)

### 워크로드에 따른 Admission Control 정책설정

![2024-09-23 19 01 46](https://github.com/user-attachments/assets/ebb4365a-7f75-4ca1-b412-a183a4a3073c)

- 쿠폰 이벤트, 수강신청처럼 일시적으로 큰 리소스를 VM에 할당해야 한다면 AdmissionControl을 동적인 워크로드에 적합한 Percentage 전략을 사용해야 한다.
  - Slot 전략을 사용하게 되면 일시적으로 크게 늘린 VM의 리소스만큼 Slot의 크기가 결정된다.

### vSphere Cluster 기능 적극적으로 활용하기

![2024-09-23 19 03 51](https://github.com/user-attachments/assets/d075b3ba-62de-4707-a288-4fd775c80c14)

- vSphere Cluster에서 제공하는 HA, DRS, FT, vMotion 등을 적극적으로 활용해야 한다.
  - DRS를 사용하여 비대해진 클러스터의 각 호스트의 자원에 적합하도록 VM을 분배시켜야 한다.
  - FT를 사용하여 다운타임을 허용하지 않는 서비스의 고가용성을 보장한다.
  - vMotion을 사용하여 사용자에게 무중단 서비스를 제공하면서 유지 보수를 진행한다.

### 3Tier 구조에서 HCI로 전환하기

![2024-09-23 19 06 02](https://github.com/user-attachments/assets/4479f581-1db8-4b73-82f7-bf7c56026f15)

- 서버, 스토리지 스위치, 스토리지 3단계로 분리되어 있는 아키텍처를 HCI로 변경하여 서버 증설의 비용을 줄인다.
  - vSan, vSphere, NSX를 사용하여 스토리지, 서버, 네트워크를 가상화하여 관리한다.
  - 다만 레거시 환경의 통합과 비용이 더 들 수 있으므로 장점과 단점을 잘 고려하여 도입해야 한다.
