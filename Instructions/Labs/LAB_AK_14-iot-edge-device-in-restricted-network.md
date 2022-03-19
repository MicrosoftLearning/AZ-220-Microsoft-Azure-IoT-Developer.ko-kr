---
lab:
  title: '랩 14: 제한된 네트워크 및 오프라인에서 IoT Edge 디바이스 실행'
  module: 'Module 7: Azure IoT Edge Module'
ms.openlocfilehash: 1413872367a9e3f0364b162a1d671fe8d122c469
ms.sourcegitcommit: 06dc1e6caa88a09b1246dd1161f15f619db9c6f8
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/10/2022
ms.locfileid: "138421536"
---
# <a name="run-an-iot-edge-device-in-restricted-network-and-offline"></a>제한된 네트워크 및 오프라인에서 IoT Edge 디바이스 실행

## <a name="lab-scenario"></a>랩 시나리오

Contoso의 치즈 포장 및 운송 시설에서 구현한 컨베이어 벨트 모니터링 시스템이 성과를 거두고 있습니다. 이제 시스템은 벨트의 진동 수준을 관리하는 작업을 지원하는 Azure IoT Hub로 원격 분석 데이터를 전송하고 있으며, 새로운 IoT Edge 디바이스는 시스템을 통과하는 치즈 패키지 수를 추적하여 인벤토리를 관리하는 데 도움을 줍니다.

관리자는 시스템이 네트워크 중단에 탄력적이기를 원하며, 네트워크 중단은 치즈 처리 시설의 일부 영역에서 가끔씩 발생합니다. 또한, IT 부서는 네트워크의 부하를 분산하기 위해 하루 중 특정 시간에 중요하지 않은 원격 분석 데이터를 일괄 업로드하도록 시스템을 최적화할 것을 요청했습니다.

이에 따라 네트워크 중단될 경우 오프라인 시나리오를 지원하도록 IoT Edge를 구성하는 것을 제안하고, 센서의 원격 분석을 로컬(디바이스에서)로 저장하고, Edge 디바이스가 지정된 시간에 정기적으로 동기화하도록 구성하려 합니다.

다음 리소스가 만들어집니다.

![랩 14 아키텍처](media/LAB_AK_14-architecture.png)

## <a name="in-this-lab"></a>랩 내용

이 랩에서는 다음 활동을 완료할 예정입니다.

* 랩 필수 구성 요소(필수 Azure 리소스) 구성
* Azure IoT Edge 사용 Linux VM 배포
* 하위 IoT 디바이스가 있는 IoT Edge 상위 디바이스 설정
* IoT Edge 디바이스를 게이트웨이로 구성
* Azure CLI를 사용하여 IoT Edge 게이트웨이 디바이스 인바운드 포트 열기
* IoT Edge 게이트웨이 디바이스 TTL(Time to Live) 및 메시지 저장소 구성
* IoT Edge 게이트웨이에 하위 IoT 디바이스 연결
* 디바이스 연결 및 오프라인 지원 테스트

## <a name="lab-instructions"></a>랩 지침

### <a name="exercise-1-configure-lab-prerequisites"></a>연습 1: 랩 필수 구성 요소 구성

이 랩에서는 다음과 같은 Azure 리소스를 사용할 수 있다고 가정합니다.

| 리소스 종류 | 리소스 이름 |
| :-- | :-- |
| 리소스 그룹 | rg-az220 |
| IoT Hub | iot-az220-training-{사용자 ID} |
| IoT Edge 디바이스 | vm-az220-training-gw0002-{사용자 ID} |
| IoT 디바이스 | sensor-th-0084 |

이러한 리소스를 사용할 수 있게 하려면 다음 단계를 완료합니다.

1. 가상 머신 환경에서 Microsoft Edge 브라우저 창을 열고 다음 웹 주소로 이동합니다.
 
    +++https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoftLearning%2FAZ-220-Microsoft-Azure-IoT-Developer%2Fbicep%2FAllfiles%2FARM%2Flab14.json+++

    > **참고**: 녹색 “T” 기호가 표시될 때마다(예: +++이 text+++를 입력) 연결된 텍스트를 클릭하면 가상 머신 환경 내의 현재 필드에 정보가 입력됩니다.

1. Azure Portal에 로그인하라는 메시지가 표시되면 이 과정에 사용 중인 Azure 자격 증명을 입력합니다.

    **사용자 지정 배포** 페이지가 표시됩니다.

1. **프로젝트 세부 정보** 의 **구독** 드롭다운에서 이 과정에서 사용할 Azure 구독이 선택되어 있는지 확인합니다.

1. **리소스 그룹** 드롭다운에서 **rg-az220** 을 선택합니다.

    > **참고**: **rg-az220** 이 목록에 없는 경우:
    >
    > 1. **리소스 그룹** 드롭다운에서 **새로 만들기** 를 클릭합니다.
    > 1. **이름** 아래에서 **rg-az220** 을 입력합니다.
    > 1. **확인** 을 클릭합니다.

1. **인스턴스 세부 정보** 의 **지역** 드롭다운에서 가장 가까운 지역을 선택합니다.

    > **참고**: **rg-az220** 그룹이 이미 있는 경우 **지역** 필드는 리소스 그룹에서 사용하는 지역으로 설정되며 읽기 전용입니다.

1. **사용자 ID** 필드에 연습 1에서 만든 고유 ID를 입력합니다.

1. **과정 ID** 필드에 **az220** 을 입력합니다.

1. **VM 리소스 그룹** 필드에 **rg-az220vm** 을 입력합니다.

1. **관리자 사용자 이름** 에 사용할 계정 이름을 입력합니다.

1. **인증 유형** 필드에서 **암호** 를 선택합니다.

1. **관리자 암호 또는 키** 필드에 관리자 계정에 사용할 암호를 입력합니다.

1. 템플릿의 유효성을 검사하려면 **검토 및 만들기** 를 클릭합니다.

1. 유효성 검사를 통과하면 **만들기** 를 클릭합니다.

    배포가 시작됩니다.

1. 배포가 완료되면 왼쪽 탐색 영역에서 템플릿의 출력 값을 검토하려면 **출력** 을 클릭합니다.

    나중에 사용할 수 있도록 출력을 기록해 둡니다.
    * connectionString
    * deviceConnectionString
    * gatewayConnectionString
    * devicePrimaryKey
    * publicFQDN
    * publicSSH

이제 리소스가 만들어졌습니다.

