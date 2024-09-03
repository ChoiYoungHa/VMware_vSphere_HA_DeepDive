# vSphere HA

### 기본 호스트와 보조 호스트
- **기본 호스트와 보조 호스트의 개념**:
    - **기본 호스트**는 클러스터 내에서 주요 관리 역할을 담당하는 호스트입니다. 클러스터에서 데이터스토어를 가장 많이 마운트한 호스트가 기본 호스트로 선택될 가능성이 높습니다.
    - **보조 호스트**는 기본 호스트의 지시에 따라 동작하며, 주로 가상 시스템의 실행 및 모니터링을 담당합니다.
- **기본 호스트의 책임**:
    - 보조 호스트의 상태를 모니터링하고, 보조 호스트에 장애가 발생하면 이를 복구합니다.
    - 클러스터 내의 모든 보호된 가상 시스템의 전원 상태를 모니터링하고, 장애 발생 시 해당 가상 시스템을 다시 시작하도록 오케스트레이션합니다.
    - 클러스터의 호스트 및 보호된 가상 시스템의 목록을 관리합니다.
    - 클러스터의 상태를 vCenter Server에 보고하는 관리 인터페이스 역할을 수행합니다.
- **보조 호스트의 역할**:
    - 로컬에서 가상 시스템을 실행하고, 가상 시스템의 런타임 상태를 모니터링하며, 상태를 기본 호스트에 보고합니다.
- **기본 호스트 교체**:
    - 기본 호스트에 문제가 발생하면 새로운 기본 호스트가 선택되며, 이 새로운 기본 호스트는 기존의 보호된 가상 시스템 목록을 이어받아 관리하게 됩니다.
- **주의사항**:
    - 클러스터에서 호스트의 연결이 끊어지면, 해당 호스트에 등록된 가상 시스템은 더 이상 vSphere HA에 의해 보호되지 않습니다.

### 호스트 장애 유형
1. **장애 (Failure)**: 호스트가 완전히 작동을 중지하는 상황입니다. 기본 호스트는 이러한 장애를 감지하면 해당 호스트에서 실행 중이던 가상 시스템을 다른 호스트로 페일오버합니다.
2. **분리 (Isolation)**: 호스트가 네트워크에서 분리되어 다른 호스트와 통신할 수 없는 상태입니다. 분리된 호스트는 관리 네트워크에서 vSphere HA 에이전트의 트래픽을 감지할 수 없으며, 이 상태에서 클러스터 분리 주소에 대한 ping을 시도하게 됩니다. 분리된 호스트의 가상 시스템 상태를 기본 호스트가 모니터링하며, 필요 시 가상 시스템을 다시 시작합니다.
3. **파티션 (Partition)**: 호스트가 네트워크 파티션 상태로, 기본 호스트와의 네트워크 연결이 끊어진 경우입니다. 이 경우에도 기본 호스트는 보조 호스트가 데이터스토어와의 하트비트를 교환하고 있는지 확인하여 상태를 모니터링합니다.

**Proactive HA 실패**는 호스트 구성 요소의 일부가 장애를 겪었으나 가상 시스템 동작에 아직 심각한 영향을 미치지 않은 상황을 의미합니다. Proactive HA 실패가 발생하면 vSphere Client에서 VM을 다른 호스트로 이동시키고, 영향을 받은 호스트를 차단 모드나 유지 보수 모드로 설정할 수 있습니다.

