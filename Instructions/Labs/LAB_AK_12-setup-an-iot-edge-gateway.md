---
lab:
  title: '랩 12: IoT Edge 게이트웨이 설정하기'
  module: 'Module 6: Azure IoT Edge Deployment Process'
ms.openlocfilehash: a710ea3e39a2c63da58925f669b7bdb8f983a4e7
ms.sourcegitcommit: b9f2c53cb54dde700e21476bcc7435310d15445d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/11/2022
ms.locfileid: "141604968"
---
# <a name="setup-an-iot-edge-gateway"></a>IoT Edge 게이트웨이 설정하기

## <a name="lab-scenario"></a>랩 시나리오

이 랩에서는 이론적이며 IoT Edge 디바이스를 게이트웨이로 사용할 수 있는 방법을 설명합니다.

IoT Edge 디바이스를 게이트웨이로 사용하는 데는 투명, 프로토콜 변환 및 ID 변환과 같은 세 가지 패턴이 있습니다.

**투명** – 이론적으로 IoT Hub에 연결할 수 있는 디바이스는 게이트웨이 디바이스에도 연결할 수 있습니다. 다운스트림 디바이스에는 자체 IoT Hub ID가 있으며 MQTT, AMQP 또는 HTTP 프로토콜을 모두 사용하고 있습니다. 게이트웨이는 디바이스와 IoT Hub 간에 통신을 전달하기만 하면 됩니다. 디바이스는 게이트웨이를 통해 클라우드와 통신하고 있음을 인식하지 못하고, IoT Hub의 디바이스와 상호 작용하는 사용자는 중간 게이트웨이 디바이스를 인식하지 못합니다. 따라서 게이트웨이가 투명합니다. IoT Edge 디바이스를 투명 게이트웨이로 사용하는 방법에 대한 자세한 내용은 투명 게이트웨이 만들기를 참조하세요.

**프로토콜 변환** – MQTT, AMQP 또는 HTTP를 지원하지 않는 디바이스는 불투명 게이트웨이 패턴이라고도 하며, 게이트웨이 디바이스를 사용하여 데이터를 대신하여 IoT Hub에 보냅니다. 게이트웨이는 다운스트림 디바이스에서 사용하는 프로토콜을 이해하며 IoT Hub에 ID가 있는 유일한 디바이스입니다. 모든 정보는 하나의 디바이스인 게이트웨이에서 오는 것처럼 보입니다. 클라우드 애플리케이션이 디바이스별로 데이터를 분석하려는 경우 다운스트림 디바이스는 메시지에 추가 식별 정보를 포함해야 합니다. 또한 쌍 및 메서드와 같은 IoT Hub 기본 형식은 다운스트림 디바이스가 아닌 게이트웨이 디바이스에 대해서만 사용할 수 있습니다.

**ID 변환** - IoT Hub에 연결할 수 없는 디바이스는 대신 게이트웨이 디바이스에 연결할 수 있습니다. 게이트웨이는 다운스트림 디바이스를 대신하여 IoT Hub ID와 프로토콜 변환을 제공합니다. 게이트웨이는 스마트해서 다운스트림 디바이스에서 사용하는 프로토콜을 이해하고, ID를 제공하고, IoT Hub 기본 형식으로 변환할 수 있습니다. 다운스트림 디바이스는 IoT Hub에서 쌍 및 메서드를 포함한 고급 디바이스로 표시됩니다. 사용자는 IoT Hub의 디바이스와 상호 작용할 수 있고 중간 게이트웨이 디바이스를 인식하지 못합니다.

다음 리소스가 만들어집니다.

![랩 12 아키텍처](media/LAB_AK_12-architecture.png)

## <a name="in-this-lab"></a>랩 내용

이 랩에서는 다음 활동을 완료할 예정입니다.

* 랩 필수 구성 요소(필수 Azure 리소스) 구성
* Azure IoT Edge 사용 Linux VM을 IoT Edge 디바이스로 배포
* IoT Edge 디바이스 CA 인증서 생성 및 구성
* Azure Portal을 사용하여 IoT Hub에서 IoT Edge 디바이스 ID 만들기
* IoT Edge 게이트웨이 호스트 이름 설정
* IoT Edge 게이트웨이 디바이스를 IoT Hub에 연결
* 통신용 IoT Edge 게이트웨이 디바이스 포트 열기
* IoT Hub에서 다운스트림 디바이스 ID 만들기
* 다운스트림 디바이스를 IoT Edge 게이트웨이에 연결
* 이벤트 흐름 확인

## <a name="lab-instructions"></a>랩 지침

### <a name="exercise-1-configure-lab-prerequisites"></a>연습 1: 랩 필수 구성 요소 구성

이 랩은 다음 Azure 리소스를 사용할 수 있다고 가정합니다.

| 리소스 종류 | 리소스 이름 |
| :-- | :-- |
| 리소스 그룹 | rg-az220 |
| IoT Hub | iot-az220-training-{사용자 ID} |

이러한 리소스를 사용할 수 있게 하려면 다음 작업을 완료합니다.

