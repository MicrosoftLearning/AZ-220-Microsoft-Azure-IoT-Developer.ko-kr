---
lab:
  title: '랩 06: DPS를 통해 IoT 디바이스를 안전하게 대규모로 자동 프로비전'
  module: 'Module 3: Device Provisioning at Scale'
ms.openlocfilehash: e8e5d66ff173285ebc3c7c40a7815059d899a543
ms.sourcegitcommit: eec2943250f1cd1ad2c5202ecbb9c37af71e8961
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/24/2022
ms.locfileid: "140872819"
---
# <a name="automatically-provision-iot-devices-securely-and-at-scale-with-dps"></a>DPS를 통해 IoT 디바이스를 안전하게 대규모로 자동 프로비전

## <a name="lab-scenario"></a>랩 시나리오

Contoso의 자산 모니터링 및 추적 솔루션에 대한 최신 작업을 통해 개별 등록 방식을 사용하여 디바이스 프로비저닝 및 프로비전 해제 프로세스의 유효성을 검사할 수 있게 되었습니다. 이제 관리 팀에서 대규모 롤아웃을 위해 프로세스 테스트를 시작할 것을 요청했습니다.

프로젝트를 계속 진행하려면 Device Provisioning Service를 통해 X.509 인증서 인증을 사용하여 더 많은 수의 디바이스를 자동으로 안전하게 등록할 수 있음을 입증해야 합니다. 이 랩에서는 그룹 등록을 설정하여 Contoso의 요구 사항이 충족되는지를 확인합니다.

다음 리소스가 만들어집니다.

![랩 6 아키텍처](media/LAB_AK_06-architecture.png)

## <a name="in-this-lab"></a>랩 내용

이 랩에서는 먼저 랩 필수 구성 요소를 검토한 후 필요에 따라 스크립트를 실행하여 Azure 구독에 필요한 리소스가 포함되어 있는지를 확인합니다. 그런 다음 Azure Cloud Shell 내에서 OpenSSL을 사용하여 X.509 루트 CA 인증서를 생성하고, 해당 루트 인증서를 사용하여 DPS(Device Provisioning Service) 내에서 그룹 등록을 구성합니다. 그 후에는 루트 인증서를 사용해 디바이스 인증서를 생성합니다. 시뮬레이션된 디바이스 코드 내에서 이 인증서를 사용하여 디바이스를 IoT Hub에 프로비전할 예정입니다. 그리고 디바이스 코드 내에서 디바이스 초기 구성을 수행하는 데 사용되는 디바이스 트윈 속성 액세스를 구현합니다. 그 후에는 시뮬레이션된 디바이스를 테스트합니다. 그리고 전체 그룹 등록의 프로비전을 해제하는 것으로 랩을 마무리합니다. 랩에 포함된 연습은 다음과 같습니다.

* 랩 필수 구성 요소(필수 Azure 리소스) 구성
* OpenSSL을 사용하여 X.509 CA 인증서 생성 및 구성
* X.509 인증서로 시뮬레이션된 디바이스 구성
* 시뮬레이션된 디바이스 테스트
* 그룹 등록 프로비전 해제

## <a name="lab-instructions"></a>랩 지침

### <a name="exercise-1-configure-lab-prerequisites"></a>연습 1: 랩 필수 구성 요소 구성

이 랩은 다음 Azure 리소스를 사용할 수 있다고 가정합니다.

| 리소스 종류 | 리소스 이름 |
| :-- | :-- |
| 리소스 그룹 | rg-az220 |
| IoT Hub | iot-az220-training-{사용자 ID} |
| Device Provisioning Service | dps-az220-training-{사용자 ID} |

이러한 리소스를 사용할 수 있게 하려면 다음 단계를 완료합니다.

1. 가상 머신 환경에서 Microsoft Edge 브라우저 창을 열고 다음 웹 주소로 이동합니다.
 
    +++https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoftLearning%2FAZ-220-Microsoft-Azure-IoT-Developer%2Fmaster%2FAllfiles%2FARM%2Flab06.json+++

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
    * dpsScopeId

이제 리소스가 만들어졌습니다.

### <a name="exercise-2-generate-and-configure-x509-ca-certificates-using-openssl"></a>연습 2: OpenSSL을 사용하여 X.509 CA 인증서 생성 및 구성

이 연습에서는 Azure Cloud Shell 내에서 OpenSSL을 사용하여 X.509 CA 인증서를 생성합니다. 이 인증서는 DPS(Device Provisioning Service) 내에서 그룹 등록계약을 구성하는 데 사용합니다.

#### <a name="task-1-generate-the-certificates"></a>작업 1: 인증서 생성