### 호스트 문제에 대한 응답 결정
- **호스트 분리 응답**: 네트워크 연결이 끊긴 호스트의 VM을 어떻게 처리할지 결정하는 설정입니다. VM을 종료한 후 다른 호스트에서 다시 시작할 수 있습니다. 이 기능을 사용하려면 호스트 모니터링이 활성화되어 있어야 하며, VMware Tools가 설치된 가상 머신에서 "종료 후 다시 시작" 옵션을 사용하면 최신 상태를 보존할 수 있습니다.
- **분할 브레인(Split-Brain) 상태 방지**: 호스트가 분리되거나 분할된 경우 VM이 두 인스턴스로 실행되는 것을 방지하기 위한 방법입니다. VM 구성 요소 보호(VMCP)를 사용하여 이 상태를 방지하고, 문제가 발생하면 ESXi 호스트가 디스크 잠금 문제를 해결하는 절차를 따릅니다.
- **가상 시스템 종속성**: 클러스터 내에서 VM 간 종속성을 설정하여 특정 VM이 다른 VM보다 먼저 시작되도록 설정할 수 있습니다. 예를 들어, DNS나 DHCP와 같은 중요한 서비스가 다른 서비스보다 먼저 시작되도록 설정할 수 있습니다.
- **가상 시스템 재시작 시 고려 요소**: 호스트 장애 이후 VM을 다시 시작하기 위해 기본 호스트가 고려해야 하는 요소들입니다. 여기에는 파일 액세스, VM과 호스트의 호환성, 리소스 예약, 호스트 제한, 그리고 기능적 제약 조건 등이 포함됩니다. 이러한 조건이 충족되지 않으면 vSphere HA는 VM을 시작할 수 없음을 알리는 이벤트를 생성하고, 조건이 바뀌면 다시 시도합니다.
### VM 및 애플리케이션 모니터링
- **VM 모니터링**: VMware Tools 하트비트가 설정된 시간 동안 수신되지 않으면, vSphere HA(High Availability) 기능을 통해 가상 머신을 다시 시작할 수 있습니다. 이는 VMware Tools 프로세스의 하트비트 및 I/O 활동을 기반으로 가상 머신이 정상적으로 작동하고 있는지를 평가합니다.
- **애플리케이션 모니터링**: VM 모니터링과 유사하게, 실행 중인 애플리케이션이 하트비트를 전송하지 않으면 가상 머신이 다시 시작될 수 있습니다. 이를 위해 애플리케이션이 VMware의 SDK나 VMware 애플리케이션 모니터링을 지원하는 방식으로 구성되어야 합니다.
- **모니터링 감도 설정**: 모니터링 감도를 높이면 장애를 더 빨리 감지하지만, 리소스 부족 등으로 인해 정상적인 가상 머신이나 애플리케이션이 오탐으로 인해 재설정될 가능성도 있습니다. 감도가 낮으면 서비스 중단 시간이 길어질 수 있습니다.
- **I/O 통계 간격**: 하트비트가 수신되지 않더라도 I/O 활동이 있는 경우 불필요한 재설정을 피할 수 있도록 모니터링합니다. 기본 간격은 120초이며, 고급 설정에서 변경할 수 있습니다.
- **재설정 정책**: 특정 시간 내에 최대 3회까지 가상 머신을 재설정할 수 있으며, 이 설정은 사용자 지정 가능합니다. 또한, 전원 상태 변경이나 vMotion 작업 시 재설정 통계가 초기화됩니다.
### VM 구성 요소 보호(VMCP)
#### 데이터스토어 엑세스 장애의 유형
1. **PDL (Permanent Device Loss)**:
    - PDL은 호스트가 더 이상 데이터스토어에 접근할 수 없음을 나타내는 복구 불가능한 장애입니다.
    - 이 상황에서는 가상 시스템이 종료된 후에만 다시 시작할 수 있습니다.
    - VMCP는 PDL이 발생하면 해당 가상 시스템을 종료하고 다른 호스트에서 재시작할 수 있습니다.
2. **APD (All Paths Down)**:
    - APD는 일시적이거나 원인을 알 수 없는 스토리지 경로 손실을 의미하며, 복구 가능성이 있습니다.
    - APD에 대응하기 위해 VMCP는 보다 복잡한 설정을 요구합니다. 관리자는 다양한 정책(보수적 또는 적극적) 중에서 선택하여 APD 상황에 대응할 수 있습니다.
#### PDL 및 APD에 대한 설정
- **PDL 장애**: 이벤트 발생 시 문제를 알리고, 가상 시스템을 종료 후 재시작할 수 있습니다.
- **APD 장애**: 보다 복잡한 설정이 가능하며, 보수적 또는 적극적인 정책을 선택하여 가상 시스템을 종료 후 재시작할 수 있습니다.

### 네트워크 파티션
네트워크 파티션(Network Partition)은 vSphere HA(High Availability) 클러스터 환경에서 발생할 수 있는 문제로, 관리 네트워크에 오류가 발생했을 때 클러스터의 일부 호스트가 다른 호스트와 통신할 수 없는 상태를 말합니다. 이로 인해 클러스터 내에서 여러 개의 파티션이 생기며, 이는 가상 시스템 보호와 클러스터 관리에 부정적인 영향을 미칩니다.

### 네트워크 파티션의 영향

1. **가상 시스템 보호**:
    - 네트워크 파티션이 발생하면, vCenter Server는 가상 시스템의 전원이 켜지는 것을 허용하지만, 해당 가상 시스템이 그 시스템을 관리하는 기본 호스트와 동일한 파티션에 있을 경우에만 보호됩니다.
    - 기본 호스트는 가상 시스템의 구성 파일이 저장된 데이터스토어에 배타적으로 접근할 수 있으며, 이 파일을 잠금으로써 가상 시스템을 관리합니다.
2. **클러스터 관리**:
    - 네트워크 파티션이 발생하면 vCenter Server는 기본 호스트와 통신할 수 있지만, 일부 보조 호스트와는 통신하지 못하게 됩니다.
    - 이로 인해 vSphere HA에 영향을 주는 구성 변경 사항이 모든 파티션에 즉시 적용되지 않으며, 일부 파티션은 이전 구성으로, 다른 파티션은 새로운 구성으로 작동할 수 있습니다.
    -