1. 가상 머신 환경에서 Microsoft Edge 브라우저 창을 열고 다음 웹 주소로 이동합니다.
 
    +++https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoftLearning%2FAZ-220-Microsoft-Azure-IoT-Developer%2Fmaster%2FAllfiles%2FARM%2Flab12.json+++

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

1. 템플릿의 유효성을 검사하려면 **검토 및 만들기** 를 클릭합니다.

1. 유효성 검사를 통과하면 **만들기** 를 클릭합니다.

    배포가 시작됩니다.

1. 배포가 완료되면 왼쪽 탐색 영역에서 템플릿의 출력 값을 검토하려면 **출력** 을 클릭합니다.

    나중에 사용할 수 있도록 출력을 기록해 둡니다.

    * connectionString

이제 리소스가 만들어졌습니다.

### <a name="exercise-2-deploy-and-configure-a-linux-vm--as-an-iot-edge-gateway"></a>연습 2: Linux VM을 IoT Edge 게이트웨이로 배포하고 구성

이 연습에서는 Ubuntu Server VM을 배포하고 IoT Edge 게이트웨이로 구성합니다.

#### <a name="task-1-create-iot-edge-gateway-device-identity"></a>작업 1: IoT Edge 게이트웨이 디바이스 ID

이 작업에서는 Azure IoT Hub를 사용하여 IoT Edge Transparent Gateway(IoT Edge VM)에 사용할 새 IoT Edge 디바이스 ID를 만듭니다.

1. 필요한 경우 Azure 계정 자격 증명을 사용하여 Azure Portal에 로그인한 다음 Azure 대시보드로 이동합니다.

1. **rg-az220** 리소스 그룹 타일에서 IoT Hub를 열려면 **iot-az220-training-{사용자 ID}** 를 클릭합니다.

1. **iot-az220-training-{사용자 ID}** 블레이드 왼쪽 메뉴의 **디바이스 관리** 아래에서 **IoT Edge** 를 클릭합니다.

    IoT Edge 창을 사용하면 IoT Hub에 연결된 IoT Edge 디바이스를 관리할 수 있습니다.

1. 창 상단에서 **IoT Edge 디바이스 추가** 를 클릭합니다.

1. **디바이스 만들기** 블레이드의 **디바이스 ID** 필드에 **vm-az220-training-gw0001-{사용자 ID}** 를 입력합니다.

    {사용자 ID}는 과정 시작 부분에서 만든 값으로 바꿉니다. 인증 및 액세스 제어에 사용되는 디바이스 ID입니다.

1. **인증 형식** 에서 **대칭 키** 가 선택되어 있는지 확인하고 **키 자동 생성** 확인란을 선택된 상태로 둡니다.

    이렇게 하면 IoT Hub가 디바이스를 인증하기 위한 대칭 키를 자동으로 생성합니다.

1. 다른 설정을 기본값으로 두고 **저장** 을 클릭합니다.

    잠시 후 새로운 IoT Edge 디바이스가 IoT Edge 디바이스 목록에 추가됩니다.

1. **디바이스 ID** 에서 **vm-az220-training-gw0001-{사용자 ID}** 를 클릭합니다.

1. **vm-az220-training-gw0001-{사용자 ID}** 블레이드에서 **기본 연결 문자열** 을 복사합니다.

    복사 단추가 값 오른쪽에 표시됩니다.

1. **기본 연결 문자열** 의 값을 파일에 저장하여 연결된 디바이스에 대한 메모를 작성합니다.

1. **vm-az220-training-gw0001-{사용자 ID}** 블레이드에서 **모듈** 목록에는 **\$edgeAgent** 와 **\$edgeHub** 만 표시됩니다.

    IoT Edge 에이전트( **\$edgeAgent**) 및 IoT Edge 허브( **\$edgeHub**) 모듈은 IoT Edge 런타임의 일부입니다. Edge 허브는 통신을 담당하며 Edge 에이전트는 디바이스에서 모듈을 배포하고 모니터링합니다.

1. 블레이드 상단에서 **모듈 설정** 을 클릭합니다.

    **디바이스에서 모듈 설정** 블레이드를 사용하여 IoT Edge 디바이스에 추가 모듈을 추가할 수 있습니다. 지금은 이 블레이드를 사용하여 IoT Edge 게이트웨이 디바이스에 대해 메시지 라우팅이 올바르게 구성되었는지 확인합니다.