1. 필요한 경우 이 과정에서 사용 중인 Azure 계정 자격 증명을 사용하여 [Azure Portal](https://portal.azure.com)에 로그인합니다.

    Azure 계정이 두 개 이상인 경우 이 과정에 사용할 구독에 연결된 계정으로 로그인해야 합니다.

1. Azure Cloud Shell을 열려면 포털 창의 오른쪽 상단에서 **Cloud Shell** 을 클릭합니다.

    Cloud Shell 단추에는 명령 프롬프트를 나타내는 **`>_`** 아이콘이 있습니다.

    Cloud Shell 창이 디스플레이 화면 하단 근처에서 열립니다.

1. Cloud Shell 창의 왼쪽 상단에서 **Bash** 가 환경 옵션으로 선택되어 있는지 확인합니다.

    > **참고**:  Azure Cloud Shell에 대한 *Bash* 및 *PowerShell* 인터페이스는 **OpenSSL** 의 사용을 지원합니다. 이 연습에서는 *Bash* 셸용으로 작성된 일부 도우미 스크립트를 사용합니다.

1. 새 디렉터리를 만들고 해당 디렉터리로 이동하려면 Cloud Shell 명령 프롬프트에서 다음 명령을 입력합니다.

    ```sh
    # ensure the current directory is the user's home directory
    cd ~

    # make a directory named "certificates"
    mkdir certificates

    # change directory to the "certificates" directory
    cd certificates
    ```

1. 사용하려는 Azure IoT 도우미 스크립트를 다운로드하고 준비하려면 Cloud Shell 명령 프롬프트에서 다음 명령을 입력합니다.

    ```sh
    # download helper script files
    curl https://raw.githubusercontent.com/Azure/azure-iot-sdk-c/master/tools/CACertificates/certGen.sh --output certGen.sh
    curl https://raw.githubusercontent.com/Azure/azure-iot-sdk-c/master/tools/CACertificates/openssl_device_intermediate_ca.cnf --output openssl_device_intermediate_ca.cnf
    curl https://raw.githubusercontent.com/Azure/azure-iot-sdk-c/master/tools/CACertificates/openssl_root_ca.cnf --output openssl_root_ca.cnf

    # update script permissions so user can read, write, and execute it
    chmod 700 certGen.sh
    ```

    위의 코드에서는 Github에서 호스트되는 **Azure/azure-iot-sdk-c** 오픈 소스 스크립트에서 도우미 스크립트 및 지원 파일을 다운로드합니다. 이 프로젝트는 Azure IoT 디바이스 SDK의 한 구성 요소입니다. **certGen.sh** 도우미 스크립트를 사용하면 구체적인 OpenSSL 구성(이 과정에서는 설명하지 않음)을 자세히 확인하지 않고도 CA 인증서 사용 방식을 확인할 수 있습니다.

    이 도우미 스크립트를 사용하기 위한 추가 지침이나 Bash 대신 PowerShell을 사용하는 방법에 대한 지침이 필요한 경우 [https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md)를 참조하세요.

    > **경고**: 이 도우미 스크립트로 만든 인증서는 프로덕션에 사용할 수 **없습니다**. 도우미 스크립트에 하드 코드되어 있는 암호(“*1234*”)는 30일 후에 만료되며, CA 인증서 사용법을 빠르게 파악할 수 있도록 데모용으로만 제공됩니다. CA 인증서를 사용하도록 제품을 제작할 때는 인증서 만들기 및 수명 주기 관리와 관련된 회사의 보안 모범 사례를 적용해야 합니다.

    관심이 있다면 Cloud Shell에 내장된 편집기를 사용하여 다운로드한 스크립트 파일의 콘텐츠를 빠르게 검사할 수 있습니다.

    * Cloud Shell에서 편집기를 열려면 **`{}`** 를 클릭합니다.
    * 파일 목록에서 **인증서** 를 클릭한 다음 **certGen.sh** 를 클릭합니다

    > **참고**: Bash 환경에서 `more` 또는 `vi` 명령과 같은 다른 텍스트 파일 보기 도구가 있는 경우 해당 도구를 사용할 수도 있습니다.

    다음으로는 스크립트를 사용하여 루트 및 중간 인증서를 만듭니다.

1. 루트 및 중간 인증서를 만들려면 다음 명령을 입력합니다.

    ```sh
    ./certGen.sh create_root_and_intermediate
    ```

    `create_root_and_intermediate` 옵션을 사용하여 스크립트를 실행했습니다. 이 명령은 `~/certificates` 디렉터리 내에서 스크립트를 실행하는 것으로 가정합니다.

    이 명령은 `azure-iot-test-only.root.ca.cert.pem`이라는 루트 CA 인증서를 생성하고 `./certs` 디렉터리(만든 인증서 디렉터리 아래)에 배치했습니다.

1. 루트 인증서를 (DPS에 업로드할 수 있도록) 로컬 컴퓨터에 다운로드하려면 다음 명령을 입력합니다

    ```sh
    download ~/certificates/certs/azure-iot-test-only.root.ca.cert.pem
    ```

    파일을 로컬 컴퓨터에 저장하라는 메시지가 표시됩니다. 다음 작업에 파일이 필요하므로 파일이 저장되는 위치를 기록합니다.

#### <a name="task-2-configure-dps-to-trust-the-root-certificate"></a>작업 2: 루트 인증서를 신뢰하도록 DPS 구성

1. Azure Portal에서 Device Provisioning Service를 엽니다.

    대시보드의 리소스 타일에서 `dps-az220-training-{your-id}`를 클릭하면 Device Provisioning Service에 액세스할 수 있습니다.

1. **dps-az220-training-{사용자 ID}** 블레이드의 왼쪽 메뉴에서 **설정** 아래의 **인증서** 를 클릭합니다.

1. **인증서** 창 위쪽에서 **+ 추가** 를 클릭합니다.

    **+ 추가** 를 클릭하면 X.509 CA 인증서를 DPS 서비스에 업로드하는 프로세스가 시작됩니다.

1. **인증서 추가** 블레이드의 **인증서 이름** 아래에 **root-ca-cert** 를 입력합니다.

    루트 인증서 및 중간 인증서와 같은 인증서 또는 체인 내의 같은 계층 구조 수준에서 여러 인증서를 구분할 수 있는 이름을 하는 것이 중요합니다.

    > **참고**: 입력한 루트 인증서 이름은 인증서 파일의 이름과 같거나 다를 수 있습니다. 제공된 이름은 X.509 CA 인증서 내용에 포함된 _일반 이름_ 과 상관없는 논리적 이름입니다.

1. **인증서 .pem 또는 .cer 파일** 아래 텍스트 상자 오른쪽에서 **열기** 를 클릭합니다.

    텍스트 필드 오른쪽의 **열기** 단추를 클릭하면 이전에 다운로드한 `azure-iot-test-only.root.ca.cert.pem` CA 인증서로 이동할 수 있는 파일 열기 대화 상자가 열립니다.

1. 루트 CA 인증서 파일을 다운로드한 폴더 위치로 이동하여 **azure-iot-test-only.root.ca.cert.pem** 을 클릭하고 **열기** 를 클릭합니다.

1. **인증서 추가** 블레이드 아래쪽에서 **저장** 을 클릭합니다.

    X.509 CA 인증서가 업로드되면 인증서 창에 **상태** 값이 **확인되지 않음** 으로 설정된 인증서가 표시됩니다. 이 CA 인증서를 사용하여 디바이스를 DPS에 인증하려면 인증서 _보유 증명_ 을 확인해야 합니다.

1. 인증서 소지 증명 확인 프로세스를 시작하려면 **root-ca-cert** 를 클릭합니다.

1. **인증서 정보** 창 하단에서 **확인 코드 생성** 을 클릭합니다.

    **확인 코드 생성** 단추를 보려면 아래로 스크롤해야 할 수도 있습니다.

    단추를 클릭하면 생성된 코드가 확인 코드 필드(단추 바로 위에 있음)에 입력됩니다.

1. **확인 코드** 의 오른쪽에서 **클립보드에 복사** 를 클릭합니다.

    CA 인증서 보유 증명은 DPS 내에서 방금 생성된 확인 코드와 함께 CA 인증서에서 생성된 인증서를 업로드하여 DPS에 제공됩니다. 이렇게 하면 CA 인증서를 실제로 보유하고 있다는 증거를 제공할 수 있습니다.

    > **중요**: 확인 인증서를 생성하는 동안 **인증서 세부 정보** 창을 열어 두어야 합니다. 창을 닫으면 확인 코드가 무효화되고 새 코드를 생성해야 합니다.

1. **Azure Cloud Shell** 을 열고 이전에 아직 열지 않은 경우 `~/certificates` 디렉터리로 이동합니다.

1. 확인 인증서를 만들려면 다음 명령을 입력합니다.

    ```sh
    ./certGen.sh create_verification_certificate <verification-code>
    ```

    `<verification-code>` 자리 표시자를 Azure Portal에서 만든 **확인 코드** 로 바꿔야 합니다.

    예를 들어, 실행하는 명령은 다음과 유사합니다.

    ```sh
    ./certGen.sh create_verification_certificate 49C900C30C78D916C46AE9D9C124E9CFFD5FCE124696FAEA
    ```

    이렇게 하면 CA 인증서에 연결된 _확인 인증서_ 가 생성됩니다. 인증서의 주체는 확인 코드입니다. `verification-code.cert.pem`이라는 이름으로 생성된 확인 인증서는 Azure Cloud Shell의 `./certs` 디렉터리 내에 있습니다.

    다음 단계에서는 DPS에 업로드할 수 있도록 확인 인증서를 로컬 컴퓨터에 다운로드합니다(이전의 루트 인증서로 수행한 작업과 유사).

1. 확인 인증서를 로컬 컴퓨터에 다운로드하려면 다음 명령을 입력합니다.

    ```sh
    download ~/certificates/certs/verification-code.cert.pem
    ```

    > **참고**: 웹 브라우저에 따라 이 시점에서 여러 다운로드를 허용하라는 메시지가 표시될 수 있습니다. 다운로드 명령에 대한 응답이 없는 것으로 보이면 다운로드 허용 권한을 요청하는 메시지가 화면에 표시되지 않는지 확인합니다.

1. **인증서 세부 정보** 창으로 다시 전환합니다.

    DPS 서비스의 이 창이 Azure Portal에서 아직 열려 있어야 합니다.

1. **인증서 세부 정보** 창 하단의 **확인 인증서 .pem 또는 .cer 파일** 오른쪽 아래에 있는 **열기** 를 클릭합니다.

1. 파일 열기 대화 상자에서 다운로드 폴더로 이동한 다음 **verification-code.cert.pem** 을 클릭하고 **열기** 를 클릭합니다.

1. **인증서 세부 정보** 창 하단에서 **확인** 을 클릭합니다.

1. **인증서** 창에서 이제 인증서 **상태** 가 **확인됨** 으로 설정되어 있는지 확인합니다.

    이 변경 내용을 보려면 창의 상단에 있는 **새로 고침**(**추가** 단추 오른쪽)을 클릭해야 할 수 있습니다.

#### <a name="task-3-create-group-enrollment-x509-certificate-in-dps"></a>작업 3: DPS에서 그룹 등록계약(X.509 인증서) 만들기

이 작업에서는 DPS(Device Provisioning Service) 내에서 X.509 인증서 증명을 사용하는 새 등록 그룹을 만듭니다.

1. **dps-az220-training-{사용자 ID}** 블레이드의 왼쪽 메뉴에서 **설정** 아래의 **등록 관리** 를 클릭합니다.

1. **등록 관리** 창 상단에서 **등록 그룹 추가** 를 클릭합니다.

    등록 그룹은 기본적으로 자동 프로비저닝을 통해 등록할 수 있는 디바이스의 레코드입니다.

1. **등록 그룹 추가** 블레이드에서 **그룹 이름** 아래에 **eg-test-simulated-devices** 를 입력합니다.

1. **증명 유형** 이 **인증서** 로 설정되어 있는지 확인합니다.

1. **인증서 유형** 필드가 **CA 인증서** 로 설정되어 있는지 확인합니다.

1. **기본 인증서** 드롭다운에서 이전에 DPS에 업로드한 CA 인증서를 선택합니다(**root-ca-cert** 와 유사).

1. **보조 인증서** 드롭다운은 **선택한 인증서 없음** 으로 설정된 상태로 둡니다.

    보조 인증서는 일반적으로 만료 인증서 또는 손상된 인증서를 수용하기 위해 인증서 회전에 사용됩니다. 롤링 인증서에 대한 자세한 내용은 다음 위치에서 확인할 수 있습니다. [https://docs.microsoft.com/en-us/azure/iot-dps/how-to-roll-certificates](https://docs.microsoft.com/en-us/azure/iot-dps/how-to-roll-certificates)

1. **허브에 디바이스를 할당할 방법을 선택하세요** 필드를 **균등 가중 배포** 로 둡니다.

    여러 분산 허브가 있는 대규모 환경에서는 이 설정에서 이 디바이스 등록계약을 받을 IoT Hub를 선택하는 방법을 제어합니다. 이 랩의 등록과 연관된 단일 IoT Hub가 있으므로 IoT Hub에 디바이스를 할당하는 방법은 이 랩 시나리오에는 실제로 적용되지 않습니다.

1. **이 그룹에 할당할 수 있는 IoT Hub 선택** 드롭다운에서는 **iot-az220-training-{사용자 ID}** IoT Hub가 선택되어 있습니다.

    이 필드는 프로비전하는 디바이스가 올바른 IoT Hub에 추가되는지를 확인하는 데 사용됩니다.

1. **다시 프로비전닝할 때 디바이스 데이터를 처리하도록 설정하는 방법 선택** 필드는 **데이터 다시 프로비저닝 및 마이그레이션** 으로 설정된 상태로 둡니다.

    이 필드는 동일한 디바이스(동일한 등록 ID를 통해 표시된 대로)가 이미 한 번 이상 성공적으로 프로비저닝된 후 나중에 프로비저닝 요청을 제출하는 재프로비저닝 동작에 높은 수준의 제어를 제공합니다.

1. **초기 디바이스 쌍 상태** 필드에서 다음과 같이 JSON 개체를 수정합니다.

    ```json
    {
        "tags": {},
        "properties": {
            "desired": {
                "telemetryDelay": "1"
            }
        }
    }
    ```

    이 JSON 데이터는 이 등록 그룹에 참여하는 모든 디바이스의 디바이스 쌍에 대한 원하는 속성의 초기 구성을 나타냅니다.

    디바이스는 `properties.desired.telemetryDelay` 속성을 사용하여 원격 분석을 읽고 IoT Hub로 보내는 시간 지연을 설정합니다.

1. **항목 사용** 을 **사용** 으로 설정한 상태로 둡니다.

    일반적으로 새 등록 항목을 사용하도록 설정하고 사용 상태로 유지하려고 합니다.

1. **등록 그룹 추가** 블레이드 위쪽의 **저장** 을 클릭합니다.

### <a name="exercise-3-configure-simulated-device-with-x509-certificate"></a>연습 3: X.509 인증서로 시뮬레이션된 디바이스 구성

이 연습에서는 루트 인증서를 사용하여 디바이스 인증서를 생성하고, 증명용 디바이스 인증서를 사용하여 연결하는 시뮬레이션된 디바이스를 구성합니다.

#### <a name="task-1-generate-a-device-certificate"></a>작업 1: 디바이스 인증서 생성

1. 필요한 경우 Azure 계정 자격 증명을 사용하여 Azure Portal에 로그인합니다.

    Azure 계정이 두 개 이상인 경우 이 과정에 사용할 구독에 연결된 계정으로 로그인해야 합니다.

1. Azure Portal 도구 모음에서 **Cloud Shell** 을 클릭합니다.

    Azure Portal 도구 모음은 Portal 창 위쪽에 가로로 표시됩니다. Cloud Shell 단추는 오른쪽에서 6번째 단추입니다.

1. Cloud Shell에서 **Bash** 를 사용하고 있는지 확인합니다.

    Azure Cloud Shell 페이지의 왼쪽 상단에 있는 드롭다운은 환경을 선택하는 데 사용됩니다. 선택한 드롭다운 값이 **Bash** 인지 확인합니다.

1. `~/certificates` 디렉터리로 이동하려면 Cloud Shell 명령 프롬프트에서 다음 명령을 입력합니다.

    ```sh
    cd ~/certificates
    ```

    `~/certificates` 디렉터리는 `certGen.sh` 도우미 스크립트가 다운로드된 위치입니다. 이 랩 앞부분에서 해당 스크립트를 사용하여 DPS용 CA 인증서를 생성했습니다. 이 도우미 스크립트는 CA 인증서 체인 내에서 디바이스 인증서를 생성하는 데에도 사용됩니다.

1. CA 인증서 체인 내에서 X.509 디바이스 인증서를 생성하려면 다음 명령을 입력합니다.

    ```sh
    ./certGen.sh create_device_certificate sensor-thl-2000
    ```

    이 명령은 이전에 생성된 CA 인증서에 의해 서명된 새 X.509 인증서를 만듭니다. 디바이스 ID(`sensor-thl-2000`)가 `certGen.sh` 스크립트의 `create_device_certificate` 명령에 전달됩니다. 이 디바이스 ID는 디바이스 인증서의 공통 이름 또는 `CN=` 값 내에서 설정됩니다. 이 인증서는 시뮬레이션된 디바이스에 대한 리프 디바이스 X.509 인증서를 만들며 DPS(Device Provisioning Service)를 사용하여 디바이스를 인증하는 데 사용됩니다.

    `create_device_certificate` 명령이 완료되면 만들어진 X.509 디바이스 인증서의 이름이 `new-device.cert.pfx`가 되며 `/certs` 하위 디렉터리 내에 있습니다.

    > **참고**: 이 명령은 `/certs` 하위 디렉터리의 기존 디바이스 인증서를 덮어씁니다. 여러 디바이스에 대한 인증서를 만들려면 명령을 실행할 때마다 `new-device.cert.pfx`의 복사본을 저장해야 합니다.

1. 방금 만든 디바이스 인증서의 이름을 바꾸려면 다음 명령을 입력합니다.

    ```sh
    mv ~/certificates/certs/new-device.cert.pfx ~/certificates/certs/sensor-thl-2000-device.cert.pfx
    mv ~/certificates/certs/new-device.cert.pem ~/certificates/certs/sensor-thl-2000-device.cert.pem
    ```

1. 추가 디바이스 인증서를 만들려면 다음 명령을 입력합니다.

    ```sh
    ./certGen.sh create_device_certificate sensor-thl-2001
    mv ~/certificates/certs/new-device.cert.pfx ~/certificates/certs/sensor-thl-2001-device.cert.pfx
    mv ~/certificates/certs/new-device.cert.pem ~/certificates/certs/sensor-thl-2001-device.cert.pem
    ```

1. Cloud Shell에서 생성된 X.509 디바이스 인증서를 로컬 컴퓨터로 다운로드하려면 다음 명령을 입력합니다.

    ```sh
    download ~/certificates/certs/sensor-thl-2000-device.cert.pfx
    download ~/certificates/certs/sensor-thl-2001-device.cert.pfx
    ```

    다음 작업에서는 X.509 디바이스 인증서를 사용하여 Device Provisioning Service에 인증을 하는 시뮬레이션된 디바이스 빌드를 시작합니다.

#### <a name="task-2-configure-a-simulated-device"></a>작업 2: 시뮬레이션된 디바이스 구성

이 작업에서는 다음 작업을 완료합니다.

* DPS에서 코드에 포함할 ID 범위 가져오기
* 다운로드한 디바이스 인증서를 애플리케이션 루트 폴더에 복사
* Visual Studio Code에서 애플리케이션 구성

1. Azure Portal에서 Device Provisioning Service 블레이드를 열고 **개요** 창이 선택되어 있는지 확인합니다.

1. **개요** 창에서 Device Provisioning Service의 **ID 범위** 를 복사한 후 나중에 참조할 수 있도록 저장합니다.

    값 위로 마우스를 가져가면 표시되는 값의 오른쪽에 복사 단추가 있습니다.

    **ID 범위** 는 값 `0ne0004E52G`와 유사합니다.

1. Windows 파일 탐색기를 연 다음 `sensor-thl-2000-device.cert.pfx` 인증서 파일이 다운로드된 폴더로 이동합니다.

1. 파일 탐색기를 사용하여 디바이스 인증서 파일 2개의 복사본을 만듭니다.

    지금 인증서 파일 두 개를 모두 복사하면 시간을 절약할 수 있습니다. 하지만 처음 빌드하는 코드 프로젝트에서는 첫 번째 인증서인 `sensor-thl-2000-device.cert.pfx`만 사용합니다.

1. 파일 탐색기에서 랩 6(DPS에서 디바이스의 자동 등록계약)에 대한 시작 폴더로 이동합니다.

    랩 3: 개발 환경 설정에서 ZIP 파일을 다운로드하고 콘텐츠를 로컬로 추출하여 랩 리소스를 포함하는 GitHub 리포지토리를 복제했습니다. 추출된 폴더 구조에는 다음 폴더 경로가 포함됩니다.

    * Allfiles
      * 랩
          * 06-DPS에 있는 디바이스의 자동 등록계약
            * Starter
              * ContainerDevice

1. ContainerDevice 폴더를 열어 두고 복사한 디바이스 인증서 파일을 해당 폴더에 붙여넣습니다.

    ContainerDevice 폴더의 루트 디렉터리에는 시뮬레이션된 디바이스 앱용 `Program.cs` 파일이 포함되어 있습니다. 시뮬레이션된 디바이스 앱은 Device Provisioning Service에 인증할 때 디바이스 인증서 파일을 사용합니다.

1. **Visual Studio Code** 를 사용하여 ASP.NET 5 API 앱을 만드는 방법을 보여줍니다.

1. **파일** 메뉴에서 **폴더 열기** 를 클릭합니다.

1. **폴더 열기** 대화 상자에서 랩 6(DPS에서 디바이스 자동 등록)의 Starter 폴더로 이동합니다.

1. **ContainerDevice** 를 클릭하고 **폴더 선택** 을 클릭합니다.

    Visual Studio Code의 EXPLORER 창에 다음 파일이 나열되어야 합니다.

    * ContainerDevice.csproj
    * Program.cs
    * sensor-thl-2000-device.cert.pfx

    **참고**: 이 폴더에 복사한 다른 디바이스 인증서 파일은 이 랩의 뒷부분에서 사용됩니다. 여기서는 첫 번째 디바이스 인증서 파일 구현 과정만 중점적으로 진행합니다.

1. **탐색기** 창에서 ContainerDevice.csproj 파일을 열려면 **ContainerDevice.csproj** 를 클릭합니다.

1. 코드 편집기 창에서 `<ItemGroup>` 태그 내의 인증서 파일 이름을 다음과 같이 업데이트합니다.

    ```xml
    <ItemGroup>
        <None Update="sensor-thl-2000-device.cert.pfx" CopyToOutputDirectory="PreserveNewest" />
        <PackageReference Include="Microsoft.Azure.Devices.Client" Version="1.*" />
        <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Mqtt" Version="1.*" />
        <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Amqp" Version="1.*" />
        <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Http" Version="1.*" />
    </ItemGroup>
    ```

    이 구성을 사용하면 C# 코드가 컴파일될 때 `sensor-thl-2000-device.cert.pfx` 인증서 파일이 빌드 폴더에 복사되고 실행될 때 프로그램에 액세스할 수 있습니다.

1. Visual Studio Code **파일** 메뉴에서 **저장** 을 클릭합니다.

    **참고**: Visual Studio Code에서 복원 메시지가 표시되면 지금 복원을 수행합니다.

1. **탐색기** 창에서 **Program.cs** 를 클릭합니다.

    이 **ContainerDevice** 애플리케이션 버전은 이전 랩에서 사용한 버전과 사실상 동일함을 쉽게 확인할 수 있습니다. 변경된 부분은 증명 메커니즘으로 X.509 인증서를 사용하도록 지정하는 코드와 구체적으로 관련된 부분뿐입니다. 이 디바이스가 연결할 때 사용하는 방법(그룹 등록 또는 개별 등록)은 애플리케이션에서 전혀 중요하지 않습니다.

1. **GlobalDeviceEndpoint** 변수를 찾아 해당 값이 Azure Device Provisioning Service용 전역 디바이스 엔드포인트(`global.azure-devices-provisioning.net`)로 설정되어 있음을 확인합니다.

    퍼블릭 Azure Cloud 내에서 **global.azure-devices-provisioning.net** 은 DPS(Device Provisioning Service)의 전역 디바이스 엔드포인트입니다. Azure DPS에 연결하는 모든 디바이스는 이 글로벌 디바이스 엔드포인트 DNS 이름으로 구성됩니다. 다음과 유사한 코드가 표시됩니다.

    ```csharp
    private const string GlobalDeviceEndpoint = "global.azure-devices-provisioning.net";
    ```

1. **dpsIdScope** 변수를 찾은 다음 Device Provisioning Service의 개요 창에서 복사한 **ID 범위** 를 사용하여 이 변수에 할당된 값을 업데이트합니다.

    코드를 업데이트한 경우 다음과 유사하게 보입니다.

    ```csharp
    private static string dpsIdScope = "0ne000CBD6C";
    ```

1. **certificateFileName** 변수를 찾아 해당 값이 생성한 디바이스 인증서 파일의 기본 이름(**new-device.cert.pfx**)으로 설정되어 있음을 확인합니다.

    이 랩에서는 애플리케이션이 이전 랩에서처럼 대칭 키를 사용하는 대신 X.509 인증서를 사용합니다. **new-device.cert.pfx** 파일은 Cloud Shell 내의 **certGen.sh** 도우미 스크립트를 사용하여 생성한 X.509 디바이스 인증서 파일입니다. 이 변수는 Device Provisioning Service로 인증할 때 사용할 X.509 디바이스 인증서를 포함하는 파일을 디바이스 코드에 알려줍니다.

1. **certificateFileName** 변수에 할당된 값을 다음과 같이 업데이트합니다.

    ```csharp
    private static string certificateFileName = "sensor-thl-2000-device.cert.pfx";
    ```

1. **certificatePassword** 변수를 찾아 해당 값이 **certGen.sh** 스크립트에 의해 정의된 기본 암호로 설정되어 있는지 확인합니다.

    **certificatePassword** 변수에는 X.509 디바이스 인증서용 암호가 포함되어 있습니다. 이 암호는 `1234`로 설정되어 있습니다. X.509 인증서를 생성할 때 **certGen.sh** 도우미 스크립트에서 사용하는 기본 암호이기 때문입니다.

    > **참고**: 이 랩을 위해 암호는 하드 코딩됩니다. _프로덕션_ 시나리오에서 암호는 Azure Key Vault와 같이 더 안전한 방식으로 저장되어야 합니다. 또한 인증서 파일(PFX)은 HSM(하드웨어 보안 모듈)을 사용하여 프로덕션 디바이스에 안전하게 저장되어야 합니다.
    >
    > HSM(하드웨어 보안 모듈)은 디바이스 보안의 안전한 하드웨어 기반 저장에 사용되며 가장 안전한 보안 스토리지 형태입니다. X.509 인증서 및 SAS 토큰은 HSM에 저장될 수 있습니다. HSM은 프로비저닝 서비스가 지원하는 모든 증명 메커니즘과 함께 사용할 수 있습니다. HMS에 대해서는 이 과정의 뒷부분에서 자세히 설명합니다.

1. Visual Studio Code **파일** 메뉴에서 **저장** 을 클릭합니다.

    이제 시뮬레이션된 디바이스에서 Azure IoT Hub의 디바이스 쌍 속성을 사용하여 원격 분석 메시지 간의 지연을 설정합니다.

1. **터미널** 메뉴에서 **새 터미널** 을 클릭합니다.

1. 코드가 정상적으로 빌드되는지 확인하려면 터미널 명령 프롬프트에서 **dotnet build** 를 입력합니다.

    빌드 오류가 표시되면 지금 오류를 해결하고 다음 연습을 진행하세요. 필요한 경우 강사의 도움을 받을 수 있습니다.

### <a name="exercise-4-create-additional-instances-of-your-simulated-device"></a>연습 4: 시뮬레이션된 디바이스의 추가 인스턴스 만들기

이 연습에서는 시뮬레이션된 디바이스 프로젝트 복사본을 만듭니다. 그런 다음 앞에서 만들어 프로젝트 폴더에 추가한 여러 디바이스 인증서를 사용하도록 코드를 업데이트합니다.

#### <a name="task-1-make-copies-of-your-code-project"></a>작업 1: 코드 프로젝트 복사본 만들기

1. Windows 파일 탐색기를 엽니다.

1. 파일 탐색기에서 랩 6(DPS에서 디바이스의 자동 등록계약)에 대한 시작 폴더로 이동합니다.

    랩 3: 개발 환경 설정에서 ZIP 파일을 다운로드하고 콘텐츠를 로컬로 추출하여 랩 리소스를 포함하는 GitHub 리포지토리를 복제했습니다. 추출된 폴더 구조에는 다음 폴더 경로가 포함됩니다.

    * Allfiles
      * 랩
          * 06-DPS에 있는 디바이스의 자동 등록계약
            * Starter

1. **ContainerDevice** 를 마우스 오른쪽 단추로 클릭하고 **복사** 를 클릭합니다.

    ContainerDevice 폴더는 시뮬레이션된 디바이스 코드가 포함된 폴더입니다.

1. **ContainerDevice** 아래의 빈 공간을 마우스 오른쪽 단추로 클릭하고 **붙여넣기** 를 클릭합니다.

    "ContainerDevice - 복사본" 폴더가 작성되었음을 확인할 수 있습니다.

1. **ContainerDevice - 복사본** 을 마우스 오른쪽 단추로 클릭하고 **이름 바꾸기** 를 클릭한 후 **ContainerDevice2001** 을 입력합니다.

#### <a name="task-2-update-the-certificate-file-references-in-your-code-project"></a>작업 2: 코드 프로젝트에서 인증서 파일 참조 업데이트

1. 필요하면 Visual Studio Code를 엽니다.

1. **파일** 메뉴에서 **폴더 열기** 를 클릭합니다.

1. 랩 6 Starter 폴더로 이동합니다.

1. **ContainerDevice2001** 을 클릭하고 **폴더 선택** 을 클릭합니다.

1. 탐색기 창에서 **Program.cs** 를 클릭합니다.

1. 코드 편집기에서 **certificateFileName** 변수를 찾은 다음 **certificateFileName** 변수에 할당된 값을 다음과 같이 업데이트합니다.

    ```csharp
    private static string certificateFileName = "sensor-thl-2001-device.cert.pfx";
    ```

1. **탐색기** 창에서 ContainerDevice.csproj 파일을 열려면 **ContainerDevice.csproj** 를 클릭합니다.

1. 코드 편집기 창에서 `<ItemGroup>` 태그 내의 인증서 파일 이름을 다음과 같이 업데이트합니다.

    ```xml
    <ItemGroup>
        <None Update="sensor-thl-2001-device.cert.pfx" CopyToOutputDirectory="PreserveNewest" />
        <PackageReference Include="Microsoft.Azure.Devices.Client" Version="1.*" />
        <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Mqtt" Version="1.*" />
        <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Amqp" Version="1.*" />
        <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Http" Version="1.*" />
    </ItemGroup>
    ```

1. **파일** 메뉴에서 **모두 저장** 을 클릭합니다.

### <a name="exercise-5-test-the-simulated-device"></a>연습5: 시뮬레이션된 디바이스 테스트

이 연습에서는 시뮬레이션된 디바이스를 실행합니다. 디바이스가 처음 시작되면 DPS(Device Provisioning Service)에 연결되고 구성된 그룹 등록을 사용하여 자동으로 등록됩니다. DPS 그룹 등록에 등록하면 Azure IoT Hub 디바이스 레지스트리에 디바이스가 자동으로 등록됩니다. 등록되면 디바이스는 구성된 X.509 인증서 인증을 사용하여 Azure IoT Hub와 안전하게 통신합니다.

#### <a name="task-1-build-and-run-the-simulated-device-projects"></a>작업 1: 시뮬레이션된 디바이스 프로젝트 빌드 및 실행

1. Visual Studio Code가 열려 있는지 확인합니다.

1. **파일** 메뉴에서 **폴더 열기** 를 클릭합니다.

1. 랩 6 Starter 폴더로 이동합니다.

1. **ContainerDevice** 를 클릭하고 **폴더 선택** 을 클릭합니다.

1. **보기** 메뉴에서 **터미널** 을 클릭합니다.

    이렇게 하면 Visual Studio Code 창 하단에 통합 터미널이 열립니다.

1. 터미널 명령 프롬프트에서 현재 디렉터리 경로가 `\ContainerDevice` 폴더로 설정되어 있는지 확인합니다.

    다음과 비슷한 내용이 표시됩니다.

    `Allfiles\Labs\06-Automatic Enrollment of Devices in DPS\Starter\ContainerDevice>`

1. **ContainerDevice** 프로젝트를 빌드하고 실행하려면 다음 명령을 입력합니다.

    ```cmd/sh
    dotnet run
    ```

    > **참고**:  시뮬레이션된 디바이스를 처음 실행할 때 가장 흔히 발생하는 오류는 _잘못된 인증서_ 오류입니다. `ProvisioningTransportException` 예외가 표시되는 경우 이 오류 때문에 예외가 발생했을 가능성이 높습니다. 아래에 나와 있는 것과 비슷한 메시지가 표시되면 DPS의 CA 인증서, 그리고 시뮬레이션된 디바이스 애플리케이션용 디바이스 인증서가 올바르게 구성되었는지를 확인해야 작업을 계속 진행할 수 있습니다.
    >
    > ```text
    > localmachine:LabFiles User$ dotnet run
    > Found certificate: AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=sensor-thl-2000; PrivateKey: True
    > Using certificate AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=sensor-thl-2000
    > RegistrationID = sensor-thl-2000
    > ProvisioningClient RegisterAsync . . . Unhandled exception. Microsoft.Azure.Devices.Provisioning.Client.ProvisioningTransportException: {"errorCode":401002,"trackingId":"2e298c80-0974-493c-9fd9-6253fb055ade","message":"Invalid certificate.","timestampUtc":"2019-12-13T14:55:40.2764134Z"}
    >   at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.ValidateOutcome(Outcome outcome)
    >   at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.RegisterDeviceAsync(AmqpClientConnection client, String correlationId, DeviceRegistration deviceRegistration)
    >   at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.RegisterAsync(ProvisioningTransportRegisterMessage message, CancellationToken cancellationToken)
    >   at X509CertificateContainerDevice.ProvisioningDeviceLogic.RunAsync() in /Users/User/Documents/AZ-220/LabFiles/Program.cs:line 121
    >   at X509CertificateContainerDevice.Program.Main(String[] args) in /Users/User/Documents/AZ-220/LabFiles/Program.cs:line 55
    > ...
    > ```

1. 시뮬레이션된 디바이스 앱은 터미널 창으로 출력을 전송합니다.

    시뮬레이션된 디바이스 애플리케이션이 올바르게 실행되면 **터미널** 에 앱의 콘솔 출력이 표시됩니다.

    터미널 창에 표시된 정보의 위쪽까지 스크롤합니다.

    X.509 인증서가 로드되었고 디바이스가 Device Provisioning Service에 등록되었으며 **iot-az220-training-{사용자 ID}** IoT Hub에 연결하도록 할당되었습니다. 그리고 디바이스 트윈의 원하는 속성이 로드되었습니다.

    ```text
    localmachine:LabFiles User$ dotnet run
    Found certificate: AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=sensor-thl-2000; PrivateKey: True
    Using certificate AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=sensor-thl-2000
    RegistrationID = sensor-thl-2000
    ProvisioningClient RegisterAsync . . . Device Registration Status: Assigned
    ProvisioningClient AssignedHub: iot-az220-training-CP1119.azure-devices.net; DeviceID: sensor-thl-2000
    Creating X509 DeviceClient authentication.
    simulated device. Ctrl-C to exit.
    DeviceClient OpenAsync.
    Connecting SetDesiredPropertyUpdateCallbackAsync event handler...
    Loading Device Twin Properties...
    Desired Twin Property Changed:
    {"$version":1}
    Reported Twin Properties:
    {"telemetryDelay":1}
    Start reading and sending device telemetry...
    ```

    시뮬레이션된 디바이스의 소스 코드를 검토하려면 `Program.cs` 소스 코드 파일을 엽니다. 콘솔에 표시된 메시지를 출력하는 데 사용되는 여러 `Console.WriteLine` 문을 찾습니다.

1. JSON 형식의 원격 분석 메시지가 Azure IoT Hub로 전송되고 있습니다.

    ```text
    Start reading and sending device telemetry...
    12/9/2019 5:47:00 PM > Sending message: {"temperature":24.047539159212047,"humidity":67.00504162675004,"pressure":1018.8478924248358,"latitude":40.129349260196875,"longitude":-98.42877188146265}
    12/9/2019 5:47:01 PM > Sending message: {"temperature":26.628804161040485,"humidity":68.09610794675355,"pressure":1014.6454375411363,"latitude":40.093269544242695,"longitude":-98.22227128174003}
    ```

    시뮬레이션된 디바이스가 초기 시작을 통과하면 Azure IoT Hub로 시뮬레이션된 센서 원격 분석 메시지를 보내기 시작합니다.

    IoT Hub로 전송되는 각 메시지 간의 `telemetryDelay` 디바이스 쌍 속성에 정의된 지연이 현재 센서 원격 분석 메시지를 보내는 사이에 **1초** 지연되고 있습니다.

1. 시뮬레이션된 디바이스를 실행 상태로 둡니다.

#### <a name="task-2-start-the-other-simulated-device"></a>작업 2: 다른 시뮬레이션된 디바이스 시작

1. Visual Studio Code의 새 인스턴스를 엽니다.

    Windows 10 시작 메뉴에서 다음 방법으로 Visual Studio Code의 새 인스턴스를 열 수 있습니다. Windows 10 **시작** 메뉴에서 **Visual Studio Code** 를 마우스 오른쪽 단추로 클릭하고 **새 창** 을 클릭합니다.

1. 새 Visual Studio Code 창의 **파일** 메뉴에서 **폴더 열기** 를 클릭합니다.

1. 랩 6 Starter 폴더로 이동합니다.

1. **ContainerDevice2001** 을 클릭하고 **폴더 선택** 을 클릭합니다.

1. **보기** 메뉴에서 **터미널** 을 클릭합니다.

    이렇게 하면 Visual Studio Code 창 하단에 통합 터미널이 열립니다.

1. 터미널 명령 프롬프트에서 현재 디렉터리 경로가 `\ContainerDevice2001` 폴더로 설정되어 있는지 확인합니다.

    다음과 비슷한 내용이 표시됩니다.

    `Allfiles\Labs\06-Automatic Enrollment of Devices in DPS\Starter\ContainerDevice2001>`

1. **ContainerDevice** 프로젝트를 빌드하고 실행하려면 다음 명령을 입력합니다.

    ```cmd/sh
    dotnet run
    ```

#### <a name="task-3-change-the-device-configuration-through-its-twin"></a>작업 3: 쌍을 통해 디바이스 구성 변경

시뮬레이션된 디바이스가 실행 중인 상태에서 Azure IoT Hub 내에서 디바이스 쌍의 필요한 상태를 편집하여 `telemetryDelay` 구성을 업데이트할 수 있습니다.

1. **Azure Portal** 을 연 다음 **Azure IoT Hub** 서비스로 이동합니다.

1. IoT Hub 블레이드의 왼쪽 메뉴에서 **탐색기** 아래의 **IoT 디바이스** 를 클릭합니다.

1. IoT 디바이스 목록에서 **sensor-thl-2000** 을 클릭합니다.

    > **중요**: 이 랩에서 디바이스를 선택해야 합니다. 이 과정의 앞부분에서 만든 _sensor-th-0001_ 디바이스가 표시될 수도 있습니다.

1. **sensor-thl-2000** 디바이스 블레이드에서 블레이드 상단의 **디바이스 트윈** 을 클릭합니다.

    **디바이스 트윈** 블레이드의 편집기에 디바이스 트윈의 전체 JSON이 표시됩니다. 이렇게 하면 Azure Portal에서 디바이스 쌍 상태를 직접 보거나 편집할 수 있습니다.

1. 디바이스 쌍 JSON 내에서 `properties.desired` 노드를 찾습니다.

1. `telemetryDelay` 속성을 `"2"` 값으로 업데이트합니다.

    시뮬레이션된 디바이스의 `telemetryDelay`가 업데이트되어 **2초** 마다 센서 원격 분석을 전송합니다.

    디바이스 쌍이 원하는 속성의 이 섹션에 대한 결과 JSON이 다음과 유사합니다.

    ```json
    "properties": {
        "desired": {
          "telemetryDelay": "2",
          "$metadata": {
            "$lastUpdated": "2019-12-09T22:48:05.9703541Z",
            "$lastUpdatedVersion": 2,
            "telemetryDelay": {
              "$lastUpdated": "2019-12-09T22:48:05.9703541Z",
              "$lastUpdatedVersion": 2
            }
          },
          "$version": 2
        },
    ```

    JSON 내에서 `$metadata` 노드의 `$version` 값과 `properties.desired` 값을 그대로 둡니다. `telemetryDelay` 값을 업데이트하여 새 디바이스 쌍에 필요한 속성 값을 설정해야 합니다.

1. 블레이드 상단에서 디바이스의 디바이스 트윈 원하는 속성을 적용하려면 **저장** 을 클릭합니다.

    일단 저장되면, 업데이트된 디바이스 쌍이 원하는 속성은 자동으로 시뮬레이션된 디바이스로 전송됩니다.

1. 원래 **ContainerDevice** 프로젝트가 표시되어 있는 **Visual Studio Code** 창으로 다시 전환합니다.

1. 업데이트된 디바이스 쌍 `telemetryDelay`에 필요한 속성 설정이 애플리케이션에 전송된 상태입니다.

    애플리케이션은, 새 디바이스 쌍이 원하는 속성이 로드되었으며 변경 내용이 설정되어 Azure IoT Hub에 다시 보고되었음을 보여주는 메시지를 콘솔에 출력합니다.

    ```text
    Desired Twin Property Changed:
    {"telemetryDelay":2,"$version":2}
    Reported Twin Properties:
    {"telemetryDelay":2}
    ```

1. 이제 시뮬레이션된 디바이스 센서 원격 분석 메시지가 _2_ 초마다 Azure IoT Hub로 전송됩니다.

    ```text
    12/9/2019 5:48:07 PM > Sending message: {"temperature":33.89822140284731,"humidity":78.34939097908763,"pressure":1024.9467544610131,"latitude":40.020042418755764,"longitude":-98.41923808825841}
    12/9/2019 5:48:09 PM > Sending message: {"temperature":27.475786026323114,"humidity":64.4175510594703,"pressure":1020.6866468579678,"latitude":40.2089999240047,"longitude":-98.26223221770334}
    12/9/2019 5:48:11 PM > Sending message: {"temperature":34.63600901637041,"humidity":60.95207713588703,"pressure":1013.6262313688063,"latitude":40.25499096898331,"longitude":-98.51199886959347}
    ```

1. **터미널** 창 내에서 시뮬레이션된 디바이스 앱을 종료하려면 **Ctrl+C** 를 누릅니다.

1. 각 Visual Studio Code 창으로 전환한 다음 **터미널** 프롬프트를 사용하여 시뮬레이션된 디바이스 앱을 닫습니다.

1. Azure Portal 창으로 전환합니다.

1. **디바이스 트윈** 블레이드를 닫습니다.

1. **sensor-thl-2000** 블레이드에서 **디바이스 트윈** 을 클릭합니다.

1. 아래로 스크롤하여 `properties.reported` 개체에 대한 JSON을 찾습니다.

    여기에는 디바이스에서 보고한 상태가 포함됩니다.

1. `telemetryDelay` 속성도 여기에 있으며 역시 `2`로 설정되었습니다.

    `$metadata` 값이 마지막으로 업데이트된 시간을 표시하는 `reported` 값도 있습니다.

1. **디바이스 쌍** 블레이드를 다시 닫습니다.

1. **sensor-thl-2000** 블레이드를 닫은 다음 Azure Portal 대시보드로 다시 이동합니다.

### <a name="exercise-6-deprovision-a-single-device-from-the-group-enrollment"></a>연습 6: 그룹 등록에서 단일 디바이스 프로비전 해제

여러 가지 이유로 인해 그룹 등록의 일부분으로 등록된 디바이스 중 일부만 프로비전해야 할 수 있습니다. 디바이스 하나가 더 이상 필요하지 않거나, 최신 버전 디바이스를 사용할 수 있게 되었거나, 디바이스가 파손 또는 손상된 경우 등을 예로 들 수 있습니다.

등록 그룹에서 단일 디바이스의 프로비전을 해제하려면 다음의 두 가지 작업을 수행해야 합니다.

* 디바이스 리프(디바이스) 인증서에 대해 사용하지 않도록 설정된 개별 등록을 만듭니다.

    이렇게 하면 인증서 체인에 등록 그룹의 서명 인증서가 있는 다른 디바이스에 대한 액세스를 허용하면서도 해당 디바이스에 대한 프로비전 서비스에 대한 액세스가 취소됩니다. 디바이스용으로 사용하지 않도록 설정된 개별 등록을 삭제해서는 안 됩니다. 이러한 등록을 삭제하면 디바이스가 등록 그룹을 통해 다시 등록할 수 있기 때문입니다.

* IoT 허브의 ID 레지스트리에서 디바이스를 비활성화 또는 삭제합니다.

    솔루션에 IoT Hub가 여러 개 포함되어 있다면 등록 그룹에 프로비전된 디바이스 목록을 사용하여 디바이스가 프로비전된 IoT Hub를 찾아야 합니다(그래야 디바이스를 사용하지 않도록 설정하거나 삭제할 수 있음). 여기서는 IoT Hub가 하나뿐이므로 프로비전에 사용한 IoT Hub를 조회할 필요가 없습니다.

이 연습에서는 등록 그룹에서 단일 디바이스의 프로비전을 해제합니다.

#### <a name="task-1-create-a-disabled-individual-enrollment-for-the-device"></a>작업 1: 디바이스에 대한 사용하지 않도록 설정된 개별 등록 만들기

이 작업에서는 **sensor-thl-2001** 디바이스를 개별 등록에 사용합니다.

1. 필요한 경우 Azure 계정 자격 증명을 사용하여 Azure Portal에 로그인합니다.

    Azure 계정이 두 개 이상인 경우 이 과정에 사용할 구독에 연결된 계정으로 로그인해야 합니다.

1. Azure Portal 도구 모음에서 **Cloud Shell** 을 클릭합니다.

    Azure Portal 도구 모음은 Portal 창 위쪽에 가로로 표시됩니다. Cloud Shell 단추는 오른쪽에서 6번째 단추입니다.

1. Cloud Shell에서 **Bash** 를 사용하고 있는지 확인합니다.

    Azure Cloud Shell 페이지의 왼쪽 상단에 있는 드롭다운은 환경을 선택하는 데 사용됩니다. 선택한 드롭다운 값이 **Bash** 인지 확인합니다.

1. `~/certificates` 디렉터리로 이동하려면 Cloud Shell 명령 프롬프트에서 다음 명령을 입력합니다.

    ```sh
    cd ~/certificates
    ```

1. .pem 디바이스 인증서를 Cloud Shell에서 로컬 컴퓨터로 다운로드하려면 다음 명령을 입력합니다.

    ```sh
    download ~/certificates/certs/sensor-thl-2001-device.cert.pem
    ```

1. Azure 대시보드로 전환합니다.

1. 리소스 타일에서 **dps-az220-training-{사용자 ID}** 를 클릭합니다.

1. DPS 블레이드 왼쪽 메뉴의 **설정** 에서 **등록 관리** 를 클릭합니다.

1. **등록 관리** 창에서 **개별 등록 추가** 를 클릭합니다.

1. **등록 추가** 블레이드의 **메커니즘** 에서 **X.509** 가 선택되어 있는지 확인합니다.

1. **기본 인증서 .pem 또는 .cer 파일** 에서 **열기** 를 클릭합니다.

1. **열기** 대화 상자에서 downloads 폴더로 이동합니다.

1. downloads 폴더에서 **sensor-thl-2001-device.cert.pem** 을 클릭하고 **열기** 를 클릭합니다.

1. **등록 추가** 블레이드에서 **IoT Hub 디바이스 ID** 아래에 **sensor-thl-2001** 을 입력합니다.

1. **항목 사용** 에서 **사용 안 함** 을 클릭합니다.

1. 블레이드 위쪽에서 **저장** 을 클릭합니다.

1. Azure 대시보드로 다시 이동합니다.

#### <a name="task-2-deregister-the-device-from-iot-hub"></a>작업 2: IoT Hub에서 디바이스 등록 취소

1. 리소스 타일에서 **iot-az220-training-{사용자 ID}** 를 클릭합니다.

1. IoT Hub 블레이드 왼쪽 메뉴의 **디바이스 관리** 에서 **디바이스** 를 클릭합니다.

1. **IoT 디바이스** 창의 **디바이스 ID** 아래에서 **sensor-thl-2001** 디바이스를 찾습니다.

1. **sensor-thl-2001** 왼쪽의 체크박스를 클릭합니다.

1. **IoT 디바이스** 창 상단에서 **삭제** 를 클릭한 다음 **예** 를 클릭합니다.

#### <a name="task-3-confirm-that-the-device-is-deprovisioned"></a>작업 3: 디바이스 프로비전이 해제되었는지 확인

1. ContainerDevice2004 코드 프로젝트가 표시되어 있는 Visual Studio Code 창으로 전환합니다.

    이전 연습 후에 Visual Studio Code를 닫은 경우 Visual Studio Code를 사용하여 ContainerDevice2004 폴더를 엽니다.

1. **보기** 메뉴에서 **터미널** 을 클릭합니다.

1. 명령 프롬프트의 현재 위치가 **ContainerDevice2001** 폴더인지 확인합니다.

1. 시뮬레이션된 디바이스 앱 실행을 시작하려면 다음 명령을 입력합니다.

    ```cmd/sh
    dotnet run
    ```

1. 디바이스에서 프로비전할 때 나열된 예외를 확인합니다.

    디바이스가 Device Provisioning Service 연결 및 인증을 시도하면 서비스는 먼저 디바이스 자격 증명과 일치하는 개별 등록을 찾습니다. 그런 다음, 서비스가 등록 그룹을 검색하여 디바이스의 프로비전 가능 여부를 판단합니다. 서비스에서 디바이스에 대해 개별 등록이 사용하지 않도록 설정된 것을 확인하면 디바이스 연결을 차단합니다. 디바이스의 인증서 체인에 중간 또는 루트 CA에 대해 사용되는 등록 그룹이 있더라도 서비스에서는 연결을 차단합니다.

    애플리케이션이 구성된 X.509 인증서를 사용하여 DPS에 연결하려고 하면 DPS는 DeviceRegistrationResult.Status is NOT 'Assigned'를 반환합니다.

    ```txt
    Found certificate: 13F32448E03F451E897B681758BAC593A60BFBFA CN=sensor-thl-2001; PrivateKey: True
    Using certificate 13F32448E03F451E897B681758BAC593A60BFBFA CN=sensor-thl-2001
    ProvisioningClient AssignedHub: ; DeviceID:
    Unhandled exception. System.Exception: DeviceRegistrationResult.Status is NOT 'Assigned'
    at ContainerDevice.Program.ProvisionDevice(ProvisioningDeviceClient provisioningDeviceClient, SecurityProviderX509Certificate security) in C:\Users\howdc\Allfiles\Labs\06-Automatic Enrollment of Devices
    in DPS\Starter\ContainerDevice2004\Program.cs:line 107
    at ContainerDevice.Program.Main(String[] args) in C:\Users\howdc\Allfiles\Labs\06-Automatic Enrollment of Devices in DPS\Starter\ContainerDevice2004\Program.cs:line 49
    at ContainerDevice.Program.<Main>(String[] args)
    ```

    Azure Portal로 돌아가서 개별 등록을 사용하도록 설정하거나 삭제하면 디바이스는 다시 DPS에 인증하여 IoT Hub에 연결할 수 있습니다. 개별 등록을 삭제하면 디바이스는 그룹 등록에 자동으로 다시 추가됩니다.

### <a name="exercise-7-deprovision-the-group-enrollment"></a>연습 7: 그룹 등록 프로비전 해제

이 연습에서는 전체 등록 그룹의 프로비전을 해제합니다. 이번에도 Device Provisioning Service와 IoT Hubv에서 디바이스 등록을 취소합니다.

#### <a name="task-1-disenroll-the-enrollment-group-from-the-dps"></a>작업 1: DPS에서 등록 그룹 등록 취소


이 작업에서는 등록 그룹을 삭제합니다. 그러면 등록된 디바이스가 제거됩니다.

1. Azure 대시보드로 이동합니다.

1. 리소스 타일에서 **dps-az220-training-{사용자 ID}** 를 클릭합니다.

1. DPS 블레이드 왼쪽 메뉴의 **설정** 에서 **등록 관리** 를 클릭합니다.

1. **등록 그룹** 목록의 **그룹 이름** 아래에서 **eg-test-simulated-devices** 를 클릭합니다.

1. **등록 그룹 세부 정보** 블레이드에서 아래로 스크롤하여 **항목 사용** 필드를 찾은 다음 **사용 중지** 를 클릭합니다.

    DPS 내에서 그룹 등록계약을 사용 중지하면 이 등록계약 그룹 내의 디바이스를 일시적으로 사용 중지할 수 있습니다. 이렇게 하면 이러한 디바이스에서 사용하면 안 되는 X.509 인증서의 임시 목록이 제공됩니다.

1. 블레이드 위쪽에서 **저장** 을 클릭합니다.

    지금 시뮬레이션된 디바이스 중 하나를 실행하면 사용하지 않도록 설정한 개별 등록에서 표시되었던 것과 비슷한 오류 메시지가 표시됩니다.

    등록 그룹을 영구적으로 삭제하려면 DPS에서 등록 그룹을 삭제해야 합니다.

1. **등록 관리** 창의 **그룹 이름** 에서 **eg-test-simulated-devices** 왼쪽의 체크박스를 선택합니다.

    **simulated-devices** 의 왼쪽에 있는 확인란이 이미 선택된 경우 그 상태로 둡니다.

1. **등록 관리** 창 상단에서 **삭제** 를 클릭합니다.

1. **등록 제거** 작업을 확인하라는 메시지가 표시되면 **예** 를 클릭합니다.

    그룹 등록계약을 삭제하면 DPS에서 완전히 제거되고 다시 추가하려면 다시 만들어야 합니다.

    > **참고**:  인증서에 대한 등록 그룹을 삭제하는 경우 인증서 체인에 인증서가 있는 디바이스는 루트 인증서 또는 인증서 체인의 상위에 있는 다른 중간 인증서에 대해 활성화된 다른 등록 그룹이 여전히 존재하는 경우에도 등록할 수 있습니다.

1. Azure Portal 대시보드로 다시 이동합니다.

#### <a name="task-2-deregister-the-devices-from-the-iot-hub"></a>작업 2: IoT Hub에서 디바이스 등록 취소

등록 그룹이 DPS(Device Provisioning Service)에서 제거된 후에도 디바이스 등록은 Azure IoT Hub 내에 계속 존재합니다. 디바이스를 완전히 프로비전 해제하려면 해당 등록도 제거해야 합니다.

1. 리소스 타일에서 **iot-az220-training-{사용자 ID}** 를 클릭합니다.

1. IoT Hub 블레이드 왼쪽 메뉴의 **디바이스 관리** 에서 **디바이스** 를 클릭합니다.

1. **sensor-thl-2000** 및 기타 그룹 등록 디바이스의 디바이스 ID는 Azure IoT Hub 디바이스 레지스트리 내에 계속 포함되어 있습니다.

1. sensor-thl-2000 디바이스를 제거하려면 **sensor-thl-2000** 왼쪽에 있는 체크박스를 선택하고 **삭제** 를 클릭합니다.

1. “_선택한 디바이스를 삭제하시겠습니까?_ ”라는 메시지가 표시되면 **예** 를 클릭합니다.

1. 위의 4~5단계를 반복하여 sensor-thl-2001 디바이스를 제거합니다.

#### <a name="task-3-confirm-that-your-devices-have-been-deprovisioned"></a>작업 3: 디바이스 프로비전이 해제되었는지 확인

이전 작업에서 Device Provisioning Service에서 그룹 등록을 삭제하고 Azure IoT Hub 디바이스 레지스트리에서 디바이스를 삭제하여 디바이스를 솔루션에서 완전히 제거했습니다.

1. ContainerDevice 코드 프로젝트가 표시되어 있는 Visual Studio Code 창으로 전환합니다.

    이전 연습 후에 Visual Studio Code를 닫은 경우 Visual Studio Code를 사용하여 랩 6 시작 폴더를 엽니다.

1. Visual Studio Code **보기** 메뉴에서 **터미널** 을 클릭합니다.

1. 명령 프롬프트의 현재 위치가 **ContainerDevice** 폴더인지 확인합니다.

1. 시뮬레이션된 디바이스 앱 실행을 시작하려면 다음 명령을 입력합니다.

    ```cmd/sh
    dotnet run
    ```

1. 디바이스에서 프로비전할 때 나열된 예외를 확인합니다.

    그룹 등록 및 등록된 디바이스가 삭제되었으므로 시뮬레이션된 디바이스는 더 이상 프로비전하거나 연결할 수 없습니다. 애플리케이션이 구성된 X.509 인증서를 사용하여 DPS에 연결하려고 하면 `ProvisioningTransportException` 오류 메시지가 반환됩니다.

    ```txt
    Found certificate: AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=sensor-thl-2000; PrivateKey: True
    Using certificate AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=sensor-thl-2000
    RegistrationID = sensor-thl-2000
    ProvisioningClient RegisterAsync . . . Unhandled exception. Microsoft.Azure.Devices.Provisioning.Client.ProvisioningTransportException: {"errorCode":401002,"trackingId":"df969401-c766-49a4-bab7-e769cd3cb585","message":"Unauthorized","timestampUtc":"2019-12-20T21:30:46.6730046Z"}
       at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.ValidateOutcome(Outcome outcome)
       at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.RegisterDeviceAsync(AmqpClientConnection client, String correlationId, DeviceRegistration deviceRegistration)
       at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.RegisterAsync(ProvisioningTransportRegisterMessage message, CancellationToken cancellationToken)
    ```

이 랩에서는 Device Provisioning Service를 통해 IoT 디바이스 수명 주기의 일부로 등록, 구성 및 프로비전 해제를 완료했습니다.