> **참고**: ARM 템플릿은 VM 및 IoT Edge를 프로비저닝하는 것 외에도 인바운드 트래픽에 대한 방화벽 규칙을 구성하고 자식 디바이스를 만들었습니다.

### <a name="exercise-2-download-device-ca-certificate"></a>연습 2: 디바이스 CA 인증서 다운로드

이 연습에서는 방금 만든 **vm-az220-training-gw0002-{사용자 ID}** Virtual Machine을 살펴보고 생성된 테스트 인증서를 클라우드 셸에 다운로드합니다.

#### <a name="task-1-connect-to-the-vm"></a>작업 1: VM에 연결

1. IoT Edge 가상 머신이 성공적으로 배포되었는지 확인합니다.

    Azure Portal에서 알림 창을 확인할 수 있습니다.

1. **rg-az220vm** 리소스 그룹이 Azure 대시보드에 고정되었는지 확인합니다.

    대시보드에 리소스 그룹을 고정하려면 Azure 대시보드로 이동하여 다음 작업을 완료합니다.

    * Azure Portal 메뉴에서 **리소스 그룹** 을 클릭합니다.
    * **리소스 그룹** 블레이드의 **이름** 에서 **rg-az220vm** 리소스 그룹을 찾습니다.
    * **rg-az220vm** 행에서 블레이드의 오른쪽에 있는 **...** 를 클릭하고 **대시보드에 고정** 을 클릭합니다.

    RG 타일과 목록의 리소스에 더 쉽게 액세스할 수 있도록 대시보드를 편집할 수 있습니다.

1. Azure Portal 도구 모음에서 **Cloud Shell** 을 클릭합니다.

1. Cloud Shell 명령 프롬프트에서 이전 작업에서 적어 둔 **ssh** 명령을 붙여넣어 예를 들어 **ssh vmadmin@vm-az220-training-gw0002-dm080321.centralus.cloudapp.azure.com** 과 같이 만든 다음 **Enter** 키를 누릅니다.

1. **계속 연결하시겠습니까?** 라는 메시지가 나타나면 **네** 라고 입력하고 **Enter 키** 를 누르세요.

    해당 메시지는 VM에 대한 연결 보안에 사용된 인증서가 자체 서명된 인증서이므로 보안 확인입니다. 이 메시지에 대한 대답은 이후 연결을 위해 기억되며, 이 메시지는 첫 번째 연결에서만 표시됩니다.

1. 암호를 입력하라는 메시지가 표시되면 Edge 게이트웨이 VM이 프로비전될 때 만든 관리자 암호를 입력합니다.

1. 연결되면 터미널은 다음과 같이 Linux VM의 이름을 표시하도록 변경됩니다. 이는 어떤 VM에 연결되었는지 알려줍니다.

    ``` bash
    username@vm-az220-training-gw0002-{your-id}:~$
    ```

1. Virtual Machines 공용 IP 주소를 확인하려면 다음 명령을 입력합니다.

    ```bash
    nslookup vm-az220-training-gw0001-{your-id}.centralus.cloudapp.azure.com
    ```

    다음과 유사하게 출력됩니다.

    ```bash
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    Non-authoritative answer:
    Name:   vm-az220-training-gw0001-{your-id}}.centralus.cloudapp.azure.com
    Address: 168.61.181.131
    ```

    VM의 공용 IP는 최종 **주소** 값(이 경우 **168.61.181.131**)입니다.

    > **중요**: 나중에 필요하므로 이 IP 주소를 기록해 둡니다. 일반적으로 IP 주소는 VM이 다시 시작될 때마다 변경됩니다.

#### <a name="task-2-explore-the-iot-edge-configuration"></a>작업 2: IoT Edge 구성 탐색

VM을 처음 시작하는 동안 IoT Edge를 구성한 스크립트가 실행되었습니다. 이 스크립트는 다음 작업을 수행했습니다.