1. **디바이스에서 모듈 설정** 블레이드에서 **경로** 를 클릭합니다.

    **경로** 에서 편집기는 IoT Edge 디바이스에 대해 구성된 기본 경로를 표시합니다. 이때 모든 모듈의 모든 메시지를 Azure IoT Hub로 보내는 경로로 구성해야 합니다. 경로 구성이 이 경로와 일치하지 않으면 다음 경로와 일치하도록 업데이트합니다.

    * **이름**: `route`
    * **값**: `FROM /* INTO $upstream`

    메시지 경로의 `FROM /*` 부분은 모든 디바이스-클라우드 메시지 또는 모든 모듈 또는 리프 디바이스의 쌍 변경 알림과 일치합니다. 그런 다음 `INTO $upstream`은 해당 메시지를 Azure IoT Hub 전송하도록 경로에 지시합니다.

    > **참고**:  Azure IoT Edge 내에서 메시지 라우팅을 구성하는 방법에 대한 자세한 내용은 [IoT Edge에서 모듈을 배포하고 경로를 설정하는 방법 알아보기](https://docs.microsoft.com/azure/iot-edge/module-composition#declare-routes#declare-routes) 문서를 참조하세요.

1. 블레이드 아래쪽에서 **검토 + 만들기** 를 클릭합니다.

    **디바이스에서 모듈 설정** 블레이드의 이 탭에는 Edge 디바이스에 대한 배포 매니페스트가 표시됩니다. 블레이드 상단에 "유효성 검사 통과"를 나타내는 메시지가 표시됩니다.

1. 잠시 배포 매니페스트를 검토합니다.

1. 블레이드 하단에서 **만들기** 를 클릭합니다.

#### <a name="task-2-provision-iot-edge-vm"></a>작업 2: IoT Edge VM 프로비저닝

이 작업에서는 ARM(Azure Resource Manager) 템플릿을 사용하여 Linux VM을 프로비저닝하고, IoT Edge 런타임을 설치하고, IoT Hub 연결을 구성하고, 디바이스와 게이트웨이의 통신을 암호화하기 위한 X509 인증서를 생성하여 IoT Edge 런타임 구성에 추가합니다.

> **정보**: 자동화된 단계에 대한 자세한 내용은 다음 리소스를 참조하세요.
>
> * [Linux용 Azure IoT Edge 설치 또는 제거](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge?view=iotedge-2020-11)
> * [IoT Edge 디바이스에서 인증서 관리](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-manage-device-certificates?view=iotedge-2020-11)
> * [데모 인증서를 만들어 IoT Edge 디바이스 기능 테스트](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-create-test-certificates?view=iotedge-2020-11)

1. 웹 브라우저를 열고 다음 주소로 이동합니다. 

    ```
    https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoftLearning%2FAZ-220-Microsoft-Azure-IoT-Developer%2Fmaster%2FAllfiles%2FARM%2Flab12a.json
    ```

1. 메시지가 표시되면 이 랩에서 사용 중인 Azure 자격 증명을 사용하여 로그인합니다.

    **사용자 지정 배포** 페이지가 표시됩니다.

1. **프로젝트 세부 정보** 의 **구독** 드롭다운에서 이 과정에서 사용할 Azure 구독이 선택되어 있는지 확인합니다.

1. **리소스 그룹** 드롭다운에서 **rg-az220vm** 을 선택, 생성 및 입력합니다.

1. **지역** 필드에 이전에 사용한 것과 동일한 위치를 입력합니다.

1. **가상 머신 이름** 텍스트 상자에 **vm-az220-training-gw0001-{사용자 ID}** 를 입력합니다.

1. **디바이스 연결 문자열** 필드에 이전 연습의 연결 문자열 값을 입력합니다.

1. **가상 머신 크기** 필드에서 **Standard_DS1_v2** 가 입력되었는지 확인합니다.

1. **Ubuntu OS 버전** 필드에서 **18.04-LTS** 가 입력되었는지 확인합니다.

1. **관리자 사용자 이름** 필드에 사용자 이름을 입력합니다.

1. **인증 유형** 필드에서 **암호** 가 선택되어 있는지 확인합니다.

1. **관리자 암호 또는 키** 필드에 사용할 암호를 입력합니다.

1. **SSH 허용** 필드에서 **true** 가 선택되어 있는지 확인합니다.

1. 템플릿의 유효성을 검사하려면 **검토 및 만들기** 를 클릭합니다.

1. 유효성 검사를 통과하면 **만들기** 를 클릭합니다.

    > **참고**:  배포가 신속하게 완료되더라도 VM의 구성은 백그라운드에서 계속됩니다.

1. 템플릿이 완료되면 **출력** 창으로 이동하여 다음을 기록해 둡니다.

    * 공용 FQDN
    * 공용 SSH

####
 작업 3: 통신용 IoT Edge 게이트웨이 디바이스 포트 열기

IoT Hub와의 모든 통신은 아웃바운드 연결을 통해 수행되므로 표준 IoT Edge 디바이스는 작동하기 위해 인바운드 연결이 필요하지 않습니다. 게이트웨이 디바이스는 다운스트림 디바이스에서 메시지를 수신해야 하기 때문에 다릅니다. 다운스트림 디바이스와 게이트웨이 디바이스 사이에 방화벽이 있는 경우 방화벽을 통해서도 통신이 가능해야 합니다. Azure IoT Edge 게이트웨이가 작동하려면 다운스트림 디바이스의 인바운드 트래픽에 대해 IoT Edge 허브의 지원되는 프로토콜 중 하나 이상을 열어야 합니다. 지원되는 프로토콜은 MQTT, AMQP 및 HTTPS입니다.

Azure IoT Edge에서 지원하는 IoT 통신 프로토콜은 다음과 같은 포트 매핑을 갖습니다.

| 프로토콜 | 포트 번호 |
| --- | --- |
| MQTT | 8883 |
| AMQP | 5671 |
| HTTPS<br/>MQTT + WS (Websocket)<br/>AMQP + WS (Websocket) | 443 |

디바이스에 대해 선택한 IoT 통신 프로토콜에는 IoT Edge 게이트웨이 디바이스를 보호하는 방화벽에 대해 열린 해당 포트가 있어야 합니다. 이 랩의 경우 Azure NSG(네트워크 보안 그룹)이 IoT Edge 게이트웨이를 보호하는 데 사용되므로 이러한 포트에서 NSG에 대한 인바운드 보안 규칙이 열립니다.

프로덕션 시나리오에서는 디바이스가 통신할 수 있는 최소 포트 수만 열고 싶을 수 있습니다. MQTT를 사용하는 경우 인바운드 통신에 대해서 포트 8883만을 엽니다. 추가 포트를 열면 공격자가 악용할 수 있는 추가 보안 공격 벡터가 도입됩니다. 솔루션에 필요한 최소한의 포트만 여는 것이 보안 모범 사례입니다.

이 작업에서는 인터넷에서 Azure IoT Edge 게이트웨이 액세스를 보호하는 NSG(네트워크 보안 그룹)를 구성합니다. 다운스트림 IoT 디바이스가 게이트웨이와 통신할 수 있도록 MQTT, AMQP 및 HTTPS 통신에 필요한 포트를 열어야 합니다.

1. 필요한 경우 Azure 계정 자격 증명을 사용하여 Azure Portal에 로그인합니다.

1. Azure 대시보드에서 **rg-az220vm** 리소스 그룹 타일을 찾습니다.

    리소스 그룹 타일에는 연결된 네트워크 보안 그룹으로 이동할 수 있는 링크가 포함되어 있습니다.

1. **rg-az220vm** 리소스 그룹 타일에서 **nsg-vm-az220-training-gw0001-{사용자 ID}** 를 클릭합니다.

1. **네트워크 보안 그룹** 블레이드 왼쪽 메뉴의 **설정** 에서 **인바운드 보안 규칙** 을 클릭합니다.

1. **인바운드 보안 규칙** 창 상단에서 **추가** 를 클릭합니다.

1. **인바운드 보안 규칙 추가** 창에서 **원본** 이 **모두** 로 설정되어 있는지 확인합니다.

    이렇게 하면 모든 원본의 트래픽이 허용됩니다. 프로덕션 환경에서는 이를 특정 주소 등으로 제한할 수 있습니다.

1. **대상** 아래에서 **대상** 이 **모두** 로 설정되어 있는지 확인합니다.

    이렇게 하면 나가는 트래픽을 모든 위치로 라우팅할 수 있습니다. 프로덕션 환경에서는 주소를 제한할 수 있습니다.

1. **대상 포트 범위** 아래에서 값을 **8883** 로 변경합니다.

    MQTT 프로토콜의 포트입니다.

1. **프로토콜** 에서 **TCP** 를 클릭합니다.

    MQTT는 TCP를 사용합니다.

1. **작업** 에서 **허용** 이 선택되어 있는지 확인합니다.

    이 규칙은 나가는 트래픽을 허용하기 위한 것이므로 **허용** 이 선택됩니다.

1. **우선 순위** 에서 기본값이 제공됩니다. 기본값은 대부분의 경우 **1010** 이며 **반드시** 고유해야 합니다.

    규칙은 우선 순위대로 처리됩니다. 숫자가 작을수록 우선 순위가 높습니다. 규칙(100, 200, 300 등) 간 간격을 두는 것이 좋습니다. 이렇게 하여 기존 규칙을 편집하지 않고도 새 규칙을 추가하기가 더 쉽습니다.

1. **이름** 에서 값을 **MQTT** 로 변경합니다.

1. 다른 설정을 기본값으로 둔 다음 **추가** 를 클릭합니다.

    이렇게 하면 MQTT 프로토콜을 IoT Edge 게이트웨이로 통신할 수 있는 인바운드 보안 규칙이 정의됩니다.

1. MQTT 규칙이 추가된 후 **AMQP** 및 **HTTPS** 통신 프로토콜에 대한 포트를 열려면 다음 값으로 두 개의 규칙을 추가합니다.

    | 대상 포트 범위 | 프로토콜 | 이름 |
    | :--- | :--- | :--- |
    | 5671 | TCP | AMQP |
    | 443 | TCP | HTTPS |

   > **참고**: 창 상단의 도구 모음에서 **새로 고침** 단추를 사용하여 새 규칙이 표시되는지 확인해야 할 수 있습니다.

1. 이러한 세 개의 포트가 NSG(네트워크 보안 그룹)에서 열리면 MQTT, AMQP 또는 HTTPS 프로토콜을 사용하여 다운스트림 디바이스를 IoT Edge 게이트웨이에 연결할 수 있습니다.

### <a name="exercise-3-download-device-ca-certificate"></a>연습 3: 디바이스 CA 인증서 다운로드

이 연습에서는 방금 만든 **vm-az220-training-gw0001-{사용자 ID}** Virtual Machine을 살펴보고 생성된 테스트 인증서를 클라우드 셸에 다운로드합니다.

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

1. Cloud Shell 명령 프롬프트에서 이전 작업에서 적어 둔 **ssh** 명령을 붙여넣어 예를 들어 **ssh vmadmin@vm-az220-training-edge0001-dm080321.centralus.cloudapp.azure.com** 과 같이 만든 다음 **Enter** 키를 누릅니다.

1. **계속 연결하시겠습니까?** 라는 메시지가 나타나면 **네** 라고 입력하고 **Enter 키** 를 누르세요.

    해당 메시지는 VM에 대한 연결 보안에 사용된 인증서가 자체 서명된 인증서이므로 보안 확인입니다. 이 메시지에 대한 대답은 이후 연결을 위해 기억되며, 이 메시지는 첫 번째 연결에서만 표시됩니다.

1. 암호를 입력하라는 메시지가 표시되면 Edge 게이트웨이 VM이 프로비전될 때 만든 관리자 암호를 입력합니다.

1. 연결되면 터미널은 다음과 같이 Linux VM의 이름을 표시하도록 변경됩니다. 이는 어떤 VM에 연결되었는지 알려줍니다.

    ``` bash
    username@vm-az220-training-gw0001-{your-id}:~$
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

    > **참고**: 공용 IP가 Azure Cloud Shell에 표시되지 않는 경우 Azure Portal을 사용하여 VM의 공용 IP 주소를 찾습니다.

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

#### <a name="task-3-download-ssl-certs-from-vm-to-cloud-shell"></a>작업 3: VM에서 Cloud Shell로 SSL 인증서 다운로드

다음으로, 리프 디바이스와 IoT Edge 게이트웨이 간의 통신을 암호화하는 데 사용할 수 있도록 **vm-az220-training-gw0001-{사용자 ID}** 가상 머신에서 **MyEdgeDeviceCA** 인증서를 “다운로드”해야 합니다.

1. **vm-az220-training-gw0001-{사용자 ID}** 가상 머신에서 **Cloud Shell** 스토리지로 **/tmp/lab12** 디렉터리를 다운로드하려면 Cloud Shell 명령 프롬프트에서 다음 명령을 입력합니다.

    ```bash
    mkdir lab12
    scp -r -p {username}@{FQDN}:/tmp/lab12 .
    ```

    >**중요**: 위의 명령에는 `space` 및 끝에 마침표(`.`)가 포함됩니다.

    > **참고**: **{username}** 자리 표시자를 VM에 대한 관리 사용자의 사용자 이름으로 바꾸고 **{FQDN}** 자리 표시자를 VM의 정규화된 도메인 이름으로 바꿉니다. 필요한 경우 SSH 세션을 여는 데 사용한 명령을 참조하세요.
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

    **vm-az220-training-gw0001-{사용자 ID}** 가상 머신에서 파일을 Cloud Shell 스토리지로 복사하면 필요에 따라 IoT Edge 디바이스 인증서 및 키 파일을 로컬 컴퓨터에 쉽게 다운로드할 수 있습니다. `download <filename>` 명령을 사용하여 Cloud Shell에서 파일을 다운로드할 수 있습니다. 이 파일은 랩 뒷부분에서 다운로드합니다.

### <a name="exercise-4-create-a-downstream-device"></a>연습 4: 다운스트림 디바이스 만들기

이 연습에서는 다운스트림 디바이스가 만들어지고 게이트웨이를 통해 IoT Hub에 연결됩니다.

#### <a name="task-1-create-device-identity-in-iot-hub"></a>작업 1: IoT Hub에서 디바이스 ID 만들기

이 작업에서는 다운스트림 IoT 디바이스용 Azure IoT Hub에서 새 IoT 디바이스 ID를 만듭니다. 해당 디바이스 ID는 Azure IoT Edge 게이트웨이가 이 다운스트림 디바이스의 부모 디바이스가 되도록 구성됩니다.

1. 필요한 경우 Azure 계정 자격 증명을 사용하여 Azure Portal에 로그인합니다.

1. IoT Hub를 열려면 Azure 대시보드에서 **iot-az220-training-{사용자 ID}** 를 클릭합니다.

1. **iot-az220-training-{사용자 ID}** 블레이드 왼쪽 메뉴의 **디바이스 관리** 아래에서 **디바이스** 를 클릭합니다.

    이 IoT Hub 블레이드 창을 사용하면 IoT Hub에 연결된 IoT 디바이스를 관리할 수 있습니다.

1. 창 상단에서 새 IoT 디바이스 구성을 시작하려면 **+ 디바이스 추가** 를 클릭합니다.

1. **디바이스 만들기** 블레이드에서 **디바이스 ID** 아래에 **sensor-th-0072** 를 입력합니다.

    인증 및 액세스 제어에 사용되는 디바이스 ID입니다.

1. **인증 유형** 에 **대칭 키** 가 선택되어 있는지 확인합니다.

1. **키 자동 생성** 에서 확인란을 선택된 상태로 둡니다.

    이렇게 하면 IoT Hub가 디바이스를 인증하기 위한 대칭 키를 자동으로 생성합니다.

1. **부모 디바이스** 에서 **부모 디바이스 설정** 을 클릭합니다.

    이 랩의 앞에서 만든 IoT Edge 게이트웨이 디바이스를 통해 IoT Hub와 통신하도록 이 다운스트림 디바이스를 구성합니다.

1. **Edge 디바이스를 상위 디바이스로 설정** 블레이드의 **디바이스 ID** 에서 **vm-az220-training-gw0001-{사용자 ID}** 를 클릭한 다음 **확인** 을 클릭합니다.

1. **디바이스 만들기** 블레이드에서 다운스트림 디바이스에 대한 IoT 디바이스 ID를 만들려면 **저장** 을 클릭합니다.

1. **IoT 디바이스** 창에서 창 상단의 **새로 고침** 을 클릭합니다.

1. **디바이스 ID** 에서 **sensor-th-0072** 를 클릭합니다.

    그러면 이 디바이스에 대한 세부 정보 보기가 열립니다.

1. IoT 디바이스 요약 창에서 **기본 연결 문자열** 필드의 오른쪽에 있는 **복사** 를 클릭합니다.

1. 나중에 참조할 수 있도록 연결 문자열을 저장합니다.

    이 연결 문자열은 sensor-th-0072 하위 디바이스용입니다.

#### <a name="task-2-download-device-x509-xertificate"></a>작업 2: 디바이스 x509 인증서 다운로드

이 작업에서는 미리 빌드된 다운스트림 디바이스와 Edge 게이트웨이 디바이스 간의 연결을 구성합니다.

1. 필요한 경우 Azure 계정 자격 증명을 사용하여 Azure Portal에 로그인합니다.

    Azure 계정이 두 개 이상인 경우 이 과정에 사용할 구독에 연결된 계정으로 로그인해야 합니다.

1. Azure Portal 도구 모음에서 **Cloud Shell** 을 클릭합니다.

    환경이 **Bash** 로 설정되어 있는지 확인합니다.

    > **참고**: Cloud Shell이 이미 열려 있고 Edge 디바이스에 계속 연결되어 있는 경우 **exit** 명령을 사용하여 SSH 세션을 닫습니다.

1. IoT Edge 게이트웨이 가상 머신에 대한 루트 CA X.509 인증서를 다운로드하려면 Cloud Shell 명령 프롬프트에서 다음 명령을 입력합니다.

    ```bash
    download lab12/certs/azure-iot-test-only.root.ca.cert.pem
    ```

    Azure IoT Edge 게이트웨이는 게이트웨이에 연결된 다운스트림 디바이스와의 통신을 암호화하기 위해 이 루트 CA X.509 인증서를 사용하도록 이전에 **/etc/aziot/config.toml** 파일에서 구성되었습니다. 이 X.509 인증서는 게이트웨이와의 통신을 암호화하는 데 사용할 수 있도록 다운스트림 디바이스에 복사해야 합니다.

1. **azure-iot-test-only.root.ca.cert.pem** X.509 인증서 파일을 다운스트림 IoT 디바이스의 소스 코드가 있는 **/Starter/DownstreamDevice** 디렉터리에 복사합니다.

    > **중요**: 파일에 정확한 이름이 있는지 확인합니다. 이전 랩과 이름이 다를 수 있으므로(예: 파일 이름에 **(1)** 이 추가됨) 필요한 경우 파일을 복사한 후에 이름을 바꿉니다.

#### <a name="task-3-create-hosts-file-entry"></a>작업 3: 호스트 파일 항목 만들기

이 랩의 이전 버전에서는 FQDN이 디바이스 연결 문자열의 **GatewayHostName** 값으로 사용되지만 현재 버전의 테스트 스크립트에서 생성된 테스트 x509 인증서는 더 이상 이를 지원하지 않습니다. 대신, 호스트 이름만 사용되고 로컬 컴퓨터의 **호스트** 파일에 항목을 만들어서 호스트 이름을 IP 주소로 확인해야 합니다. 다음 단계를 완료하여 호스트 파일에 필요한 항목을 추가합니다.

1. Visual Studio Code를 엽니다.

1. **파일** 메뉴에서 **파일 열기** 를 클릭합니다.

1. 다음 폴더 **c:\\Windows\\System32\\Drivers\\etc\\** 파일로 이동하고 **호스트** 파일을 엽니다.

    > **참고**: **호스트** 파일에 확장명이 없습니다.

1. **호스트** 파일에 다음 줄을 추가하고 그 뒤에 빈 줄을 추가합니다.

    ```text
    {VM Public IP Address} vm-az220-training-gw0001-{your-id}
    {blank line}
    ```

    예를 들면 다음과 같습니다.

    ```text
    168.61.181.131 vm-az220-training-gw0001-dm090821

    ```

1. 파일 저장 - 저장하지 못했다는 메시지가 표시되면 **관리자로 다시 시도...** 를 클릭하고 **사용자 계정 컨트롤** 대화 상자에서 **예** 를 클릭합니다.

이제 로컬 컴퓨터에서 VM 이름을 적절한 IP 주소로 확인할 수 있습니다.

#### <a name="task-4-connect-downstream-device-to-iot-edge-gateway"></a>작업 4: 다운스트림 디바이스를 IoT Edge 게이트웨이에 연결

1. Visual Studio Code를 엽니다.

1. **파일** 메뉴에서 **폴더 열기** 를 클릭합니다.

1. **폴더 열기** 대화 상자에서 랩 12의 **Starter** 폴더로 이동하고 **DownstreamDevice** 를 클릭한 다음 **폴더 선택** 을 클릭합니다.

    azure-iot-test-only.root.ca.cert.pem 파일이 Program.cs 파일과 함께 탐색기 창에 나열됩니다.

    > **참고**: dotnet 복원 및/또는 C# 확장 로드 관련 메시지가 표시되는 경우 설치를 완료하면 됩니다.

1. 탐색기 창에서 **Program.cs** 를 클릭합니다.

    이 앱을 검토해 보면 이전 랩의 작업에서 사용했던 **CaveDevice** 애플리케이션의 변형임을 확인할 수 있습니다.

1. **connectionString** 변수의 선언을 찾은 다음 자리 표시자 값을 **sensor-th-0072** IoT 디바이스의 기본 연결 문자열로 바꿉니다.

1. 할당된 **connectionString** 값에 **GatewayHostName** 속성을 추가하고 GatewayHostName의 값을 IoT Edge 게이트웨이 디바이스의 이름으로 설정합니다. 이 이름은 이 랩의 앞부분에서 호스트 파일에 제공된 이름과 일치해야 합니다.

    완성된 연결 문자열 값은 다음 형식과 일치해야 합니다.

    ```text
    HostName=<IoT-Hub-Name>.azure-devices.net;DeviceId=sensor-th-0072;SharedAccessKey=<Primary-Key-for-IoT-Device>;GatewayHostName=<HostName-for-IoT-Edge-Device>
    ```

    > **중요**: 이전 버전의 IoTEdge 런타임에서 **GatewayHostName** 은 전체 DNS 이름입니다.

    위에 표시된 자리 표시자를 적절한 값으로 바꾸세요.

    * **\<IoT-Hub-Name\>** : Azure IoT Hub 이름입니다.
    * **\<Primary-Key-for-IoT-Device\>** : Azure IoT Hub의 **sensor-th-0072** IoT 디바이스용 기본 키입니다.
    * **\<Hostname-Name-for-IoT-Edge-Device\>** : **vm-az220-training-gw0001-{사용자 ID}** Edge 디바이스의 호스트 이름입니다.

    연결 문자열 값이 조합된 **connectionString** 변수는 다음과 같습니다.

    ```csharp
    private readonly static string connectionString = "HostName=iot-az220-training-abc201119.azure-devices.net;DeviceId=sensor-th-0072;SharedAccessKey=ygNT/WqWs2d8AbVD9NAlxcoSS2rr628fI7YLPzmBdgE=;GatewayHostName=vm-az220-training-gw0001-{your-id}";
    ```

1. **파일** 메뉴에서 **저장** 을 클릭합니다.

1. 아래로 스크롤하여 **Main** 메서드를 찾은 다음 잠시 코드를 검토합니다.

    이 메서드에는 구성된 연결 문자열을 사용하여 **DeviceClient** 를 인스턴스화하고 `MQTT`를 Azure IoT Edge 게이트웨이와 통신하는 데 사용할 전송 프로토콜로 지정하는 코드가 포함되어 있습니다.

    ```csharp
    deviceClient = DeviceClient.CreateFromConnectionString(connectionString, TransportType.Mqtt);
    SendDeviceToCloudMessagesAsync();
    ```

    Main 메서드는 다음 작업도 수행합니다.

    * **InstallCACert** 메서드를 호출합니다. 이 메서드에는 로컬 컴퓨터에 루트 CA X.509 인증서를 자동 설치하는 코드가 포함되어 있습니다.
    * 시뮬레이션된 디바이스에서 이벤트 원격 분석을 전송하는 **SendDeviceToCloudMessagesAsync** 메서드를 호출합니다.

1. **SendDeviceToCloudMessagesAsync** 메서드를 찾은 다음 잠시 시간을 내어 코드를 검토합니다.

    이 메서드는 시뮬레이션된 디바이스 원격 분석을 생성하는 코드를 포함하며 IoT Edge 게이트웨이로 이벤트를 보냅니다.

1. **InstallCACert** 를 찾아 루트 CA X.509 인증서를 로컬 컴퓨터 인증서 저장소에 설치하는 코드를 찾습니다.

    > **참고**: 이 인증서는 디바이스에서 Edge 게이트웨이로의 통신을 보호하는 데 사용됩니다. 디바이스는 연결 문자열 내의 대칭 키를 사용하여 IoT Hub에 인증합니다.

    이 메서드 내의 초기 코드는 **azure-iot-test-only.root.ca.cert.pem** 파일을 사용할 수 있는지 확인합니다. 물론 프로덕션 애플리케이션에서는 환경 변수나 TPM 등의 대체 메커니즘을 사용해 X.509 인증서 경로를 지정할 수 있습니다.

    X.509 인증서가 있음이 확인되면 **X509Store** 클래스를 사용하여 현재 사용자의 인증서 저장소로 인증서를 로드합니다. 그러면 요청 시 인증서를 사용하여 게이트웨이로의 통신을 보호할 수 있습니다. 이 과정은 디바이스 클라이언트 내에서 자동으로 진행되므로 추가 코드는 없습니다.

    > **정보**: [여기](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.x509certificates.x509store?view=netcore-3.1)서 **X509Store** 클래스에 대해 자세히 알아볼 수 있습니다.

1. **터미널** 메뉴에서 **새 터미널** 을 클릭합니다.

1. 터미널 명령 프롬프트에서 다음 명령을 입력합니다.

    ```bash
    dotnet run
    ```

    이 명령은 **sensor-th-0072** 시뮬레이션된 디바이스용 코드를 작성하고 실행합니다. 그러면 디바이스 원격 분석 전송이 시작됩니다.

    > **참고**: (IoT Edge 게이트웨이로 인증하는 데 사용하기 위해) 앱이 로컬 컴퓨터에 X.509 인증서를 설치하려고 하면 인증서 설치에 대한 보안 경고가 표시될 수 있습니다. 앱을 계속 사용하려면 **예** 를 클릭해야 합니다.

1. 인증서를 설치할지 묻는 메시지가 표시되면 **예** 를 클릭합니다.

1. 시뮬레이션된 디바이스가 실행되면 콘솔 출력에 Azure IoT Edge 게이트웨이로 전송되는 이벤트가 표시됩니다.

    터미널 출력은 다음과 유사합니다.

    ```text
    IoT Hub C# Simulated Cave Device. Ctrl-C to exit.

    User configured CA certificate path: azure-iot-test-only.root.ca.cert.pem
    Attempting to install CA certificate: azure-iot-test-only.root.ca.cert.pem
    Successfully added certificate: azure-iot-test-only.root.ca.cert.pem

    10/25/2019 6:10:12 PM > Sending message: {"temperature":27.714212817472504,"humidity":63.88147743599558}
    10/25/2019 6:10:13 PM > Sending message: {"temperature":20.017463779085066,"humidity":64.53511070671263}
    10/25/2019 6:10:14 PM > Sending message: {"temperature":20.723927165718717,"humidity":74.07808918230147}
    10/25/2019 6:10:15 PM > Sending message: {"temperature":20.48506045736608,"humidity":71.47250854944461}
    ```

    > **참고**: 디바이스 전송이 첫 번째 전송에서 1초 이상 일시 중지되는 것처럼 보이는 경우, 이전에 NSG 수신 규칙을 올바르게 추가하지 않았으므로 MQTT 트래픽이 차단된 것입니다.  NSG 구성을 확인합니다.

1. 다음 연습으로 이동하는 동안 시뮬레이션된 디바이스를 실행 상태로 둡니다.

#### <a name="task-5-verify-event-flow"></a>작업 5: 이벤트 흐름 확인

이 작업에서는 Azure CLI를 사용하여 IoT Edge 게이트웨이를 통해 다운스트림 IoT 디바이스에서 Azure IoT Hub로 전송되는 이벤트를 모니터링합니다. 이렇게 하면 모든 것이 올바르게 작동하는지 확인합니다.

1. 필요한 경우 Azure 계정 자격 증명을 사용하여 Azure Portal에 로그인합니다.

    Azure 계정이 두 개 이상인 경우 이 과정에 사용할 구독에 연결된 계정으로 로그인해야 합니다.

1. Cloud Shell이 실행되지 않는 경우 Azure Portal 도구 모음에서 **Cloud Shell** 을 클릭합니다.

1. Cloud Shell의 SSH 연결을 사용하여 Edge 디바이스에 계속 연결되어 있으면 해당 연결을 종료합니다.

1. Azure IoT Hub로 이동하는 이벤트 스트림을 모니터링하려면 Cloud Shell 명령 프롬프트에서 다음 명령을 실행합니다.

    ```bash
    az iot hub monitor-events -n iot-az220-training-{your-id}
    ```

    `-n` 매개 변수에 대한 `{your-id}` 자리 표시자를 Azure IoT Hub 이름으로 바꾸어야 합니다.

    `az iot hub monitor-events` 명령을 사용하여 Azure IoT Hub으로 전송되는 디바이스 원격 분석 및 메시지를 모니터링할 수 있습니다. 이렇게 하면 IoT Edge 게이트웨이로 전송되는 시뮬레이션된 디바이스의 이벤트가 Azure IoT Hub에서 수신되는지 확인합니다.

    > **참고**: `Dependency update (uamqp 1.2) required for IoT extension version: 0.10.13.` 메시지가 표시되면 **Y* 를 입력합니다.

1. 모든 것이 제대로 작동하는 경우 `az iot hub monitor-events` 명령의 출력은 다음과 유사하게 표시됩니다.

    ```text
    chris@Azure:~$ az iot hub monitor-events -n iot-az220-training-1119
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

이 랩을 완료하고 이벤트 흐름을 확인한 후에는 **CTRL+C** 를 눌러 콘솔 애플리케이션을 종료합니다.
