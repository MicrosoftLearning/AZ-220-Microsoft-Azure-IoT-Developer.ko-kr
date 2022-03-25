---
lab:
  title: '랩 16: Azure IoT Hub를 통해 IoT 디바이스 관리 자동화'
  module: 'Module 8: Device Management'
ms.openlocfilehash: 3b752cc477c664f1c44b754c49b2e20542de1f72
ms.sourcegitcommit: eec2943250f1cd1ad2c5202ecbb9c37af71e8961
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/24/2022
ms.locfileid: "140872827"
---
# <a name="automate-iot-device-management-with-azure-iot-hub"></a>Azure IoT Hub를 통해 IoT 디바이스 관리 자동화

IoT 디바이스는 종종 최적화된 운영 체제를 사용하거나 실제 운영 체제가 없어도 실리콘에서 직접 코드를 실행합니다. 이와 같은 디바이스에서 실행되는 소프트웨어를 업데이트하기 위해 가장 흔히 사용하는 방법은 OS와 OS에서 실행 중인 앱(펌웨어라고 함)을 포함하여 전체 소프트웨어 패키지의 새 버전을 플래시하는 것입니다.

각 디바이스에는 특정 목적이 있으므로 펌웨어도 디바이스의 목적뿐만 아니라 사용 가능한 제한된 리소스에 맞게 매우 구체적이고 최적화되어 있습니다.

펌웨어 업데이트 프로세스도 하드웨어와 하드웨어 제조업체가 보드를 만든 방식에 따라 다를 수 있습니다. 이는 펌웨어 업데이트 프로세스의 일부가 일반적이지 않으며 펌웨어 업데이트 프로세스의 세부 정보를 얻으려면 디바이스 제조업체와 협력해야 함을 의미합니다(단, 자체 하드웨어를 개발하는 경우, 즉 펌웨어 업데이트 프로세스를 알고 있는 경우는 예외).

예전에는 펌웨어 업데이트를 개별 디바이스에 수동으로 적용하였지만, 이 방법은 일반적인 IoT 솔루션에 사용되는 디바이스 수를 고려할 때 더 이상 적합하지 않습니다. 요즘에는 펌웨어 업데이트가 무선(OTA)으로 진행되는 경우가 많으며, 이 경우 클라우드에서 원격으로 신규 펌웨어의 배포가 관리됩니다.

IoT 디바이스의 OTA 펌웨어 업데이트에는 다음과 같은 공통점이 있습니다.

1. 펌웨어 버전이 고유하게 식별됩니다.
1. 펌웨어는 이진 파일 형식으로 제공되며, 디바이스는 온라인 원본에서 이진 파일을 가져와야 합니다.
1. 펌웨어는 로컬로 저장되며 이는 일종의 실제 저장소(ROM 메모리, 하드 드라이브 등)입니다.
1. 디바이스 제조업체는 펌웨어를 업데이트하기 위해 디바이스에서 필요한 작업에 대한 설명을 제공합니다.