* **aziot-identity-service** 패키지 설치함
* **aziot-edge** 패키지 설치함
* 초기 버전의 **config.toml**(IoT Edge용 구성 파일)을 **/etc/aziot/config.toml** 에 다운로드함
* ARM 템플릿이 **/etc/aziot/config.toml** 에 실행될 때 제공된 디바이스 연결 문자열을 추가함
* [Iot Edge git 리포지토리](https://github.com/Azure/iotedge.git)를 **/etc/gw-ssl/iotedge** 에 복제함
* **/tmp/lab12** 디렉터리를 만들고 **/etc/gw-ssl/iotedge** 에서 IoT Edge 게이트웨이 SSL 테스트 도구를 복사함
* **/tmp/lab12** 에서 테스트 SSL 인증서를 생성하고 **/etc/aziot** 에 복사함
* 인증서를 **/etc/aziot/config.toml** 에 추가함
* 업데이트된 **/etc/aziot/config.toml** 을 IoT Edge 런타임에 적용함

1. 설치된 IoT Edge 버전을 확인하려면 다음 명령을 입력합니다.

    ```bash
    iotedge --version
    ```

    이 문서를 작성할 당시의 버전은 `iotedge 1.2.3`입니다.

1. IoT Edge 구성을 보려면 다음 명령을 입력합니다.

    ```bash
    cat /etc/aziot/config.toml
    ```

    다음과 유사하게 출력됩니다.

    ```s
    [provisioning]
    source = "manual"
    connection_string = "HostName=iot-az220-training-dm080221.azure-devices.net;DeviceId=sensor-th-0067;SharedAccessKey=2Zv4wruDViwldezt0iNMtO1mA340tM8fnmxgoQ3k0II="

    [agent]
    name = "edgeAgent"
    type = "docker"

    [agent.config]
    image = "mcr.microsoft.com/azureiotedge-agent:1.2"

    [connect]
    workload_uri = "unix:///var/run/iotedge/workload.sock"
    management_uri = "unix:///var/run/iotedge/mgmt.sock"

    [listen]
    workload_uri = "fd://aziot-edged.workload.socket"
    management_uri = "fd://aziot-edged.mgmt.socket"

    [moby_runtime]
    uri = "unix:///var/run/docker.sock"
    network = "azure-iot-edge"

    trust_bundle_cert = 'file:///etc/aziot/azure-iot-test-only.root.ca.cert.pem'

    [edge_ca]
    cert = 'file:///etc/aziot/iot-edge-device-ca-MyEdgeDeviceCA-full-chain.cert.pem'
    pk = 'file:///etc/aziot/iot-edge-device-ca-MyEdgeDeviceCA.key.pem'
    ```

    설치하는 동안 **connection_string**, **trust_bundle_cert**, **cert** 및 **pk** 값이 업데이트되었습니다.

1. IoT Edge 디먼이 실행되고 있는지 확인하려면 다음 명령을 입력합니다.

    ```bash
    sudo iotedge system status
    ```

    이 명령은 다음과 유사한 출력을 표시합니다.

    ```bash
    System services:
        aziot-edged             Running
        aziot-identityd         Running
        aziot-keyd              Running
        aziot-certd             Running
        aziot-tpmd              Ready

    Use 'iotedge system logs' to check for non-fatal errors.
    Use 'iotedge check' to diagnose connectivity and configuration issues.
    ```

1. IoT Edge 런타임이 연결되었는지 확인하려면 다음 명령을 실행합니다.

    ```bash
    sudo iotedge check
    ```

    그러면 여러 검사가 실행되어 결과가 표시됩니다. 이 랩에서 **Configuration checks** 경고/오류는 무시하면 됩니다. **Connectivity checks** 는 성공하며, 다음과 같은 결과가 표시됩니다.

    ```bash
    Connectivity checks (aziot-identity-service)
    --------------------------------------------
    √ host can connect to and perform TLS handshake with iothub AMQP port - OK
    √ host can connect to and perform TLS handshake with iothub HTTPS / WebSockets port - OK
    √ host can connect to and perform TLS handshake with iothub MQTT port - OK

    Configuration checks
    --------------------
    ** entries removed for legibility **

    Connectivity checks
    -------------------
    √ container on the default network can connect to IoT Hub AMQP port - OK
    √ container on the default network can connect to IoT Hub HTTPS / WebSockets port - OK
    √ container on the default network can connect to IoT Hub MQTT port - OK
    √ container on the IoT Edge module network can connect to IoT Hub AMQP port - OK
    √ container on the IoT Edge module network can connect to IoT Hub HTTPS / WebSockets port - OK
    √ container on the IoT Edge module network can connect to IoT Hub MQTT port - OK
    ```

    연결이 실패하면 **config.toml** 의 연결 문자열 값을 다시 확인합니다.

1. VM 셸을 종료하려면 다음 명령을 입력합니다.

    ```bash
    exit
    ```

    VM에 대한 연결이 닫히고 클라우드 셸 프롬프트가 표시됩니다.

#### <a name="task-3-download-ssl-certs"></a>작업 3: SSL 인증서 다운로드

다음으로, 리프 디바이스와 IoT Edge 게이트웨이 간의 통신을 암호화하는 데 사용할 수 있도록 **vm-az220-training-gw0002-{사용자 ID}** 가상 머신에서 **MyEdgeDeviceCA** 인증서를 “다운로드”해야 합니다.

1. **vm-az220-training-gw0002-{사용자 ID}** 가상 머신에서 **Cloud Shell** 스토리지로 **/tmp/lab12** 디렉터리를 다운로드하려면 Cloud Shell 명령 프롬프트에서 다음 명령을 입력합니다.

    ```bash
    mkdir lab12
    scp -r -p <username>@<FQDN>:/tmp/lab12 .
    ```

    > **참고**: **<username>** 자리 표시자를 VM에 대한 관리 사용자의 사용자 이름으로 바꾸고 **<FQDN>** 자리 표시자를 VM의 정규화된 도메인 이름으로 바꿉니다. 필요한 경우 SSH 세션을 여는 데 사용한 명령을 참조하세요.
    > `scp -r -p vmadmin@vm-az220-training-edge0001-dm080321.centralus.cloudapp.azure.com:/tmp/lab12 .`

1. 메시지가 표시되면 VM의 관리자 암호를 입력합니다.

    명령이 실행되면 SSH를 통해 인증서 및 키 파일이 있는 **/tmp/lab12** 디렉터리의 복사본이 Cloud Shell 스토리지에 다운로드됩니다.

1. 파일이 다운로드되었는지 확인하려면 다음 명령을 입력합니다.

    ```bash
    cd lab12
    ls
    ```

    다음 파일이 나열됩니다.

    ```bash
    certGen.sh  csr        index.txt.attr      index.txt.old  openssl_root_ca.cnf  serial
    certs       index.txt  index.txt.attr.old  newcerts       private              serial.old
    ```

    **vm-az220-training-gw0002-{사용자 ID}** 가상 머신에서 파일을 Cloud Shell 스토리지로 복사하면 필요에 따라 IoT Edge 디바이스 인증서 및 키 파일을 로컬 컴퓨터에 쉽게 다운로드할 수 있습니다. `download <filename>` 명령을 사용하여 Cloud Shell에서 파일을 다운로드할 수 있습니다. 이 파일은 랩 뒷부분에서 다운로드합니다.

1. 이 랩의 후반부에 사용하기 위해 루트 인증서를 다운로드하려면 다음 명령을 입력합니다.

    ```bashd
    download ~/lab12/
    ```

### <a name="exercise-3-configure-iot-edge-device-time-to-live-and-message-storage"></a>연습 3: IoT Edge 디바이스 TTL(Time-to-Live) 및 메시지 스토리지 구성

확장 오프라인 시나리오용으로 IoT Edge 디바이스를 구성할 때는 오프라인 상태를 유지할 수 있는 기간과 로컬 스토리지 설정을 지정합니다. 오프라인 상태를 유지할 수 있는 기간은 대개 TTL(Time to Live)라고 합니다.

TTL(Time to Live)의 기본값은 `7200`(7200초, 2시간)입니다. 이 기본값은 디바이스 사용을 잠시 중단하기에는 충분한 시간입니다. 하지만 장시간 오프라인 모드에서 작동해야 하는 디바이스나 솔루션에는 2시간이 충분하지 않을 수도 있습니다. 장시간 연결이 끊긴 상태에서 원격 분석 데이터 손실 없이 작동하는 솔루션의 경우 IoT Edge 허브 모듈의 TTL 속성을 최대 1,209,600초(2주 TTL 기간)로 구성할 수 있습니다.

IoT Edge 허브 모듈(`$edgeHub`)은 게이트웨이 디바이스에서 실행되는 IoT Edge 허브와 Azure IoT Hub 서비스 간의 통신을 조정하는 데 사용됩니다. 모듈 쌍에 필요한 속성에서 `storeAndForwardConfiguration.timeToLiveSecs` 속성은 Azure IoT Hub 서비스와 같은 라우팅 엔드포인트에서 연결이 끊어진 상태로 IoT Edge 허브가 메시지를 유지하는 시간을 초 단위로 지정합니다. Edge 허브의 `timeToLiveSecs` 속성은 단일 디바이스 또는 대규모 배포의 일부로 특정 디바이스의 배포 매니페스트에 지정할 수 있습니다.

IoT Edge 디바이스는 연결이 끊어졌거나 오프라인 된 상태에서 메시지를 자동으로 저장합니다. 스토리지 위치는 `HostConfig` 개체를 사용하여 구성할 수 있습니다.

이 연습에서는 Azure IoT Hub용 Azure Portal 사용자 인터페이스를 사용하여 단일 IoT Edge 게이트웨이 디바이스에서 Edge 허브(`$edgeHub`) 모듈에 대한 `timeToLiveSecs` 속성을 수정합니다. 또한 메시지를 저장할 IoT Edge 디바이스에서 스토리지 위치를 구성합니다.

#### <a name="task-1-configure-the-edgehub-module-twin"></a>작업 1: $edgeHub 모듈 쌍 구성

1. 필요한 경우 Azure 계정 자격 증명을 사용하여 Azure Portal에 로그인합니다.

    Azure 계정이 두 개 이상인 경우 이 과정에 사용할 구독에 연결된 계정으로 로그인해야 합니다.

1. **rg-az220** 리소스 그룹 타일에서 **iot-az220-training-{사용자 ID}** 를 클릭합니다.

1. IoT Hub 블레이드 왼쪽 메뉴의 **디바이스 관리** 에서 **IoT Edge** 를 클릭합니다.

    이 창을 사용하면 IoT Hub에 연결된 IoT Edge 디바이스를 관리할 수 있습니다.

1. **디바이스 ID** 에서 **vm-az220-training-gw0002-{사용자 ID}** 를 클릭합니다.

1. **모듈** 에서 **$edgeHub** 를 클릭합니다.

    **Edge 허브** 모듈의 모듈 ID 세부 정보 블레이드는 IoT Edge 디바이스의 모듈 ID 쌍 및 기타 리소스에 대한 액세스를 제공합니다.

1. **모듈 ID 세부 정보** 블레이드에서 **모듈 ID 쌍** 을 클릭합니다.

    이 블레이드에는 편집기 창에 JSON으로 표시되는 `vm-az220-training-gw0002-{your-id}/$edgeHub`의 모듈 ID 쌍이 포함되어 있습니다.

1. 잠시 $edgeHub 모듈 ID 쌍의 내용을 살펴봅니다.

    새 디바이스이기 때문에 필요한 속성은 기본적으로 비어 있습니다.

1. **모듈 ID 쌍** 블레이드를 닫습니다.

1. **vm-az220-training-gw0002-{사용자 ID}** 블레이드로 다시 이동합니다.

1. 블레이드 상단에서 **모듈 설정** 을 클릭합니다.

    **디바이스에서 모듈 설정** 블레이드를 사용하면 이 IoT Edge 디바이스에 배포된 IoT Edge 모듈을 만들고 구성할 수 있습니다.

1. **모듈 설정** 블레이드의 **IoT Edge 모듈** 에서 **런타임 설정** 을 클릭합니다.

1. **런타임 설정** 창에서 **Edge 허브** 탭을 선택합니다.

1. **저장소 및 전달 구성 - TTL(Time to Live)(초)** 필드를 찾습니다.

1. **저장소 및 전달 구성 - TTL(Time to Live)(초)** 텍스트 상자에 **1209600** 을 입력합니다.

    이렇게 하면 IoT Edge 디바이스에 2주(최대 시간)의 메시지 TTL(Time to Live) 값이 지정됩니다.

    > **참고**:  Edge 허브(`$edgeHub`) 모듈에 **메시지 TTL(Time to Live)** 을 구성할 때 고려해야 할 것이 몇 가지 있습니다. IoT Edge 디바이스의 연결이 끊어지면 메시지가 로컬 디바이스에 저장됩니다. TTL 기간 동안 저장되는 데이터의 양을 계산하고 해당하는 데이터에 대한 충분한 스토리지 공간이 디바이스에 있는지 확인해야 합니다. 중요한 데이터의 손실을 방지하려면 구성된 스토리지 및 TTL의 양이 솔루션 요구 사항을 충족해야 합니다.
    >
    > 디바이스에 충분한 스토리지가 없으면 더 짧은 TTL을 구성해야 합니다. 메시지 보존 기간이 TTL 시간 제한에 도달하면 아직 Azure IoT Hub으로 전송되지 않은 경우 삭제됩니다.

    IoT Edge 디바이스는 연결이 끊어졌거나 오프라인 된 상태에서 메시지를 자동으로 저장합니다. 스토리지 위치는 `HostConfig` 개체를 사용하여 구성할 수 있습니다.

1. **환경 변수** 영역을 찾습니다.

    메시지 스토리지 위치 구성을 완료하려면 새 환경 변수를 추가해야 합니다.

1. **환경 변수** 의 **이름** 텍스트 상자에 **storageFolder** 를 입력합니다.

1. **환경 변수** 의 **값** 텍스트 상자에 **/iotedge/storage/** 를 입력합니다.

1. **컨테이너 만들기 옵션** 필드를 찾습니다.

    이 필드에는 구성할 수 있는 `HostConfig` JSON 개체가 포함되어 있습니다. 여기서는 `HostConfig` 속성과 환경 변수를 만들어 Edge 디바이스용 스토리지 위치를 구성합니다.

1. `HostConfig` 개체의 `PortBindings` 속성 닫는 괄호 아래에 다음 `Binds` 속성을 추가합니다.

    ```json
    "Binds": [
        "/etc/aziot/storage/:/iotedge/storage/"
    ]
    ```

    > **참고**: `PortBindings` 속성과 `Binds` 속성을 쉼표로 분리해야 합니다.

    **옵션 만들기** 텍스트 상자의 결과 JSON은 다음과 유사해야 합니다.

    ```json
    {
        "HostConfig": {
            "PortBindings": {
                "443/tcp": [
                {
                    "HostPort": "443"
                }
                ],
                "5671/tcp": [
                {
                    "HostPort": "5671"
                }
                ],
                "8883/tcp": [
                {
                    "HostPort": "8883"
                }
                ]
            },
            "Binds": [
                "/etc/aziot/storage/:/iotedge/storage/"
            ]
        }
    }
    ```

    이 `Binds` 값은 Edge 허브 모듈용 Docker 컨테이너의 `/iotedge/storage/` 디렉터리를 실제 IoT Edge 디바이스의 `/etc/aziot/storage/` 호스트 시스템 디렉터리에 매핑하도록 구성합니다.

    값은 `<HostStoragePath>:<ModuleStoragePath>` 형식으로 되어 있습니다. `<HostStoragePath>` 값은 IoT Edge 디바이스의 호스트 디렉터리 위치입니다. `<ModuleStoragePath>`는 컨테이너 내에서 사용할 수 있는 모듈 스토리지 경로입니다. 이러한 두 값은 모두 절대 경로로 지정해야 합니다.

1. **런타임 설정** 창 아래쪽에서 **적용** 을 클릭합니다.

1. **디바이스에서 모듈 설정** 블레이드에서 **검토 + 만들기** 를 클릭합니다.

1. 잠시 배포 매니페스트의 내용을 살펴봅니다.

    배포 매니페스트에서 업데이트를 찾습니다. `$edgeAgent`와 `$edgeHub`에서 찾아야 합니다.

1. 블레이드 하단에서 **만들기** 를 클릭합니다.

    변경 내용이 저장되면 **IoT Edge 디바이스** 에 모듈 구성 변경에 대한 알림이 전달되고 그에 따라 새 설정이 디바이스에 다시 구성됩니다.

    변경 내용이 Azure IoT Edge 디바이스에 전달되면 새 구성으로 **edgeHub** 모듈이 다시 시작됩니다.

    >**참고**: **모듈** 목록에서 **$edgeHub** 모듈의 **런타임 상태** 에 오류가 표시됩니다.

1. 오류 메시지를 검토하려면 **오류** 를 클릭합니다.

    **문제 해결** 페이지에 오류 로그가 표시됩니다. 다음과 유사한 예외가 포함됩니다.

    ```log
    Unhandled exception. System.AggregateException: One or more errors occurred. (Access to the path '/iotedge/storage/edgeHub' is denied.)
    ```

    다음 작업은 이 오류를 해결합니다.

#### <a name="task-2-update-directory-permissions"></a>작업 2: 디렉터리 권한 업데이트

계속하기 전에 IoT Edge Hub 모듈의 사용자 프로필에 **/etc/aziot/storage/** 디렉터리에 대한 읽기, 쓰기 및 실행 권한이 있는지 확인해야 합니다.

1. Azure Portal 도구 모음에서 **Cloud Shell** 을 클릭합니다.

1. Cloud Shell 명령 프롬프트에서 이전 작업에서 적어 둔 **ssh** 명령을 붙여넣어 예를 들어 **ssh vmadmin@vm-az220-training-gw0002-dm080321.centralus.cloudapp.azure.com** 과 같이 만든 다음 **Enter** 키를 누릅니다.

1. **계속 연결하시겠습니까?** 라는 메시지가 나타나면 **네** 라고 입력하고 **Enter 키** 를 누르세요.

    해당 메시지는 VM에 대한 연결 보안에 사용된 인증서가 자체 서명된 인증서이므로 보안 확인입니다. 이 메시지에 대한 대답은 이후 연결을 위해 기억되며, 이 메시지는 첫 번째 연결에서만 표시됩니다.

1. 암호를 입력하라는 메시지가 표시되면 Edge 게이트웨이 VM이 프로비전될 때 만든 관리자 암호를 입력합니다.

1. 연결되면 터미널은 다음과 같이 Linux VM의 이름을 표시하도록 변경됩니다. 이는 어떤 VM에 연결되었는지 알려줍니다.

    ``` bash
    username@vm-az220-training-gw0002-{your-id}:~$
    ```

1. 실행 중인 IoT Edge 모듈을 보려면 다음 명령을 입력합니다.

    ```bash
    iotedge list
    ```

1. 잠시 `iotedge list` 명령의 출력을 검토합니다.

    *edgeHub* 가 시작되지 않은 것을 볼 수 있습니다.

    ```text
    NAME             STATUS           DESCRIPTION                 CONFIG
    edgeAgent        running          Up 4 seconds                mcr.microsoft.com/azureiotedge-agent:1.1
    edgeHub          failed           Failed (139) 0 seconds ago  mcr.microsoft.com/azureiotedge-hub:1.1
    ```

    이것은 *edgeHub* 프로세스에 **/etc/aziot/storage/** 디렉터리에 대한 쓰기 권한이 없기 때문입니다.

1. 디렉터리 사용 권한에 문제가 있는지 확인하려면 다음 명령을 입력합니다.

    ```bash
    iotedge logs edgeHub
    ```

    터미널에서 현재 로그가 출력됩니다. 로그를 스크롤하면 다음과 같은 관련 항목이 표시됩니다.

    ```text
    Unhandled Exception: System.AggregateException: One or more errors occurred. (Access to the path '/iotedge/storage/edgeHub' is denied.) ---> System.UnauthorizedAccessException: Access to the path '/iotedge/storage/edgeHub' is denied. ---> System.IO.IOException: Permission denied
    ```

1. 디렉터리 사용 권한을 업데이트하려면 다음 명령을 입력합니다.

    ```sh
    sudo chown $( whoami ):iotedge /etc/aziot/storage/
    sudo chmod 775 /etc/aziot/storage/
    ```

    첫 번째 명령은 디렉터리 소유자를 현재 사용자로, 소유한 사용자 그룹을 **iotedge** 로 설정합니다. 두 번째 명령은 **iotedge** 그룹의 현재 사용자와 구성원 모두에 대한 모든 권한을 부여합니다. 이렇게 하면 *edgeHub* 모듈이 **/etc/iotedge/storage/** 디렉터리 내에 디렉터리와 파일을 만들 수 있습니다.

    > **참고**: **chown: cannot access '/etc/iotedge/storage/': No such file or directory** 오류가 표시되면 다음 명령을 사용하여 디렉터리를 만들고 위의 명령을 다시 실행합니다.

    ```sh
    sudo mkdir /etc/iotedge/storage
    ```

1. *edgeHub* 모듈을 다시 시작한 후 시작되었는지 확인하려면 다음 명령을 입력합니다.

    ```bash
    iotedge restart edgeHub
    iotedge list
    ```

    >**참고**: 다시 시작의 모듈 이름은 대/소문자를 구분합니다(**edgeHub**).

    *edgeHub* 모듈이 실행 중임을 알 수 있습니다.

    ```text
    NAME             STATUS           DESCRIPTION      CONFIG
    edgeAgent        running          Up 13 minutes    mcr.microsoft.com/azureiotedge-agent:1.1
    edgeHub          running          Up 6 seconds     mcr.microsoft.com/azureiotedge-hub:1.1
    ```

이제 이 IoT Edge 게이트웨이 디바이스에 IoT 디바이스(하위 디바이스/리프)를 연결할 준비가 되었습니다.

### <a name="exercise-4-connect-child-iot-device-to-iot-edge-gateway"></a>연습 4: IoT Edge 게이트웨이에 자식 IoT 디바이스 연결

대칭 키를 사용하여 IoT Hub에 대한 일반 IoT 디바이스를 인증하기 위한 프로세스는 다운스트림(또는 자식/리프) 디바이스에도 적용됩니다. 유일한 차이점은 연결을 라우팅하기 위해 게이트웨이 디바이스에 포인터를 추가하거나 오프라인 시나리오에서 IoT Hub를 대신하여 인증을 처리해야 한다는 것입니다.

> **참고**: 여기서는 랩의 앞부분에서 저장한 **sensor-th-0084** 의 연결 문자열 값을 사용합니다. 연결 문자열의 새 복사본이 필요한 경우 Azure Portal의 Azure IoT Hub에서 액세스할 수 있습니다. IoT Hub의 **IoT 디바이스** 창을 열고 **sensor-th-0084** 를 클릭한 후 **기본 연결 문자열** 을 복사한 다음 텍스트 파일에 저장합니다.

#### <a name="task-1-create-hosts-file-entry"></a>작업 1: 호스트 파일 항목 만들기

이 랩의 이전 버전에서는 FQDN이 디바이스 연결 문자열의 **GatewayHostName** 값으로 사용되지만 현재 버전의 테스트 스크립트에서 생성된 테스트 x509 인증서는 더 이상 이를 지원하지 않습니다. 대신, 호스트 이름만 사용되고 로컬 컴퓨터의 **호스트** 파일에 항목을 만들어서 호스트 이름을 IP 주소로 확인해야 합니다. 다음 단계를 완료하여 호스트 파일에 필요한 항목을 추가합니다.

1. Visual Studio Code를 엽니다.

1. **파일** 메뉴에서 **파일 열기** 를 클릭합니다.

1. 다음 폴더 **c:\\Windows\\System32\\Drivers\\etc\\** 파일로 이동하고 **호스트** 파일을 엽니다.

    > **참고**: **호스트** 파일에 확장명이 없습니다.

1. **호스트** 파일에 다음 줄을 추가하고 그 뒤에 빈 줄을 추가합니다.

    ```text
    {VM Public IP Address} vm-az220-training-gw0002-{your-id}
    {blank line}
    ```

    예를 들면 다음과 같습니다.

    ```text
    168.61.181.131 vm-az220-training-gw0002-dm090821

    ```

1. 파일 저장 - 저장하지 못했다는 메시지가 표시되면 **관리자로 다시 시도...** 를 클릭하고 **사용자 계정 컨트롤** 대화 상자에서 **예** 를 클릭합니다.

이제 로컬 컴퓨터에서 VM 이름을 적절한 IP 주소로 확인할 수 있습니다.

#### <a name="task-1-configure-device-app"></a>작업 1: 디바이스 구성 앱

이 작업에서는 대칭 키를 사용하여 IoT Hub에 연결하도록 다운스트림 IoT 디바이스(하위 또는 리프 디바이스)를 구성합니다. 디바이스는 대칭 키(부모 IoT Edge 디바이스의 게이트웨이 호스트 이름 외에)가 포함된 연결 문자열을 사용하여 IoT Hub 및 부모 IoT Edge 디바이스에 연결하도록 구성됩니다.

1. Windows **파일 탐색기** 앱을 연 다음 **다운로드** 폴더로 이동합니다.

    다운로드 폴더에는 IoT Edge 게이트웨이를 구성할 때 다운로드한 X.509 인증서 파일이 있어야 합니다. IoT 디바이스 앱의 루트 디렉터리에 이 인증서 파일을 복사해야 합니다.

1. **다운로드** 폴더에서 **azure-iot-test-only.root.ca.cert.pem** 을 마우스 오른쪽 단추로 클릭한 다음 **복사** 를 클릭합니다.

    > **참고**: Downloads 폴더에 azure-iot-test-only.root.ca.cert.pem 파일이 이미 있었다면 여기서 필요한 파일의 이름이 azure-iot-test-only.root.ca.cert (1).pem일 수 있습니다. 그러면 대상 폴더에 해당 파일을 추가한 후 이름을 azure-iot-test-only.root.ca.cert.pem으로 바꿔야 합니다.

    이 파일은 다운로드한 X.509 인증서 파일이며 랩 14 /Starter/ChildIoTDevice 디렉터리(하위 IoT 디바이스의 소스 코드가 있는 위치)에 추가됩니다.

1. 랩 14 Starter 폴더로 이동하여 복사한 파일을 **ChildIoTDevice** 폴더에 붙여넣습니다.

1. 복사한 인증서 파일의 이름이 **azure-iot-test-only.root.ca.cert.pem** 인지 확인합니다.

    Downloads 폴더에 azure-iot-test-only.root.ca.cert.pem 파일이 이미 있었다면 파일의 이름이 azure-iot-test-only.root.ca.cert (1).pem으로 지정되었을 수 있습니다.

1. Visual Studio Code의 새 인스턴스를 엽니다.

1. **파일** 메뉴에서 **폴더 열기** 를 클릭합니다.

1. **폴더 열기** 대화 상자에서 랩 14 **Starter** 폴더로 이동하고 **ChildIoTDevice** 를 클릭한 다음 **폴더 선택** 을 클릭합니다.

    이제 탐색기 창에 프로젝트 파일이 표시됩니다.

1. Visual Studio Code **탐색기** 창에서 **Program.cs** 를 클릭합니다.

1. **Program.cs** 파일에서 **connectionString** 변수의 선언을 찾습니다.

1. 자리 표시자 값을 **sensor-th-0084** IoT 디바이스의 기본 연결 문자열로 바꿉니다.

1. 할당된 **connectionString** 값에 **GatewayHostName** 속성을 추가하고 GatewayHostName의 값을 IoT Edge 게이트웨이 디바이스의 전체 DNS 이름으로 설정합니다.

    Edge 게이트웨이 디바이스의 전체 DNS 이름은 디바이스 ID인 **vm-az220-training-gw0002-{사용자 ID}** 에 지정한 지역 및 Azure 상업용 클라우드 도메인 이름(예: **.westus2.cloudapp.azure.com**)이 추가된 형식입니다.

    완성된 연결 문자열 값은 다음 형식과 일치해야 합니다.

    ```text
    HostName=<IoT-Hub-Name>.azure-devices.net;DeviceId=sensor-th-0072;SharedAccessKey=<Primary-Key-for-IoT-Device>;GatewayHostName=<DNS-Name-for-IoT-Edge-Device>
    ```

    위에 표시된 자리 표시자를 적절한 값으로 바꾸세요.

    * **\<IoT-Hub-Name\>** : Azure IoT Hub 이름입니다.
    * **\<Primary-Key-for-IoT-Device\>** : Azure IoT Hub의 **sensor-th-0084** IoT 디바이스용 기본 키입니다.
    * **\<DNS-Name-for-IoT-Edge-Device\>** : **vm-az220-training-gw0002-{사용자 ID}** Edge 디바이스의 호스트 이름입니다.

    **connectionString** 변수 할당 코드는 다음과 같습니다.

    ```csharp
    private readonly static string connectionString = "HostName=iot-az220-training-1119.azure-devices.net;DeviceId=sensor-th-0084;SharedAccessKey=ygNT/WqWs2d8AbVD9NAlxcoSS2rr628fI7YLPzmBdgE=;GatewayHostName=vm-az220-training-gw0002-{your-id}";
    ```

1. **파일** 메뉴에서 **저장** 을 클릭합니다.

1. **보기** 메뉴에서 **터미널** 을 클릭합니다.

    **터미널** 명령 프롬프트에 `/Starter/ChildIoTDevice` 디렉터리가 나열되어 있는지 확인합니다.

1. **ChildIoTDevice** 시뮬레이션된 디바이스를 만들고 실행하려면 다음 명령을 입력합니다.

    ```cmd/sh
    dotnet run
    ```

    > **참고**: 앱이 로컬 컴퓨터에 **X.509 인증서** 를 설치하면(IoT Edge 게이트웨이로 인증할 때 사용할 수 있음) 인증서를 설치할지 묻는 팝업 창이 표시될 수 있습니다. 앱이 인증서를 설치하도록 허용하려면 **예** 를 클릭합니다.

1. 터미널에 표시된 출력을 확인합니다.

    시뮬레이션된 디바이스가 실행되면 콘솔 출력에 Azure IoT Edge 게이트웨이로 전송되는 이벤트가 표시됩니다.

    터미널 출력은 다음과 유사합니다.

    ```cmd/sh
    IoT Hub C# Simulated Cave Device. Ctrl-C to exit.

    User configured CA certificate path: azure-iot-test-only.root.ca.cert.pem
    Attempting to install CA certificate: azure-iot-test-only.root.ca.cert.pem
    Successfully added certificate: azure-iot-test-only.root.ca.cert.pem
    11/27/2019 4:18:26 AM > Sending message: {"temperature":21.768769073192388,"humidity":79.89793652663843}
    11/27/2019 4:18:27 AM > Sending message: {"temperature":28.317862208149332,"humidity":73.60970909409677}
    11/27/2019 4:18:28 AM > Sending message: {"temperature":25.552859350830715,"humidity":72.7897707153064}
    11/27/2019 4:18:29 AM > Sending message: {"temperature":32.81164186439088,"humidity":72.6606041624493}
    ```

1. 다음 연습으로 이동하는 동안 시뮬레이션된 디바이스를 실행 상태로 둡니다.

#### <a name="task-2-test-device-connectivity-and-offline-support"></a>작업 2: 디바이스 연결 및 오프라인 지원 테스트

이 작업에서는 **vm-az220-training-gw0002-{사용자 ID}** IoT Edge Transparent Gateway를 통해 Azure IoT Hub로 전송되는 **sensor-th-0084** 의 이벤트를 모니터링합니다. 그런 다음 **vm-az220-training-gw0002-{your-id}** 와 Azure IoT Hub 간 연결을 중단하여 원격 분석이 하위 IoT 디바이스에서 IoT Edge 게이트웨이로 계속 전송되는지 확인합니다. 그런 다음, Azure IoT Hub에 다시 연결하고 IoT Edge 게이트웨이가 Azure IoT Hub로 원격 분석을 다시 전송하는지 모니터링합니다.

1. 필요한 경우 Azure 계정 자격 증명을 사용하여 Azure Portal에 로그인합니다.

    Azure 계정이 두 개 이상인 경우 이 과정에 사용할 구독에 연결된 계정으로 로그인해야 합니다.

1. Azure Portal 도구 모음에서 **Cloud Shell** 을 클릭합니다.

    환경 드롭다운이 **Bash** 로 설정되어 있는지 확인합니다.

1. Cloud Shell 명령 프롬프트에서 Azure IoT Hub에서 수신되는 이벤트 모니터링을 시작하려면 다음 명령을 입력합니다.

    ```cmd/sh
    az iot hub monitor-events --hub-name iot-az220-training-{your-id}
    ```

    `{your-id}` 자리 표시자를 Azure IoT Hub 인스턴스의 고유한 접미사로 바꾸어야 합니다.

1. Azure IoT Hub로 전송되는 **sensor-th-0084** 의 원격 분석을 확인합니다.

    **sensor-th-0084** 시뮬레이션된 디바이스 애플리케이션은 **vm-az220-training-gw0002-{your-id}** IoT Edge Transparent Gateway 가상 머신으로 원격 분석을 보내도록 구성되었으며 이후에 Azure IoT Hub로 원격 분석을 전송합니다.

    Cloud Shell은 다음과 유사한 이벤트 메시지를 표시하기 시작합니다.

    ```text
    Starting event monitor, use ctrl-c to stop...
    {
        "event": {
            "origin": "sensor-th-0072",
            "module": "",
            "interface": "",
            "component": "",
            "payload": "{\"temperature\":29.995470051651573,\"humidity\":70.47896838303608}"
        }
    }
    {
        "event": {
            "origin": "sensor-th-0072",
            "module": "",
            "interface": "",
            "component": "",
            "payload": "{\"temperature\":28.459910635584922,\"humidity\":60.49697355390386}"
        }
    }
    ```

    > **참고**: 다음으로, **오프라인** 기능을 테스트해야 합니다. 이렇게 하려면 **vm-az220-training-gw0002-{사용자 ID}** 디바이스를 오프라인 상태로 설정해야 합니다. Azure에서 실행 중인 가상 머신이므로 VM의 **네트워크 보안 그룹** 에 **아웃바운드 규칙** 을 추가하여 시뮬레이션할 수 있습니다.

#### <a name="task3-add-rule-to-block-traffic"></a>작업 3: 트래픽을 차단하는 규칙 추가

1. **Azure Portal** 에서 대시보드로 이동한 다음 **rg-az220vm** 리소스 그룹 타일을 찾습니다.

1. **vm-az220-training-gw0002-{사용자 ID}** 가상 머신의 **네트워크 보안 그룹** 을 열려면 리소스 목록에서 **nsg-vm-az220-training-gw0002-{사용자 ID}** 를 클릭합니다.

1. **네트워크 보안 그룹** 블레이드에서 **설정** 아래의 왼쪽 탐색 창에서 **아웃바운드 보안 규칙** 을 클릭합니다.

1. 블레이드 상단에서 **+추가** 를 선택합니다.

1. **아웃바운드 보안 규칙 추가** 창에서 다음 필드 값을 설정합니다.

    * 대상 포트 범위: **\***
    * 작업: **Deny**
    * 우선 순위: 100
    * 이름: **DenyAll**

    **대상 포트 범위** 인 “ **\*** ”는 모든 포트에 규칙을 적용합니다.

1. 블레이드 하단의 **추가** 를 클릭하세요.

1. Azure Portal에서 **Cloud Shell** 로 다시 돌아갑니다.

1. `az iot hub monitor-events` 명령이 계속 실행 중인 경우 **Ctrl + C** 를 눌러 종료합니다.

1. `ssh`를 사용하여 **vm-az220-training-gw0002-{사용자 ID}** VM에 연결하려면 Cloud Shell 명령 프롬프트에서 다음 명령을 입력합니다.

    ```sh
    ssh <username>@<ipaddress>
    ```

    자리 표시자는 `ssh` 명령에 필요한 값으로 바꿔야 합니다.

    | 자리 표시자 | 교체할 값 |
    | :--- | :--- |
    | `<username>` | **IoTEdgeGateaway** 가상 머신에 대한 관리자 **사용자 이름**. **vmadmin** 이어야 합니다.
    | `<ipaddress>` | **vm-az220-training-gw0002-{사용자 ID}** 가상 머신의 공용 IP 주소

1. 메시지가 표시되면 **vm-az220-training-gw0002-{사용자 ID}** 의 관리자 **암호** 를 입력합니다.

    `ssh`를 통해 **vm-az220-training-gw0002-{사용자 ID}** VM에 연결되면 명령 프롬프트가 업데이트됩니다.

1. IoT Edge 런타임을 다시 설정하려면 다음 명령을 입력합니다.

    ```sh
    sudo iotedge system restart
    ```

    이렇게 하면 IoT Edge 런타임이 Azure IoT Hub 서비스와의 연결을 끊은 다음 다시 연결합니다.

1. *edgeHub* 모듈이 올바르게 다시 시작되었는지 확인하려면 다음 명령을 입력합니다.

    ```bash
    iotedge list
    ```

    *edgeHub* 모듈이 성공적으로 다시 시작되지 못한 경우 다음 명령을 입력하여 다시 시작합니다.

    ```bash
    iotedge restart edgeHub
    iotedge list
    ```

1. **vm-az220-training-gw0002-{사용자 ID}** 와의 `ssh` 세션을 종료하려면 다음 명령을 입력합니다.

    ```cmd/sh
    exit
    ```

1. Azure IoT Hub가 수신하는 이벤트를 모니터링하려면 Cloud Shell 명령 프롬프트에서 다음 명령을 입력합니다.

    ```cmd/sh
    az iot hub monitor-events --hub-name iot-az220-training-{your-id}
    ```

    `{your-id}` 자리 표시자를 Azure IoT Hub 인스턴스의 고유한 접미사로 바꾸어야 합니다.

1. **Azure IoT Hub** 에서 수신하는 이벤트가 더 이상 없습니다.

1. Visual Studio Code 창으로 전환합니다.

1. **sensor-th-0084** 시뮬레이션된 디바이스 애플리케이션이 실행 중인 **터미널** 을 엽니다. 그러면 애플리케이션이 **vm-az220-training-gw0002-{사용자 ID}** 로 디바이스 원격 분석을 계속 전송 중임을 확인할 수 있습니다.

    이때 **vm-az220-training-gw0002-{your-id}** 와 Azure IoT Hub의 연결이 끊어집니다. **sensor-th-0084** 에 의한 연결을 계속 인증하고, 자식 디바이스에서 디바이스 원격 분석을 수신합니다. 이 시간 동안 IoT Edge 게이트웨이에서는 구성에 따라 IoT Edge 게이트웨이 디바이스 스토리지에 자식 디바이스의 이벤트 원격 분석을 저장합니다.

1. **Azure Portal** 창으로 전환합니다.

1. **vm-az220-training-gw0002-{사용자 ID}** 의 **네트워크 보안 그룹** 블레이드로 다시 이동합니다.

1. 왼쪽 탐색 메뉴의 **설정** 에서 **아웃바운드 보안 규칙** 을 클릭합니다.

1. **아웃바운드 보안 규칙** 창에서 **DenyAll** 을 클릭합니다.

1. **DenyAll** 창에서 이 거부 규칙을 NSG에서 제거하려면 **삭제** 를 클릭합니다.

1. **보안 규칙 삭제** 프롬프트에서 **예** 를 클릭합니다.

    **vm-az220-training-gw0002-{your-id}** IoT Edge Transparent Gateway가 Azure IoT Hub와 다시 연결되면 연결된 모든 자식 디바이스에서 이벤트 원격 분석을 동기화합니다. 여기에는 연결이 끊어진 동안 전송할 수 없는 저장된 원격 분석과 게이트웨이로 전송되는 모든 원격 분석이 포함됩니다.

    > **참고**:  IoT Edge 게이트웨이 디바이스는 Azure IoT Hub에 다시 연결하고 원격 분석 전송을 다시 시작하는 데 몇 분 정도 걸릴 수 있습니다. 대기한 후에는 `az iot hub monitor-events` 명령 출력에 이벤트가 다시 표시됩니다.

이 랩에서는 Azure IoT Edge Gateway가 로컬 저장소를 사용하여 IoT Hub 연결 중단으로 인해 보낼 수 없는 메시지를 보존할 수 있다는 것을 입증했습니다. 다시 연결된 후 메시지가 전송된다는 것을 확인했습니다.

> **참고**:  랩을 완료한 후에는 터미널에서 **CTRL+C** 를 눌러 디바이스 시뮬레이션 애플리케이션을 종료해야 합니다.