### 데이터스토어 하트비트
- **데이터스토어 하트비트의 역할**:
    - VMware vSphere HA 클러스터에서 기본 호스트가 관리 네트워크를 통해 보조 호스트와 통신할 수 없는 경우, 데이터스토어 하트비트를 사용하여 보조 호스트의 상태(장애 여부, 네트워크 파티션 여부 등)를 확인합니다.
    - 보조 호스트의 데이터스토어 하트비트가 중지되면 해당 호스트가 장애가 발생한 것으로 간주되며, 그 호스트에서 실행 중이던 가상 시스템이 다른 위치에서 재시작됩니다.
- **하트비트 데이터스토어 선택**:
    - VMware vCenter Server는 기본적으로 하트비트에 사용할 데이터스토어 집합을 선택합니다. 선택된 데이터스토어는 최대한 많은 호스트가 액세스할 수 있도록 구성되며, 동일한 LUN 또는 NFS 서버에 백업되는 것을 피하도록 설정됩니다.
    - 고급 옵션인 `das.heartbeatdsperhost`를 통해 하트비트 데이터스토어의 개수를 호스트당 최대 5개까지 지정할 수 있습니다.
- **데이터스토어 하트비트의 디렉토리 구조**:
    - vSphere HA는 각 데이터스토어의 루트에 `.vSphere-HA`라는 디렉토리를 생성하여 하트비트 관련 데이터를 저장합니다. 이 디렉토리와 그 하위 디렉토리는 루트만 접근할 수 있으며, 삭제나 수정이 금지됩니다.
- **오버헤드 및 성능 영향**:
    - vSphere HA에서 데이터스토어를 사용하더라도 성능 오버헤드는 거의 발생하지 않으며, 다른 데이터스토어 작업의 성능에 영향을 주지 않습니다.
- **vSAN 데이터스토어**:
    - vSAN 데이터스토어는 데이터스토어 하트비트에 사용할 수 없으므로, 클러스터의 모든 호스트가 접근할 수 있는 다른 공유 스토리지가 필요합니다. 대체로 vSAN 네트워크에 대해 독립적인 네트워크 경로를 통해 접근 가능한 스토리지가 있다면 이를 하트비트 데이터스토어로 사용할 수 있습니다.
    -
### vSphere HA 보안
- **방화벽 포트 관리**:
    - vSphere HA는 에이전트 간 통신에 TCP 및 UDP 포트 8182를 사용합니다. 이 포트는 필요할 때만 열리며, 자동으로 열리고 닫힙니다. 이를 통해 불필요한 네트워크 공격을 방지할 수 있습니다.
- **파일 시스템 사용 권한 보호**:
    - vSphere HA는 구성 파일을 로컬 스토리지나 ramdisk에 저장하며, 이러한 파일은 파일 시스템 권한으로 보호됩니다. 루트 사용자만 이 파일에 접근할 수 있습니다. 특히, Auto Deploy 기능을 사용하는 경우 로컬 스토리지가 없는 호스트만 지원됩니다.
- **상세 로깅**:
    - vSphere HA의 로그는 ESXi 호스트의 syslog에 저장되며, 로그 파일에는 서비스 오류 도메인 관리자를 나타내는 "fdm" 접미사가 붙습니다. 기존 ESXi 호스트의 경우, 로그는 로컬 디스크와 syslog에 기록됩니다.
- **보안된 vSphere HA 로그인**:
    - vSphere HA는 vCenter Server에서 생성된 `vpxuser` 계정을 사용하여 에이전트에 로그인합니다. 이 계정은 주기적으로 암호가 변경되며, 변경 주기는 `VirtualCenter.VimPasswordExpirationInDays` 설정에 따라 조정됩니다. 호스트의 루트 사용자만 에이전트에 로그인할 수 있습니다.
- **통신 보안**:
    - vCenter Server와 vSphere HA 에이전트 간의 모든 통신은 SSL을 통해 암호화됩니다. 에이전트 간 통신에서도 SSL이 사용되며, 특히 선택 메시지는 SSL로 검증되어 악성 에이전트가 기본 호스트로 선택되지 않도록 합니다.
- **호스트 SSL 인증서 확인**:
    - vSphere HA를 사용하려면 각 호스트에 확인된 SSL 인증서가 필요합니다. 처음 부팅 시 각 호스트는 자체 서명된 인증서를 생성하며, 이후 인증서를 기관에서 발행한 인증서로 교체할 수 있습니다. 인증서를 교체한 경우, vSphere HA를 재구성해야 합니다.