Azure IoT Hub는 단일 디바이스와 디바이스 컬렉션에서 장치 관리 작업을 구현할 수 있는 고급 지원을 제공합니다. [자동 장치 관리](https://docs.microsoft.com/azure/iot-hub/iot-hub-auto-device-config) 기능을 사용하면 간단하게 일련의 작업을 구성하고 트리거한 다음 진행 상황을 모니터링할 수 있습니다.

## <a name="lab-scenario"></a>랩 시나리오

Contoso의 치즈 동굴에서 구현한 자동 공기 처리 시스템은 회사를 고품질 전문점으로 만드는 데 도움이 되었습니다. 회사는 그 어느 때보다 많은 상을 수상한 치즈를 보유하고 있습니다.

기본 솔루션은 센서와 통합된 IoT 디바이스 및 방이 여러 개 있는 동굴 시스템의 온도와 습도를 실시간으로 제어하는 실내 온도 제어 시스템으로 구성됩니다. 또한 직접 메서드와 디바이스 쌍 속성을 모두 사용하여 디바이스를 관리하는 기능을 보여주는 간단한 백 엔드 앱을 개발했습니다.

Contoso는 운영자가 동굴 환경을 모니터링하고 원격으로 관리하는 데 사용할 수 있는 온라인 포털을 포함하도록 초기 솔루션에서 간단한 백 엔드 앱을 확장했습니다. 새로운 포털을 통해 작업자는 치즈 종류나 치즈 숙성 과정의 특정 단계에 따라 동굴의 온도와 습도를 사용자 정의할 수 있습니다. 동굴의 각 방이나 구역은 별도로 제어할 수 있습니다.

IT 부서는 운영자를 위해 개발한 백 엔드 포털을 유지 관리하지만, 관리자는 솔루션의 디바이스 쪽을 관리하기로 동의했습니다.

여러분에게 이것은 다음 두 가지를 의미합니다.

1. Contoso의 운영 팀은 항상 개선 방법을 찾고 있습니다. 이러한 개선은 종종 디바이스 소프트웨어의 새 기능에 대한 요청으로 이어집니다.

1. 동굴 위치에 배포되는 IoT 디바이스에는 개인 정보를 보호하고 해커가 시스템을 제어하지 못하도록 막는 최신 보안 패치가 필요합니다. 시스템 보안을 유지하려면, 펌웨어를 원격으로 업데이트하여 디바이스를 최신 상태로 유지해야 합니다.

자동 디바이스 관리 및 대규모 디바이스 관리를 가능하게 하는 IoT Hub 기능을 구현할 예정입니다.

다음 리소스가 만들어집니다.

![랩 16 아키텍처](media/LAB_AK_16-architecture.png)

## <a name="in-this-lab"></a>랩 내용

이 랩에서는 다음 활동을 완료할 예정입니다.

* 랩 필수 구성 요소(필수 Azure 리소스) 구성
* 펌웨어 업데이트를 구현할 시뮬레이션된 디바이스에 대한 코드 쓰기
* Azure IoT Hub 자동 디바이스 관리를 사용하여 단일 디바이스에서 펌웨어 업데이트 프로세스 테스트

## <a name="lab-instructions"></a>랩 지침

### <a name="exercise-1-configure-lab-prerequisites"></a>연습 1: 랩 필수 구성 요소 구성

이 랩에서는 다음과 같은 Azure 리소스를 사용할 수 있다고 가정합니다.

| 리소스 종류 | 리소스 이름 |
| :-- | :-- |
| 리소스 그룹 | rg-az220 |
| IoT Hub | iot-az220-training-{사용자 ID} |
| IoT 디바이스 | sensor-th-0155 |

이러한 리소스를 사용할 수 있게 하려면 다음 단계를 완료합니다.

1. 가상 머신 환경에서 Microsoft Edge 브라우저 창을 열고 다음 웹 주소로 이동합니다.
 
    +++https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoftLearning%2FAZ-220-Microsoft-Azure-IoT-Developer%2Fmaster%2FAllfiles%2FARM%2Flab16.json+++

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
    * deviceConnectionString
    * devicePrimaryKey

이제 리소스가 만들어졌습니다.

### <a name="exercise-2-examine-code-for-a-simulated-device-that-implements-firmware-update"></a>연습 2: 펌웨어 업데이트를 구현하는 시뮬레이션된 디바이스용 코드 검토

이 연습에서는 디바이스 쌍에 필요한 속성 변경을 관리하고 펌웨어 업데이트를 시뮬레이션하는 로컬 프로세스를 트리거할 시뮬레이션된 디바이스를 검토합니다. 펌웨어 업데이트 시작을 위해 구현하는 프로세스는 실제 디바이스에서 펌웨어 업데이트에 사용되는 프로세스와 비슷합니다. 여기서는 새 펌웨어 버전 다운로드, 펌웨어 업데이트 설치, 디바이스 다시 시작 프로세스를 시뮬레이트합니다.

또한 Azure Portal에서 디바이스 쌍 속성을 사용해 펌웨어 업데이트를 구성하고 실행합니다. 구체적으로는 구성 변경 요청을 디바이스로 전송하고 진행 상황을 모니터링하도록 디바이스 쌍 속성을 구성합니다.

#### <a name="task-1-examine-the-device-simulator-app"></a>작업 1: 디바이스 시뮬레이터 앱 검토

이 작업에서는 Visual Studio Code를 사용하여 콘솔 앱을 검토합니다.

1. **Visual Studio Code** 를 사용하여 ASP.NET 5 API 앱을 만드는 방법을 보여줍니다.

1. **파일** 메뉴에서 **폴더 열기** 를 클릭합니다.

1. 폴더 열기 대화 상자에서 랩 16 Starter 폴더로 이동합니다.

    랩 3: 개발 환경 설정에서 ZIP 파일을 다운로드하고 콘텐츠를 로컬로 추출하여 랩 리소스를 포함하는 GitHub 리포지토리를 복제했습니다. 추출된 폴더 구조에는 다음 폴더 경로가 포함됩니다.

    * Allfiles
      * 랩
          * 16-Azure IoT Hub를 통한 IoT 장치 관리 자동화
            * Starter
              * FWUpdateDevice

1. **FWUpdateDevice** 를 클릭하고 **폴더 선택** 을 클릭합니다.

    Visual Studio Code의 EXPLORER 창에 다음 파일이 나열되어야 합니다.

    * FWUpdateDevice.csproj
    * Program.cs

1. **탐색기** 창에서 **FWUpdateDevice.csproj** 파일을 클릭하여 열고 참조된 NuGet 패키지를 확인합니다.

    * Microsoft.Azure.Devices.Client - Azure IoT Hub용 디바이스 SDK
    * Microsoft.Azure.Devices.Shared - Azure IoT 디바이스 및 서비스 SDK의 일반적인 코드
    * Newtonsoft.Json - Json.NET은 .NET에 많이 사용되는 고성능 JSON 프레임워크입니다.

1. **탐색기** 창에서 **Program.cs** 를 클릭합니다.

#### <a name="task-2-review-the-application-code"></a>작업 2: 애플리케이션 코드 검토

이 작업에서는 IoT Hub가 생성한 요청에 대한 응답으로 디바이스에서 펌웨어 업데이트를 시뮬레이션하기 위한 코드를 검토합니다.

1. Visual Studio Code에서 **Program.cs** 파일을 열려 있는지 확인합니다.

1. `Global Variables` 주석을 찾습니다.

    이 간단한 예제에서는 디바이스 연결 문자열, 디바이스 ID 및 현재 펌웨어 버전이 추적됩니다.

1. 코드 편집기에서 다음 코드 줄을 찾습니다.

    ```csharp
    private readonly static string deviceConnectionString = "<your device connection string>";
    ```

1. **\<your device connection string\>** 을 이전에 저장한 디바이스 연결 문자열로 바꿉니다.

    이것이 필요한 유일한 코드 변경입니다.

1. **Main** 메서드를 찾습니다.

    이 메서드는 이전에 사용한 디바이스 시뮬레이터와 유사합니다. **deviceConnectionString** 은 IoT Hub 등에 연결할 **DeviceClient** 인스턴스를 만드는 데 사용되며 디바이스 쌍 속성 변경 콜백이 구성됩니다.

    **InitDevice** 메서드는 새로운 기능이며 단지 디바이스의 부팅 주기를 시뮬레이션하고 **UpdateFWUpdateStatus** 메서드를 통해 디바이스 쌍을 업데이트하여 현재 펌웨어를 보고합니다.

    그러면 앱이 루프를 실행하여 펌웨어 업데이트를 트리거하는 디바이스 쌍 업데이트를 대기합니다.

1. **UpdateFWUpdateStatus** 메서드를 찾고 코드를 검토합니다.

    This method creates a new **TwinCollection** instance, populates it with the provided values, and then updates the device twin.

1. **OnDesiredPropertyChanged** 메서드를 찾아 다음 코드를 검토합니다.

    이 메서드는 디바이스에서 디바이스 쌍 업데이트를 수신할 때 콜백으로 호출됩니다. 펌웨어 업데이트가 검색되면 **UpdateFirmware** 메서드가 호출됩니다. 이 메서드는 펌웨어 다운로드를 시뮬레이션하고 펌웨어를 업데이트한 다음 디바이스를 다시 부팅합니다.

### <a name="exercise-3-test-firmware-update-on-a-single-device"></a>연습 3: 단일 디바이스에서 펌웨어 업데이트 테스트

이 연습에서는 Azure Portal을 사용하여 새 장치 관리 구성을 만들고 이를 단일 시뮬레이션된 디바이스에 적용합니다.

#### <a name="task-1-start-device-simulator"></a>작업 1: 디바이스 시뮬레이터 시작

1. 필요한 경우 Visual Studio Code에서 **FWUpdateDevice** 프로젝트를 엽니다.

1. 터미널 창을 열어야 합니다.

    명령 프롬프트의 폴더 위치는 `FWUpdateDevice` 폴더입니다.

1. `FWUpdateDevice` 앱을 실행하려면 다음 명령을 입력합니다.

    ``` bash
    dotnet run "<device connection string>"
    ```

    > **참고**: 자리 표시자를 실제 디바이스 연결 문자열로 바꾸고 연결 문자열 주위에 "" 기호를 포함해야 합니다.
    >
    > `"HostName=iot-az220-training-{your-id}.azure-devices.net;DeviceId=sensor-th-0155;SharedAccessKey={}="`

1. 터미널 창의 내용을 검토합니다.

    터미널에 다음 출력이 표시되어야 합니다.

    ``` bash
        sensor-th-0155: Device booted
        sensor-th-0155: Current firmware version: 1.0.0
    ```

#### <a name="task-2-create-the-device-management-configuration"></a>작업 2: 디바이스 관리 구성 만들기

1. 필요한 경우 Azure 계정 자격 증명을 사용하여 [Azure Portal](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true)에 로그인합니다.

    Azure 계정이 두 개 이상인 경우 이 과정에 사용할 구독에 연결된 계정으로 로그인해야 합니다.

1. Azure Portal 대시보드에서 **iot-az220-training-{사용자 ID}** 를 클릭합니다.

    이제 IoT Hub 블레이드가 표시되어야 합니다.

1. 왼쪽 탐색 메뉴의 **디바이스 관리** 에서 **구성** 을 클릭합니다.

1. **IoT 디바이스 구성** 창에서 **+ 디바이스 구성 추가** 를 클릭합니다.

1. **디바이스 쌍 구성 만들기** 블레이드에서 **이름** 에 **firmwareupdate** 를 입력합니다.

    **레이블** 아래가 아니라 구성의 필수 **이름** 필드에 `firmwareupdate`를 입력해야 합니다.

1. 블레이드 하단에서 **다음: 트윈 설정 >** 을 클릭합니다.

1. **디바이스 쌍 설정** 에서 **디바이스 쌍 속성** 필드에 **properties.desired.firmware** 를 입력합니다.

1. **디바이스 쌍 속성 콘텐츠** 필드에서 다음을 입력합니다.

    ``` json
    {
        "fwVersion":"1.0.1",
        "fwPackageURI":"https://MyPackage.uri",
        "fwPackageCheckValue":"1234"
    }
    ```

1. 블레이드 하단에서 **다음: 메트릭 >** 을 클릭합니다.

    사용자 지정 메트릭을 사용하여 펌웨어 업데이트가 효과적인지 추적합니다.

1. **메트릭** 탭에서 **메트릭 이름** 아래에 **fwupdated** 를 입력합니다.

1. **메트릭 기준** 아래에 다음을 입력합니다.

    ``` SQL
    SELECT deviceId FROM devices
        WHERE properties.reported.firmware.currentFwVersion='1.0.1'
    ```

1. 블레이드 하단에서 **다음: 대상 디바이스 >** 를 클릭합니다.

1. **대상 디바이스** 탭에서 **우선 순위** 아래 **우선 순위(더 높은 값 ...)** 필드에 **10** 을 입력합니다.

1. **대상 조건** 아래 **대상 조건** 필드에 다음 쿼리를 입력합니다.

    ``` SQL
    deviceId='<your device id>'
    ```

    > **참고**: `'<your device id>'`를 디바이스를 만드는 데 사용한 디바이스 ID로 바꿔야 합니다. `'sensor-th-0155'`

1. 블레이드 하단에서 **다음: 검토 + 만들기 >** 를 클릭합니다.

    **검토 + 만들기** 탭이 열리면 새 구성에 대한 "유효성 검사 통과" 메시지가 표시되어야 합니다.

1. **검토 + 만들기** 탭에서 "유효성 검사 통과" 메시지가 표시되면 **만들기** 를 클릭합니다.

    "유효성 검사 통과" 메시지가 표시되면 되돌아가서 작업을 확인해야 구성을 만들 수 있습니다.

1. **IoT 디바이스 구성** 창에서 **구성 이름** 아래에 새 **firmwareupdate** 구성이 나열되어 있는지 확인합니다.

    새 구성이 만들어지면 IoT Hub는 구성의 대상 디바이스 기준과 일치하는 디바이스를 찾고 펌웨어 업데이트 구성을 자동으로 적용합니다.

1. Visual Studio Code 창으로 전환하고 터미널 창의 내용을 검토합니다.

    터미널 창에는 트리거된 펌웨어 업데이트 프로세스의 진행률을 나열하는 앱에서 생성된 새 출력이 포함되어야 합니다.

1. 시뮬레이션된 앱을 중지하고 Visual Studio Code를 닫습니다.

    터미널에서 "Enter" 키를 누르기만 하면 디바이스 시뮬레이터를 중지할 수 있습니다.
