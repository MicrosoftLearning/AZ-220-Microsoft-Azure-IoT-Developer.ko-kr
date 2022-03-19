---
lab:
  title: '랩 19: ADT(Azure Digital Twins) 솔루션 개발'
  module: 'Module 11: Develop with Azure Digital Twins'
ms.openlocfilehash: 654a98a8b7affffb99e1c88a6c1c9c5a32546488
ms.sourcegitcommit: 7874419a1f0f346f972914893b4b3644ba84a267
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/04/2022
ms.locfileid: "139262462"
---
# <a name="develop-azure-digital-twins-adt-solutions"></a>ADT(Azure Digital Twins) 솔루션 개발

## <a name="lab-scenario"></a>랩 시나리오

디지털 혁신의 차기 단계 진행을 결정한 Contoso Management에서는 ADT(Azure Digital Twins)를 사용하여 치즈 생산 시설의 모델을 개발하기로 했습니다. Azure Digital Twins를 사용하면 실제 환경의 라이브 모델을 만들어 해당 모델과 상호 작용할 수 있습니다. 먼저 개별 요소를 디지털 트윈으로 모델링합니다. 그런 다음, 라이브 이벤트에 응답하고 정보를 쿼리할 수 있는 기술 자료 그래프에 이러한 모델을 연결합니다.

여러분은 ADT를 가장 효율적으로 활용하는 방법을 자세히 파악하기 위한 개념 증명 프로토타입 제작 업무를 맡게 되었습니다. 이 프로토타입은 기존 Cheese Cave Device 센서 원격 정보를 다음과 같은 간단한 모델 계층 구조에 통합할 수 있는 방법을 제시해야 합니다.

* Cheese Factory
* Cheese Cave
* Cheese Cave Device

이 1차 프로토타입에서 다음 시나리오에 사용 가능한 솔루션을 제시하라는 요청을 받았습니다.

* 디바이스 원격 분석을 IoT Hub에서 ADT의 적절한 디바이스로 매핑하는 방법
* 자식 디지털 트윈 속성을 업데이트하여 부모 트윈 속성을 업데이트하는 방법(Cheese Cave Device 속성을 업데이트하여 Cheese Cave 속성 업데이트)
* ADT를 통해 Time Series Insights로 디바이스 원격 분석을 라우팅할 수 있는 방법

다음 리소스가 만들어집니다.

![랩 19 아키텍처](media/LAB_AK_19-architecture.png)

## <a name="in-this-lab"></a>랩 내용

이 랩에서는 다음 활동을 완료할 예정입니다.

* 랩 필수 구성 요소(필수 Azure 리소스) 구성
* 디지털 트윈 만들기 및 구성
  * 제공된 DTDL을 사용하여 디지털 트윈 만들기
  * 디지털 트윈 인스턴스를 사용하여 ADT 그래프 작성
* ADT 그래프 상호 작용 구현(ADT Explorer)
  * ADT 그래프 쿼리
  * 그래프에서 ADT 엔터티의 속성 업데이트
* 업스트림 및 다운스트림 시스템과 ADT 통합
  * IoT 디바이스 메시지를 수집하여 ADT로 변환
  * 원격 분석을 TSI(Time Series Insights)에 게시하도록 ADT 경로 및 엔드포인트 구성

## <a name="lab-instructions"></a>랩 지침

### <a name="exercise-1---configure-lab-prerequisites"></a>연습 1 - 랩 필수 구성 요소 구성

#### <a name="task-1---create-resources"></a>작업 1 – 리소스 만들기

이 랩은 다음 Azure 리소스를 사용할 수 있다고 가정합니다.

| 리소스 종류  | 리소스 이름                |
| :------------- | :--------------------------- |
| 리소스 그룹 | rg-az220                     |
| IoT Hub        | iot-az220-training-{사용자 ID} |
| TSI            | tsi-az220-training-{사용자 ID} |
| TSI 액세스 정책 | access1                   |

이러한 리소스를 사용할 수 있게 하려면 다음 단계를 완료합니다.

1. 가상 머신 환경에서 Microsoft Edge 브라우저 창을 열고 다음 웹 주소로 이동합니다.
 
    +++https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoftLearning%2FAZ-220-Microsoft-Azure-IoT-Developer%2Fmaster%2FAllfiles%2FARM%2Flab19.json+++

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

1. 현재 사용자 개체 ID를 확인하려면 **Cloud Shell** 을 열고 다음 명령을 실행합니다.

    ```sh
    az ad signed-in-user show --query objectId -o tsv
    ```

    표시된 개체 ID를 복사합니다.

1. **개체 ID** 필드에 위에서 복사한 개체 ID를 입력합니다.

1. 템플릿의 유효성을 검사하려면 **검토 및 만들기** 를 클릭합니다.

1. 유효성 검사를 통과하면 **만들기** 를 클릭합니다.

    배포가 시작됩니다.

1. 배포가 완료되면 왼쪽 탐색 영역에서 템플릿의 출력 값을 검토하려면 **출력** 을 클릭합니다.

    나중에 사용할 수 있도록 출력을 기록해 둡니다.

    * connectionString
    * deviceConnectionString
    * devicePrimaryKey
    * storageAccountName

이제 리소스가 만들어졌습니다.

#### <a name="task-2---verify-tools"></a>작업 2 – 도구 확인

1. 가상 머신 환경에서 Windows 명령 프롬프트 창을 엽니다.

1. 로컬에 설치된 Azure CLI 버전을 표시하려면 다음 명령을 입력합니다.

    ```powershell
    az --version
    ```

1. azure-cli 버전 2.11.0 이상이 출력되는지 확인합니다.

    Azure CLI 버전 2.11은 최신 버전으로 업그레이드하는 기능을 제공합니다. 업그레이드하려면 다음 명령을 입력합니다.

        ```powershell
        az update
        ```

    > **참고**: Azure CLI가 설치되어 있지 않으면 먼저 설치한 후에 진행해야 합니다.

    1. 브라우저를 연 다음 Azure CLI 도구 다운로드 페이지로 이동합니다. [Azure CLI 설치](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest "Azure CLI 설치")

        Azure CLI 도구의 최신 버전을 설치해야 합니다. 현재 버전의 Azure CLI(2022년 2월 기준)는 버전 2.33이지만 새 버전은 매월 릴리스되므로 최신 버전이 변경되었을 가능성이 있습니다.

    1. **Azure CLI 설치** 페이지에서 OS에 맞는 설치 옵션(예: **Windows에 설치**)을 선택한 다음 화면의 지침에 따라 Azure CLI 도구를 설치합니다.

### <a name="exercise-2---create-an-instance-of-the-azure-digital-twins-resource"></a>연습 2 - Azure Digital Twins 리소스의 인스턴스 만들기

이 연습에서는 Azure Portal을 사용하여 ADT(Azure Digital Twins) 인스턴스를 만듭니다. 그러면 Azure Digital Twins의 연결 데이터가 나중에 사용 가능하도록 텍스트 파일에 저장됩니다. 마지막으로, 현재 사용자에게 ADT 리소스 액세스를 허용하는 역할을 할당합니다.

#### <a name="task-1---use-the-azure-portal-to-create-a-resource-azure-digital-twins"></a>작업 1 - Azure Portal을 사용하여 리소스(Azure Digital Twins) 만들기

1. 새 브라우저 창에서 [Azure Portal](https://portal.azure.com)을 엽니다.

1. Azure Portal 메뉴에서 **+ 리소스 만들기** 를 클릭합니다.

    그러면 열리는 **새로 만들기** 블레이드는 Azure Marketplace의 프런트 엔드입니다. Azure Marketplace에서는 Azure에서 만들 수 있는 모든 리소스의 컬렉션이 제공됩니다. 마켓플레이스에는 Microsoft와 커뮤니티의 리소스가 포함되어 있습니다.

1. **Marketplace 검색** 텍스트 상자에 **Azure Digital Twins** 를 입력합니다.

1. 옵션이 표시되면 **Azure Digital Twins** 를 선택하고 **만들기** 를 클릭합니다.

1. **리소스 만들기** 창의 **구독** 에서 이 과정에 사용 중인 구독이 선택되어 있는지 확인합니다.

    > **참고**: 계정에 구독의 관리자 역할이 할당되어 있어야 합니다.

1. **리소스 그룹** 으로 **rg-az220** 를 선택합니다.

1. **리소스 이름** 으로 **adt-az220-training-{사용자 ID}** 를 입력합니다.

1. **지역** 드롭다운에서 Azure IoT Hub가 프로비저닝된 지역 또는 사용 가능한 가장 가까운 지역을 선택합니다.

1. 현재 사용자가 **Digital Twins Explorer** 앱을 사용할 수 있도록 **리소스에 액세스 권한 부여** 에서 **Azure Digital Twins 소유자 역할 할당** 을 선택합니다.

    > **참고**: 인스턴스 내의 요소를 관리하려면 사용자가 Azure Digital Twins 데이터 평면 API에 액세스할 수 있어야 합니다. 위에서 제안된 역할을 선택하면 현재 사용자에게 데이터 평면 API에 대한 모든 권한이 부여됩니다. 나중에 IAM(Access Control)을 사용하여 적절한 역할을 선택할 수도 있습니다. Azure Digital Twins 보안에 대한 자세한 내용은 [여기](https://docs.microsoft.com/azure/digital-twins/concepts-security)에서 확인할 수 있습니다.

1. 입력한 값을 검토하려면 **검토 + 만들기** 를 클릭합니다.

1. 배포 프로세스를 시작하려면 **만들기** 를 클릭합니다.

    **배포 진행 중** 이 표시되는 동안 잠시 기다립니다.

1. **리소스로 이동** 을 선택합니다.

    ADT 리소스의 개요 창이 표시됩니다. 이 창에는 제목이 **Azure Digital Twins 시작** 인 본문 섹션이 포함되어 있습니다.

#### <a name="task-2---save-the-connection-data-to-a-reference-file"></a>작업 2 - 참조 파일에 연결 데이터 저장

1. **메모장** 또는 유사한 텍스트 편집기를 사용해 **adt-connection.txt** 파일을 만듭니다.

1. 이 파일에 Azure Digital Twins 인스턴스의 이름인 **adt-az220-training-{사용자 ID}** 를 추가합니다.

1. 이 파일에 리소스 그룹인 **rg-az220** 을 추가합니다.

1. 브라우저에서 Digital Twins 인스턴스 **개요** 창으로 돌아옵니다.

1. 개요 창의 **필수** 섹션에서 **호스트 이름** 필드를 찾습니다.

1. **호스트 이름** 필드를 마우스 포인터로 가리키고 값 오른쪽에 나타나는 아이콘을 사용하여 호스트 이름을 클립보드에 복사한 후 텍스트 파일에 붙여넣습니다.

1. 텍스트 파일에서 호스트 이름 시작 부분에 **https://** 를 추가하여 호스트 이름을 Digital Twins 인스턴스의 연결 URL로 변환합니다.

    수정된 URL은 다음과 같습니다.

    ```http
    https://adt-az220-training-dm030821.api.eus.digitaltwins.azure.net
    ```

1. **adt-connection.txt** 파일을 저장합니다.

Azure Digital Twin 리소스가 작성되었고 사용자 계정이 업데이트되었으므로, API를 통해 리소스에 액세스할 수 있습니다.

### <a name="exercise-3---create-a-graph-of-the-models"></a>연습 3 - 모델의 그래프 만들기

분석가는 모델링 작업의 일환으로 Cheese Cave Device 메시지 콘텐츠와 같은 여러 요소를 고려하고 DTDL **속성** 및 **원격 분석** 필드 정의에 매핑을 만듭니다. 이러한 DTDL 코드 조각을 사용하려면 **인터페이스**(모델의 최상위 코드 항목)에 통합해야 합니다. 그러나 Cheese Cave Device 모델의 **인터페이스** 는 Contoso Cheese Factory용 Azure Digital Twins 환경의 작은 일부분일 뿐입니다. 공장 전체를 나타내는 환경을 모델링하는 것은 본 과정의 범위를 벗어나기 때문에 여기서는 Cheese Cave Device 모델, 연결된 Cheese Cave 모델 및 Factory 모델을 중심으로 하는 크게 간소화된 환경을 살펴봅니다. 모델 계층 구조는 다음과 같습니다.

* Cheese Factory 인터페이스
* Cheese Cave 인터페이스
* Cheese Cave Device 인터페이스

위의 인터페이스 정의 계층 구조와 정의 간의 관계를 고려하면 **Cheese Factory에는 Cheese Cave가 포함** 되어 있고 **Cheese Cave에는 Cheese Cave Device가 포함** 되어 있다고 할 수 있습니다.

ADT 환경용 디지털 트윈 모델을 디자인할 때는 일관된 방식을 사용하여 인터페이스, 스키마, 관계에 사용되는 ID를 작성하는 것이 가장 좋습니다. 환경 내의 각 엔터티에는 **@id** 속성(인터페이스의 필수 속성)이 포함됩니다. 이 속성은 해당 엔터티를 고유하게 식별해야 합니다. ID 값의 형식은 **DTMI(디지털 트윈 모델 식별자)** 의 형식과 동일합니다. DTMI의 세 가지 구성 요소는 scheme, path, version입니다. scheme과 path는 콜론 `:`으로 구분하고 path와 version은 세미콜론 `;`으로 구분합니다. 형식은 다음과 같습니다. `<scheme> : <path> ; <version>` DTMI 형식 ID 내의 scheme 값은 항상 **dtmi** 입니다.

Contoso Cheese Factory에서 사용되는 ID 값의 예로 `dtmi:com:contoso:digital_factory:cheese_factory;1`을 들 수 있습니다.

이 예에서 scheme 값은 필수 형식인 **dtmi** 이며, 버전은 **1** 로 설정되어 있습니다. 이 ID 값에서 `<path>` 구성 요소는 다음 분류법을 사용합니다.

* 모델의 원본 - **com:contoso**
* 모델 범주 - **digital_factory**
* 범주 내의 유형 - **cheese_factory**

> **팁**: DTMI 형식에 대해 자세히 알아보려면 다음 리소스를 검토하세요.
> * [디지털 트윈 모델 식별자](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#digital-twin-model-identifier)

위에서 설명한 모델 계층 구조와 관계를 적용할 때 사용 가능한 ID의 예는 다음과 같습니다.

| 인터페이스                      | ID                                                                     |
| :----------------------------- | :--------------------------------------------------------------------- |
| Cheese Factory 인터페이스       | dtmi:com:contoso:digital_factory:cheese_factory;1                      |
| Cheese Cave 인터페이스        | dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1        |
| Cheese Cave Device 인터페이스 | dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave_device;1 |

그리고 ID 간게는 다음과 같은 관계가 설정될 수 있습니다.

| 관계 | ID                                                                | 관계 시작 ID                                                         | 관계 종료 ID                                                                  |
| :----------- | :---------------------------------------------------------------- | :-------------------------------------------------------------- | :--------------------------------------------------------------------- |
| Cave 포함  | dtmi:com:contoso:digital_factory:cheese_factory:rel_has_caves;1 | dtmi:com:contoso:digital_factory:cheese_factory;1               | dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1        |
| Device 포함  | dtmi:com:contoso:digital_factory:cheese_cave:rel_has_devices;1  | dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1 | dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave_device;1 |

> **참고**: 랩 3: 개발 환경 설정에서 ZIP 파일을 다운로드하고 콘텐츠를 로컬로 추출하여 랩 리소스를 포함하는 GitHub 리포지토리를 복제했습니다. 추출된 폴더 구조에는 다음 폴더 경로가 포함됩니다.
>
> * Allfiles
>   * 랩
>       * 19-Azure Digital Twins
>           * 최종
>               * 모델
>
>  이 연습에서 참조한 전체 모델이 이 폴더 위치에서 제공됩니다.

이 연습에서는 개념 증명에서 사용할 각 디지털 트윈의 인터페이스를 정의했습니다. 이번에는 디지털 트윈의 실제 그래프를 생성해 보겠습니다. 그래프를 작성하는 과정은 단순합니다.

* 모델 정의 가져오기
* 적절한 모델에서 트윈 인스턴스 만들기
* 정의된 모델 관계를 사용하여 트윈 인스턴스 간의 관계 만들기

이 과정은 다양한 방식으로 진행할 수 있습니다.

* 명령줄이나 스크립트에서 Azure CLI 명령 사용
* 프로그래밍 방식으로(SDK 중 하나를 사용하거나 REST API를 통해 직접 명령 실행)
* ADT Explorer 샘플 등의 도구 사용

ADT 그래프의 다양한 시각화가 포함되어 있는 ADT Explorer는 개념 증명용으로 간단한 그래프를 작성할 때 유용하게 활용할 수 있습니다. 그러나 더욱 복잡한 대형 모델도 지원됩니다. 그리고 포괄적인 대량 가져오기/내보내기 기능을 활용하면 반복 디자인 작업을 쉽게 수행할 수 있습니다. 이 연습에서는 다음 작업을 완료합니다.

* Azure Portal을 통해 ADT Explorer(미리 보기)에 액세스
* Contoso Cheese 모델 가져오기
* 모델을 사용하여 디지털 트윈 만들기
* 그래프에 관계 추가
* ADT에서 트윈, 관계, 모델을 사용하고 삭제하는 방법 알아보기
* ADT로 그래프 대량 가져오기

#### <a name="task-1---access-the-adt-explorer"></a>작업 1 - ADT Explorer에 액세스

**ADT Explorer** 는 Azure Digital Twins 서비스용 애플리케이션입니다. Azure Digital Twins 인스턴스에 연결되는 이 앱은 다음 기능을 제공합니다.

* 모델 업로드 및 탐색
* 트윈 그래프 업로드 및 편집
* 여러 레이아웃 기법을 사용하여 트윈 그래프 시각화
* 트윈 속성 편집
* 트윈 그래프에 대해 쿼리 실행

ADT Explorer는 Azure Portal에 미리 보기 기능으로 통합되어 있으며, 독립 실행형 샘플 애플리케이션으로도 사용할 수 있습니다. 이 랩에서는 Azure Portal에 통합된 버전을 사용합니다.

1. 새 브라우저 창에서 [Azure Portal](https://portal.azure.com)을 엽니다.

1. 브라우저에서 Digital Twins 인스턴스 **개요** 창으로 이동합니다.

1. 새 브라우저 탭에서 ADT Explorer를 열려면 **Azure Digital Twins Explorer(미리 보기) 열기** 를 클릭합니다.

    ADT Explorer를 호스트하는 새 브라우저 탭이 열립니다. 결과를 찾을 수 없다는 경고가 표시됩니다. 아직 가져온 모델이 없기 때문에 이 경고는 예상되는 것입니다.

    **참고**: 새 창에서 ADT 인스턴스의 URL을 입력하라는 메시지가 표시되면 텍스트 편집기에 저장해 둔 URL 값을 입력합니다.

    > **중요**: 로그인하라는 메시지가 표시되면 Azure Digital Twins 인스턴스를 만들 때 사용한 것과 동일한 계정을 사용해야 합니다. 그러지 않으면 데이터 평면 API에 액세스할 수 없으며 오류가 표시됩니다.


이제 **ADT Explorer** 샘플 애플리케이션을 사용할 수 있습니다. 다음 작업에서는 모델을 로드합니다. 사용 가능한 모델이 없다는 오류 메시지가 표시되어도 놀라지 마세요.

#### <a name="task-2---import-models"></a>작업 2 – 모델 가져오기

ADT에서 디지털 트윈을 만들려면 먼저 모델을 업로드해야 합니다. 다음과 같은 여러 가지 방식으로 모델을 업로드할 수 있습니다.

* [데이터 평면 SDK](https://docs.microsoft.com/azure/digital-twins/how-to-use-apis-sdks)
* [데이터 평면 REST API](https://docs.microsoft.com/rest/api/azure-digitaltwins/)
* [Azure CLI](https://docs.microsoft.com/cli/azure/ext/azure-iot/dt?view=azure-cli-latest)
* [ADT Explorer](https://docs.microsoft.com/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/)의 가져오기 기능

처음 두 옵션은 프로그래밍 방식 시나리오에 적합하며, Azure CLI는 **CaC(Configuration as Code)** 시나리오 또는 한 번만 충족하면 되는 요구 사항 적용 시에 유용할 수 있습니다. **ADT Explorer** 앱을 사용하면 ADT와 직관적인 방식으로 상호 작용할 수 있습니다.

> **팁**: **코드로 구성** 은 무엇일까요? 구성은 소스 코드(예: Azure CLI 명령을 포함하는 스크립트)로 작성되므로 개발 모범 사례(예: 모델 업로드의 재사용 가능한 정의 만들기, 매개 변수화, 루프를 사용하여 모델의 여러 인스턴스 만들기)를 활용하여 구성을 최적화할 수 있습니다. 이러한 스크립트는 소스 코드 컨트롤에 저장하여 보존하고 버전을 제어할 수 있습니다.

이 작업에서는 Azure CLI 명령과 ADT Explorer 샘플 앱을 사용하여 **Allfiles\Labs\19-Azure Digital Twins\Final\Models** 폴더에 포함된 모델을 업로드합니다.

1. 명령 프롬프트 창을 엽니다.

1. 올바른 Azure 계정 자격 증명을 사용 중인지 확인하려면 다음 명령을 사용하여 Azure에 로그인합니다.

    ```powershell
    az login
    ```

1. **Cheese Factory 인터페이스** 를 업로드하려면 다음 명령을 입력합니다.

    ```powershell
    az dt model create --models "{file-root}\Allfiles\Labs\19-Azure Digital Twins\Final\Models\CheeseFactoryInterface.json" -n adt-az220-training-{your-id}
    ```

    여기서 **{파일 루트}** 는 이 랩용 도우미 파일이 포함된 폴더로 바꾸고 **{사용자 ID}** 는 사용자의 고유 식별자로 바꿔야 합니다.

    명령 실행이 정상적으로 완료되면 다음과 같은 출력이 표시됩니다.

    ```json
    [
        {
            "decommissioned": false,
            "description": {},
            "displayName": {
            "en": "Cheese Factory - Interface Model"
            },
            "id": "dtmi:com:contoso:digital_factory:cheese_factory;1",
            "uploadTime": "2021-03-24T19:56:53.8723857+00:00"
        }
    ]
    ```

    **참고**: Azure CLI 명령을 구성할 수 없는 경우 아래 지침에서는 Azure Digital Twins Explorer 인터페이스를 사용하여 모델을 가져오는 방법을 보여 줍니다.
  
1. **ADT Explorer** 로 돌아갑니다.

    > **팁**: **MODELS** 탐색기에서 **새로 고침** 단추를 클릭하여 모델 목록을 업데이트합니다.

    업로드한 **Cheese Factory - Interface Model** 이 표시됩니다.

    ![Factory Model이 표시된 ADT Explorer MODEL VIEW](media/LAB_AK_19-modelview-factory.png)

1. **ADT Explorer** 를 사용하여 나머지 두 모델을 가져오려면 **MODELS** 탐색기에서 **모델 업로드** 아이콘을 클릭합니다.

    ![ADT Explorer MODEL VIEW의 Upload a Model 단추](media/LAB_AK_19-modelview-addmodel.png)

1. **Open** 대화 상자에서 **Models** 폴더로 이동하여 **CheeseCaveInterface.json** 및 **CheeseCaveDeviceInterface.json** 파일을 선택하고 **Open** 을 클릭합니다.

    두 파일이 ADT에 업로드되고 모델이 추가됩니다. 완료되면 **MODELS** 탐색기가 업데이트되고 3개의 모델이 모두 표시됩니다.

모델을 업로드했으므로 이제 디지털 트윈을 만들 수 있습니다.

#### <a name="task-3---creating-twins"></a>작업 3 – 트윈 만들기

Azure Digital Twins 솔루션에서 환경의 엔터티는 디지털 트윈으로 표시됩니다. 디지털 트윈은 사용자 정의 모델중 하나의 인스턴스입니다. 이는 관계를 통해 다른 디지털 트윈에 연결하여 트윈 그래프를 형성할 수 있습니다. 이 트윈 그래프는 전체 환경에 대한 표현입니다.

모델과 마찬가지로 디지털 트윈과 관계도 다음과 같은 여러 가지 방식으로 만들 수 있습니다.

* [데이터 평면 SDK](https://docs.microsoft.com/azure/digital-twins/how-to-use-apis-sdks)
* [데이터 평면 REST API](https://docs.microsoft.com/rest/api/azure-digitaltwins/)
* [Azure CLI](https://docs.microsoft.com/cli/azure/ext/azure-iot/dt?view=azure-cli-latest)
* [ADT Explorer](https://docs.microsoft.com/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/)의 가져오기 기능

앞에서 설명한 것처럼, 처음 두 옵션은 프로그래밍 방식 시나리오에 적합하며 Azure CLI는 **CaC(Configuration as Code)** 시나리오 또는 한 번만 충족하면 되는 요구 사항 적용 시에 유용할 수 있습니다. 디지털 트윈과 관계를 만드는 가장 직관적인 방식은 **ADT Explorer** 를 사용하는 것입니다. 그러나 ADT Explorer 사용 시에는 속성 초기화가 다소 제한됩니다.

1. CheeseFactoryInterface 모델을 업로드하는 데 사용한 명령줄 창을 엽니다.

1. Azure CLI를 사용하여 Cheese Factory 모델에서 디지털 트윈을 만들려면 다음 명령을 입력합니다.

    ```powershell
    az dt twin create --dt-name adt-az220-training-{your-id} --dtmi "dtmi:com:contoso:digital_factory:cheese_factory;1" --twin-id factory_1 --properties "{file-root}\Allfiles\Labs\19-Azure Digital Twins\Final\Properties\FactoryProperties.json"
    ```

    여기서 **{파일 루트}** 는 이 랩용 도우미 파일이 포함된 폴더로 바꾸고 **{사용자 ID}** 는 사용자의 고유 식별자로 바꿔야 합니다.

    다음을 확인합니다.

    * **--dt-name** 값은 ADT 트윈 인스턴스를 지정합니다.
    * **--dtmi** 값은 앞에서 업로드한 Cheese Factory 모델을 지정합니다.
    * **--twin-id** 는 디지털 트윈에 지정되는 ID를 지정합니다.
    * **--properties** 값은 트윈을 초기화하는 데 사용할 JSON 문서의 파일 경로를 제공합니다. 간단한 JSON을 인라인으로 지정할 수도 있습니다.

    명령 실행이 정상적으로 완료되면 다음과 같은 명령 출력이 표시됩니다.

    ```json
    {
        "$dtId": "factory_1",
        "$etag": "W/\"09e781e5-c31f-4bf1-aed4-52a4472b0c5b\"",
        "$metadata": {
            "$model": "dtmi:com:contoso:digital_factory:cheese_factory;1",
            "FactoryName": {
                "lastUpdateTime": "2021-03-24T21:51:04.1371421Z"
            },
            "GeoLocation": {
                "lastUpdateTime": "2021-03-24T21:51:04.1371421Z"
            }
        },
        "FactoryName": "Contoso Cheese 1",
        "GeoLocation": {
            "Latitude": 47.64319985218156,
            "Longitude": -122.12449651580214
        }
    }
    ```

    **$metadata** 속성에는 속성을 마지막으로 업데이트한 시간을 추적하는 개체가 포함되어 있습니다.

1. **FactoryProperties.json** 파일에 포함된 JSON은 다음과 같습니다.

    ```json
    {
        "FactoryName": "Contoso Cheese 1",
        "GeoLocation": {
            "Latitude": 47.64319985218156,
            "Longitude": -122.12449651580214
        }
    }
    ```

    속성 이름은 Cheese Factory 인터페이스에서 선언한 DTDL 속성 값과 일치합니다.

    > **참고**: 복합 속성 **GeoLocation** 은 **Latitude** 및 **Longitude** 속성이 포함된 JSON 개체를 통해 할당됩니다.

1. 브라우저에서 **ADT Explorer** 로 돌아갑니다.

1. 지금까지 작성한 디지털 트윈을 표시하려면 **Run Query** 를 클릭합니다.

    > **참고**: 쿼리와 쿼리 언어에 대해서는 잠시 후에 설명합니다.

    몇 분 정도 지나면 **TWIN GRAPH** 보기에 **factory_1** 디지털 트윈이 표시됩니다.

    ![ADT Explorer GRAPH VIEW Factory 1](media/LAB_AK_19-graphview-factory_1.png)

1. 디지털 트윈 속성을 보려면 **TWIN GRAPH** 보기에서 **factory_1** 을 클릭합니다.

    **factory_1** 의 속성이 **Property View** 에 트리 보기의 노드로 표시됩니다.

1. 경도 및 위도 속성 값을 확인하려면 **GeoLocation** 을 클릭합니다.

    이 두 값은 **FactoryProperties.json** 파일의 값과 일치합니다.

1. Cheese Factory 모델에서 다른 디지털 트윈을 만들려면 **MODELS** 탐색기에서 **Cheese Factory** 모델을 찾은 다음 **트윈 만들기** 를 클릭합니다.

    ![ADT Explorer MODEL VIEW Create a Twin 단추](media/LAB_AK_19-modelview-createtwin.png)

1. **New Twin Name** 을 입력하라는 메시지가 표시되면 **factory_2** 를 입력하고 **Save** 를 클릭합니다.

1. **factory_2** 의 디지털 트윈 속성을 보려면 **TWIN GRAPH** 보기에서 **factory_2** 를 클릭합니다.

    **FactoryName** 및 **GeoLocation** 속성은 초기화되어 있지 않습니다.

1. **factoryName** 을 설정하려면 속성 오른쪽에 마우스 커서를 놓습니다. 그러면 텍스트 상자 컨트롤이 나타납니다. **Cheese Factory 2** 를 입력합니다.

    ![ADT Explorer Property View factoryName 입력](media/LAB_AK_19-propertyexplorer-factoryname.png)

1. **PROPERTIES** Explorer 창에서 업데이트된 속성을 저장하려면 **트윈 패치** 아이콘을 선택합니다.

    > **참고**: Patch Twin 아이콘은 Run Query 단추 오른쪽에 있는 Save Query 아이콘과 같은 모양으로 표시됩니다. 하지만 Save Query 아이콘을 클릭하면 안 됩니다.

    Patch Twin을 선택하면 JSON Patch가 작성 및 전송되어 디지털 트윈이 업데이트됩니다. 그러면 대화 상자에 **Patch Information** 이 표시됩니다. 값을 처음으로 설정하는 것이므로 **op**(작업) 속성이 **추가** 된 것을 볼 수 있습니다. 이후에 값을 변경하면 작업이 **대체** 됩니다. 직접 살펴보려면 다른 업데이트를 진행하기 전에 **쿼리 실행** 을 클릭하여 **TWIN GRAPH** 보기를 새로 고칩니다.

   > **팁**: JSON Patch 문서에 대해 자세히 알아보려면 다음 리소스를 검토하세요.
   > * [JSON(Javascript Object Notation) 패치](https://tools.ietf.org/html/rfc6902)
   > * [JSON 패치란?](http://jsonpatch.com/)

1. **PROPERTIES** 탐색기에서 **factory_2** **GeoLocation** 속성을 살펴보면 **Latitude** 와 **Longitude** 의 값이 **Unset** 인 것을 알 수 있습니다.

    > **정보**: 이전 버전의 ADT Explorer는 UI를 통해 “하위 속성”을 편집하는 것을 지원하지 않았습니다. 최신 버전에서는 이 유용한 기능이 추가되었습니다.

1. 다음과 같이 **Latitude** 값과 **Longitude** 값을 업데이트합니다.

    | 속성 이름 | 값 |
    | :-- | :-- |
    | 위도 | 47.64530450740752 |
    | 경도 | -122.12594819866645 |

1. **PROPERTIES** 탐색기 창에서 업데이트된 속성을 저장하려면 **트윈 패치** 아이콘을 선택합니다.

    패치 정보가 다시 한번 표시되는 것을 볼 수 있습니다.

1. **MODELS** 탐색기에서 적절한 모델을 선택하고 **트윈 추가** 를 클릭하여 다음 디지털 트윈을 추가합니다.

    | 모델 이름                             | 디지털 트윈 이름 |
    | :------------------------------------- | :---------------- |
    | Cheese Cave - Interface Model        | cave_1          |
    | Cheese Cave - Interface Model        | cave_2          |
    | Cheese Cave Device - Interface Model | device_1          |
    | Cheese Cave Device - Interface Model | device_2          |

    ![작성한 트윈이 표시된 ADT Explorer GRAPH VIEW](media/LAB_AK_19-graphview-createdtwins.png)

트윈을 몇 개 만들었으므로 이번에는 관계를 추가해 보겠습니다.

#### <a name="task-4---adding-relationships"></a>작업 4 – 관계 추가

트윈은 해당 관계에 따라 트윈으로 연결됩니다. 트윈이 가질 수 있는 관계는 해당 모델의 일부로 정의됩니다.

예를 들어 **Cheese Factory** 모델은 **Cheese Cave** 유형 트윈을 대상으로 하는 "contains" 관계를 정의합니다. 이 정의를 사용하는 경우 Azure Digital Twins에서는 모든 **Cheese Factory** 트윈과 모든 **Cheese Cave** 트윈(특정 치즈에 사용되는 특수 **Cheese Cave** 와 같이 **Cheese Cave** 의 하위 유형에 해당되는 트윈 포함) 간에 **rel_has_caves** 관계를 작성할 수 있습니다.

이 프로세스의 결과는 그래프의 가장자리(해당 관계)를 통해 연결된 노드(디지털 트윈) 집합입니다.

모델 및 트윈과 마찬가지로 관계도 여러 가지 방식으로 만들 수 있습니다.

1. Azure CLI를 통해 관계를 만들려면 명령 프롬프트로 돌아와 다음 명령을 실행합니다.

    ```powershell
    az dt twin relationship create -n adt-az220-training-{your-id} --relationship-id factory_1_has_cave_1 --relationship rel_has_caves --twin-id factory_1 --target cave_1
    ```

    **{사용자 ID}** 를 사용자의 고유 식별자로 바꿔야 합니다.

    명령 실행이 정상적으로 완료되면 다음과 같은 명령 출력이 표시됩니다.

    ```json
    {
        "$etag": "W/\"cdb10516-36e7-4ec3-a154-c050afed3800\"",
        "$relationshipId": "factory_1_has_cave_1",
        "$relationshipName": "rel_has_caves",
        "$sourceId": "factory_1",
        "$targetId": "cave_1"
    }
    ```

1. 관계를 시각화하려면 브라우저에서 **ADT Explorer** 로 돌아갑니다.

1. 업데이트된 디지털 트윈을 표시하려면 **Run Query** 를 클릭합니다.

    다이어그램이 새로 고쳐지고 새 관계가 표시됩니다.

    ![관계가 표시된 ADT Explorer GRAPH VIEW](media/LAB_AK_19-graphview-relationship.png)

    관계가 표시되지 않으면 브라우저 창을 새로 고친 후에 쿼리를 실행합니다.

1. **ADT Explorer** 를 사용하여 관계를 추가하려면 먼저 **cave_1** 을 클릭하여 선택한 다음 **device_1** 을 **마우스 오른쪽 단추로 클릭** 합니다. 표시되는 바로 가기 메뉴에서 **관계 추가** 를 선택합니다.

1. **Create Relationship** 대화 상자의 **Source ID** 에 **cave_1** 이 표시되는지 확인합니다.

1. **Target ID** 에는 **device_1** 이 표시되는지 확인합니다.

1. **관계** 아래에서 **rel_has_devices** 를 선택합니다.

    > **참고**: Azure CLI를 사용하여 만든 관계와 달리 **$relationshipId** 값을 제공하는 UI가 없습니다. 대신 GUID가 할당됩니다.

1. 관계를 만들려면 **Save** 를 클릭합니다.

    관계가 작성되며 다이어그램이 업데이트되어 관계가 표시됩니다. 이제 다이어그램에 **factory_1** 에는 **cave_1** 이 포함되며 cave_1에는 **device_1** 이 포함됨을 나타내는 관계가 표시됩니다.

1. 관계를 2개 더 추가합니다.

    | 원본    | 대상   | 관계    |
    | :-------- | :------- | :-------------- |
    | factory_1 | cave_2 | rel_has_caves |
    | cave_2  | device_2 | rel_has_devices |

    그러면 그래프가 다음과 같이 표시됩니다.

    ![업데이트된 그래프가 표시된 ADT Explorer GRAPH VIEW](media/LAB_AK_19-graphview-updatedgraph.png)

1. **TWIN GRAPH** 보기의 레이아웃 옵션을 보려면 **레이아웃 선택** 단추를 클릭합니다.

    ![ADT Explorer 그래프 보기 레이아웃 선택](media/LAB_AK_19-twingraph-chooselayout.png)

    **TWIN GRAPH** 보기는 여러 알고리즘을 사용하여 그래프에 레이아웃을 적용합니다. 기본적으로는 **Klay** 레이아웃이 선택됩니다. 다른 레이아웃을 선택하여 그래프의 모양이 어떻게 바뀌는지를 확인할 수 있습니다.

#### <a name="task-5---deleting-models-relationships-and-twins"></a>작업 5 - 모델, 관계 및 트윈 삭제

ADT를 사용하는 모델링 과정의 디자인 프로세스에서는 개념 증명을 여러 개 만들 가능성이 높습니다. 이렇게 만든 개념 증명 중 대부분은 삭제됩니다. 디지털 트윈에 대해 수행하는 다른 작업과 마찬가지로 모델과 트윈 삭제 역시 프로그래밍 방식(API, SDK, CLI)으로 수행할 수도 있고 **ADT Explorer** 를 사용할 수도 있습니다.

> **참고**: 삭제 작업은 비동기식으로 수행됩니다. 따라서 REST API 호출이나 **ADT Explorer** 에서 수행하는 삭제 작업이 즉시 완료되는 것처럼 보여도 ADT 서비스 내에서 해당 작업이 완료되려면 몇 분 정도 걸릴 수 있습니다. 그러므로 백 엔드 작업이 완료될 때까지는 최근 삭제한 모델과 이름이 같은 수정된 모델 업로드 시도가 예기치 않게 실패할 수 있습니다.

1. CLI를 통해 **factory_2** 디지털 트윈을 삭제하려면 명령 프롬프트 창으로 돌아와 다음 명령을 입력합니다.

    ```powershell
    az dt twin delete -n adt-az220-training-{your-id} --twin-id factory_2
    ```

    다른 명령과 달리 이 명령은 실행이 완료되어도 출력이 표시되지 않습니다(명령에서 오류가 생성되는 경우는 제외).

1. **factory_1** 과 **cave_1** 간의 관계를 삭제하려면 다음 명령을 입력합니다.

    ```powershell
    az dt twin relationship delete -n adt-az220-training-{your-id} --twin-id factory_1 --relationship-id factory_1_has_cave_1
    ```

    이 명령을 실행하려면 관계 ID가 필요합니다. 지정된 트윈의 관계 ID를 확인할 수 있습니다. 예를 들어 **factory_1** 의 관계 ID를 확인하려는 경우 다음 명령을 입력하면 됩니다.

    ```powershell
    az dt twin relationship list -n adt-az220-training-{your-id} --twin-id factory_1
    ```

    cave_1에 대한 관계를 삭제하기 전에 이 명령을 실행하면 다음과 같은 출력이 표시됩니다.

    ```json
    [
        {
            "$etag": "W/\"a6a9f506-3cfa-4b62-bcf8-c51b5ecc6f6d\"",
            "$relationshipId": "47b0754a-25d1-4b71-ac47-c2409bb08535",
            "$relationshipName": "rel_has_caves",
            "$sourceId": "factory_1",
            "$targetId": "cave_2"
        },
        {
            "$etag": "W/\"b5207e88-7c86-498f-a272-7f81dde88dee\"",
            "$relationshipId": "factory_1_has_cave_1",
            "$relationshipName": "rel_has_caves",
            "$sourceId": "factory_1",
            "$targetId": "cave_1"
        }
    ]
    ```

1. 모델을 삭제하려면 다음 명령을 입력합니다.

    ```powershell
    az dt model delete -n adt-az220-training-{your-id} --dtmi "dtmi:com:contoso:digital_factory:cheese_factory;1"
    ```

    이번에도 출력은 표시되지 않습니다.

    > **중요**: 이 명령이 정상적으로 실행되어 Factory 모델이 삭제되더라도 **factory_1** 디지털 트윈은 계속 남아 있습니다. 그래프를 쿼리하면 삭제된 모델을 사용하여 만든 디지털 트윈도 계속 찾을 수 있습니다. 그러나 모델이 없는 트윈의 속성은 더 이상 업데이트할 수 없습니다. 그러므로 일치하지 않는 그래프가 생성되지 않도록 하려면 모델 관리 작업(버전 관리, 삭제 등)을 완료할 때 각별히 주의해야 합니다.

1. 디지털 트윈의 최근 변경 내용을 표시하려면 **ADT Explorer** 로 돌아갑니다.

1. 표시를 업데이트하려면 브라우저 페이지를 새로 고친 후 **Run Query** 를 선택합니다.

    **Cheese Factory** 모델이 **MODEL** 탐색기에 표시되지 않으며, **TWIN GRAPH** 보기에 **factory_1** 과 **cave_1** 간의 관계도 없습니다.

1. **cave_1** 과 **device_1** 간의 관계를 선택하려면 두 트윈 사이의 선을 클릭합니다.

    선이 굵게 바뀌어 관계가 선택되었음이 표시되며, **Delete Relationship** 단추가 활성화됩니다.

    ![ADT Explorer GRAPH VIEW Delete Relationship](media/LAB_AK_19-graphview-deleterel.png)

1. 관계를 삭제하려면 줄을 마우스 오른쪽 단추로 클릭하고 바로 가기 메뉴에서 **관계 삭제** 를 선택한 다음 **삭제** 를 클릭하여 확인합니다.

    관계가 삭제되며 그래프가 업데이트됩니다.

1. **device_1** 디지털 트윈을 삭제하려면 **device_1** 을 마우스 오른쪽 단추로 클릭하고 바로 가기 메뉴에서 **트윈 삭제** 를 선택합니다.

    > **참고**: **CTRL** 키와 마우스 왼쪽 단추 클릭을 사용하면 여러 개의 트윈을 선택할 수 있습니다. 선택한 여러 개의 트윈을 삭제하려면 마지막 트윈을 마우스 오른쪽 단추로 클릭하고 바로 가기 메뉴에서 **트윈 삭제** 를 선택합니다.

1. 그래프의 모든 디지털 트윈을 삭제하려면 ADT Explorer 페이지 오른쪽 위에서 **Delete All Twins** 를 클릭한 후 **Delete** 를 클릭하여 삭제를 확인합니다.

    ![ADT Explorer Delete All Twins](media/LAB_AK_19-deletealltwins.png)

    > **중요**: 이 기능은 실행 취소할 수 없으므로 사용 시 주의하시기 바랍니다.

1. **MODELS** 탐색기에서 **Cheese Cave Device** 모델을 삭제하려면 해당 **모델 삭제** 단추를 클릭한 다음 **삭제** 를 클릭하여 확인합니다.

1. 모든 모델을 삭제하려면 **MODELS** 탐색기 상단에서 **모든 모델 삭제** 를 클릭합니다.

    > **중요**: 이 기능은 실행 취소할 수 없으므로 사용 시 주의하시기 바랍니다.

이제 ADT 인스턴스에서 모델, 트윈, 관계가 모두 삭제되었습니다. 다음 작업에서 **Import Graph** 기능을 사용하여 새 그래프를 만들 것이므로 걱정하지 않아도 됩니다.

#### <a name="task-6---bulk-import-with-adt-explorer"></a>작업 6 - ADT Explorer를 사용하여 대량 가져오기

**ADT Explorer** 에서는 디지털 트윈 그래프 가져오기와 내보내기가 지원됩니다. **Export** 기능은 모델, 트윈, 관계를 비롯한 최신 쿼리 결과를 JSON 기반 형식으로 직렬화합니다. **Import** 기능은 내보내기에서 생성된 JSON 기반 형식 또는 사용자 지정 Excel 기반 형식에서 역직렬화를 수행합니다. 가져오기를 실행하기 전에 유효성 검사를 위해 그래프 미리 보기가 제공됩니다.

Excel 가져오기 형식에서는 다음 열이 사용됩니다.

* **ModelId**: 인스턴스화해야 하는 모델의 전체 dtmi입니다.
* **ID**: 작성할 트윈의 고유 ID입니다.
* **관계**: 새로운 트윈으로의 나가는 관계가 설정된 트윈 ID입니다.
* **Relationship name**: 이전 열의 트윈에서 나가는 관계의 이름입니다.
* **Init data**: 만들 트윈의 **Property** 설정이 포함된 JSON 문자열입니다.

> **참고**: Excel 가져오기 기능으로는 모델 정의를 가져올 수 **없으며** 트윈과 관계만 가져올 수 있습니다. JSON 형식에서는 모델도 지원됩니다.

아래 표에는 이 작업에서 만들 트윈과 관계가 나와 있습니다(내용을 쉽게 확인할 수 있도록 **Init Data** 값은 제거됨).

| ModelID                                                                | ID             | Relationship (From) | 관계 이름 | Init 데이터 |
| :--------------------------------------------------------------------- | :------------- | :------------------ | :---------------- | :-------- |
| dtmi:com:contoso:digital_factory:cheese_factory;1                      | factory_1      |                     |                   |           |
| dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1        | cave_1       | factory_1           | rel_has_caves   |           |
| dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1        | cave_2       | factory_1           | rel_has_caves   |           |
| dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1        | cave_3       | factory_1           | rel_has_caves   |           |
| dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave_device;1 | sensor-th-0055 | cave_1            | rel_has_devices   |           |
| dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave_device;1 | sensor-th-0056 | cave_2            | rel_has_devices   |           |
| dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave_device;1 | sensor-th-0057 | cave_3            | rel_has_devices   |           |

**cheese-factory-scenario.xlsx** 스프레드시트는 **{file-root}\Allfiles\Labs\19-Azure Digital Twins\Final\Models** 폴더에 있습니다.

1. 브라우저에서 **ADT Explorer** 로 돌아갑니다.

1. **ADT Explorer** 를 사용하여 모델을 가져오려면 **MODELS** 탐색기에서 **모델 업로드** 아이콘을 클릭합니다.

1. **Open** 대화 상자에서 **Models** 폴더로 이동하여 **CheeseFactoryInterface.json**, **CheeseCaveInterface.json** 및 **CheeseCaveDeviceInterface.json** 파일을 선택하고 **Open** 을 클릭합니다.

    그러면 모든 모델이 다시 로드됩니다.

1. **cheese-factory-scenario.xlsx** 스프레드시트를 가져오려면 **Import Graph** 를 클릭합니다.

    ![ADT Explorer GRAPH VIEW Import Graph](media/LAB_AK_19-graphview-importgraph.png)

1. **Open** 대화 상자에서 **Models** 폴더로 이동하여 **cheese-factory-scenario.xlsx** 파일을 선택하고 **Open** 을 클릭합니다.

    **IMPORT** 보기에 가져올 그래프의 미리 보기가 표시됩니다.

    ![ADT Explorer GRAPH VIEW 가져오기 미리 보기](media/LAB_AK_19-graphview-importpreview.png)

1. 가져오기를 완료하려면 **Start Import** 를 클릭합니다.

    트윈 7개와 관계 6개를 가져왔다는 메시지가 포함된 **Import Successful** 대화 상자가 표시됩니다. **Close** 를 클릭하여 계속 진행합니다.

1. **TWIN GRAPH** 보기로 돌아가서 **쿼리 실행** 을 클릭합니다.

    가져온 그래프가 표시됩니다. 각 트윈을 클릭하여 속성을 확인할 수 있습니다(각 트윈은 값이 적용되어 초기화됨).

1. 현재 그래프를 JSON으로 내보내려면 **Export Graph**(앞에서 사용한 **Import Graph** 단추 옆에 있음)를 클릭합니다.

    왼쪽 위에 **Download** 링크가 있는 **Export** 보기가 표시됩니다.

1. JSON으로 모델을 다운로드하려면 **Download** 링크를 클릭합니다.

    브라우저에서 모델이 다운로드됩니다.

1. JSON을 확인하려면 Visual Studio Code에서 다운로드된 파일을 엽니다.

    JSON이 한 줄로 표시되면 명령 팔레트를 통해 **Format Document** 명령을 사용하거나 **Shift+Alt+F** 를 눌러 JSON의 서식을 다시 지정합니다.

    JSON에는 세 가지 주요 섹션이 있습니다.

    * **digitalTwinsFileInfo** - 내보낸 파일 형식의 버전을 포함합니다.
    * **digitalTwinsGraph** - 내보낸 그래프에 표시된 모든 트윈 및 관계의 인스턴스 데이터를 포함합니다(쿼리에 따라 표시된 것만).
    * **digitalTwinsModels** - 모델 정의입니다.

    > **참고**: Excel 형식과는 달리 JSON 파일에는 모델 정의가 포함되어 있습니다. 그러므로 파일 하나만 사용하면 모든 항목을 가져올 수 있습니다.

1. JSON 파일을 가져오려면 **ADT Explorer** 를 사용하여 이전 작업의 지침에 따라 모델과 트윈을 삭제한 후에 방금 만든 JSON 내보내기 파일을 가져옵니다. 모델, 트윈과 해당 속성 및 관계는 다시 작성됩니다.

이 트윈 그래프는 쿼리 관련 연습에서 기준 그래프로 사용됩니다.

### <a name="exercise-4---query-the-graph-using-adt-explorer"></a>연습 4 - ADT Explorer를 사용하여 그래프 쿼리

>**참고**: 이 연습을 진행하려면 이전 연습에서 가져온 그래프가 필요합니다.

먼저 디지털 트윈 그래프 쿼리 언어를 잠시 검토해 보겠습니다.

방금 작성한 디지털 트윈 그래프를 쿼리하여 디지털 트윈과 해당 트윈에 포함된 관계에 대한 정보를 확인할 수 있습니다. 이러한 쿼리는 Azure Digital Twins 쿼리 언어라고 하는 SQL과 유사한 사용자 지정 쿼리 언어로 작성합니다. 또한 이 언어는 Azure IoT Hub의 쿼리 언어와 유사합니다.

Digital Twins REST API와 SDK를 통해 쿼리를 수행할 수 있습니다. 이 연습에서는 API 호출을 자동으로 처리하는 Azure Digital Twins Explorer 샘플 앱을 사용합니다. 추가 도구는 이 랩 뒷부분에서 살펴봅니다.

> **참고**: 그래프 시각화용 도구인 **ADT Explorer** 에서는 이름 등 트윈에서 선택한 단일 값이 아닌 전체 트윈만 표시할 수 있습니다.

#### <a name="task-1---query-using-the-adt-explorer"></a>작업 1 - ADT Explorer를 사용하여 쿼리

이 작업에서는 ADT Explorer를 사용해 그래프 쿼리를 실행하고 결과를 그래프로 렌더링합니다. 트윈은 속성, 모델 유형 및 관계를 기준으로 쿼리할 수 있습니다. 조합 연산자를 사용해 여러 쿼리를 결합하여 복합 쿼리로 만들 수 있습니다. 이러한 복합 쿼리를 사용하면 한 번에 여러 트윈 설명자 유형을 쿼리할 수 있습니다.

1. 브라우저에서 **ADT Explorer** 로 돌아갑니다.

1. **QUERY EXPLORER** 쿼리가 다음과 같이 설정되어 있는지 확인합니다.

    ```sql
    SELECT * FROM digitaltwins
    ```

    SQL 사용법을 알고 있다면 이 쿼리는 디지털 트윈의 모든 정보를 반환함을 알 수 있을 것입니다.

1. 이 쿼리를 실행하려면 **Run Query** 를 클릭합니다.

    방금 언급했던 대로 전체 그래프가 표시됩니다.

1. 이 그래프를 명명된 쿼리로 저장하려면 **Save** 아이콘(**Run Query** 단추 바로 오른쪽에 있음)을 클릭합니다.

1. **Save Query** 대화 상자에서 이름으로 **All Twins** 를 입력하고 **Save** 를 클릭합니다.

    쿼리가 로컬에 저장되며, 쿼리 텍스트 상자 왼쪽의 **Saved Queries** 드롭다운에서 해당 쿼리를 사용할 수 있습니다. 저장한 쿼리를 삭제하려면 **Saved Queries** 드롭다운이 열려 있을 때 쿼리 이름 옆의 **X** 아이콘을 클릭합니다.

    > **팁**: **All Twins** 쿼리를 실행하면 언제든지 전체 보기로 돌아올 수 있습니다.

1. **Cheese Cave** 트윈만 표시되도록 그래프를 필터링하려면 다음 쿼리를 입력하여 실행합니다.

    ```sql
    SELECT * FROM digitaltwins
    WHERE IS_OF_MODEL('dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1')
    ```

    이제 그래프에 **Cheese Cave** 트윈 3개만 표시됩니다.

    이 쿼리를 **Just Caves** 로 저장합니다.

1. **inUse** 상태인 **Cheese Cave** 트윈만 표시하려면 다음 쿼리를 입력하여 실행합니다.

    ```sql
    SELECT * FROM digitaltwins
    WHERE IS_OF_MODEL('dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1')
    AND inUse = true
    ```

    이제 그래프에 **cave_3** 및 **cave_1** 만 표시됩니다.

1. **inUse** 상태이며 **temperatureAlert** 가 있는 **Cheese Cave** 트윈만 표시하려면 다음 쿼리를 입력하여 실행합니다.

    ```sql
    SELECT * FROM digitaltwins
    WHERE IS_OF_MODEL('dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1')
    AND inUse = true
    AND temperatureAlert = true
    ```

    이제 그래프에 **cave_3** 만 표시됩니다.

1. 관계를 사용하여 조인을 통해 **sensor-th-0055** 디바이스의 부모를 찾으려면 다음 쿼리를 입력합니다.

    ```sql
    SELECT Parent FROM digitaltwins Parent
    JOIN Child RELATED Parent.rel_has_devices
    WHERE Child.$dtId = 'sensor-th-0055'
    ```

    **cave_1** 트윈이 표시됩니다.

    SQL JOIN 사용법을 알고 있다면 여기서 사용된 구문이 기존의 JOIN 구문과 다르다는 점을 눈치채셨을 것입니다. 즉, 기존 JOIN 구문에서와 같이 WHERE 절의 키 값을 사용하여 이 JOIN의 상관 관계를 설정하거나 JOIN 정의와 인라인으로 키 값을 지정하는 대신 여기서는 **rel_has_devices** 관계의 이름을 지정했습니다. 관계 속성 자체가 대상 엔터티를 식별하기 때문에 이 상관 관계는 자동으로 컴퓨팅됩니다. 관계 정의는 다음과 같습니다.

    ```json
    {
        "@type": "Relationship",
        "@id": "dtmi:com:contoso:digital_factory:cheese_cave:rel_has_devices;1",
        "name": "rel_has_devices",
        "displayName": "Has devices",
        "target": "dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave_device;1"
    }
    ```

#### <a name="task-2---query-for-properties-using-the-adt-explorer"></a>작업 2 - ADT Explorer를 사용하여 속성 쿼리

**ADT Explorer** 의 주요 제한 사항 중 하나는, 이 앱은 그래프 렌더링용으로 제작되었으므로 속성만 반환하는 쿼리의 결과는 표시할 수 없다는 점입니다. 이 작업에서는 코딩 솔루션을 사용하지 않고도 이러한 쿼리의 결과를 확인하는 방법을 알아봅니다.

1. 속성만 반환하는 유효한 쿼리를 실행하려면 다음 쿼리를 입력하고 **쿼리 실행** 을 클릭합니다.

    ```sql
    SELECT Parent.desiredTemperature FROM digitaltwins Parent
    JOIN Child RELATED Parent.rel_has_devices
    WHERE Child.$dtId = 'sensor-th-0055'
    ```

    이 쿼리는 오류 없이 실행되기는 하지만 그래프는 표시되지 않습니다. 하지만 **ADT Explorer** 에서도 결과를 확인할 수 있는 방법이 있습니다. 다음 작업에서 **Output** 창을 열어 쿼리 결과를 확인하겠습니다.

1. **Output** 창을 열려면 페이지 오른쪽 위의 **Settings** 아이콘을 클릭합니다.

1. 그러면 표시되는 대화 상자의 **View** 아래에서 **Output** 을 사용하도록 설정하고 대화 상자를 닫습니다.

    페이지 아래쪽에 **Output** 창이 나타납니다.

1. 위의 쿼리를 다시 실행하고 **Output** 창의 내용을 검토합니다.

    OUTPUT 창에 **Requested query** 가 표시된 다음 반환된 JSON이 표시됩니다. 이 JSON은 다음과 같습니다.

    ```json
    {
        "queryCharge": 20.259999999999998,
        "connection": "close",
        "content-encoding": "gzip",
        "content-type": "application/json; charset=utf-8",
        "date": "Thu, 25 Mar 2021 21:34:40 GMT",
        "strict-transport-security": "max-age=2592000",
        "traceresponse": "00-182f5e54efb95c4b8b3e2a6aac15499f-9c5ffe6b8299584e-01",
        "transfer-encoding": "chunked",
        "vary": "Accept-Encoding",
        "x-powered-by": "Express",
        "value": [
            {
            "desiredTemperature": 50
            }
        ],
        "continuationToken": null
    }
    ```

    위의 JSON에는 추가 결과 메타데이터가 포함되어 있으며, **value** 속성에는 선택한 **desiredTemperature** 속성과 값이 포함되어 있습니다.

### <a name="exercise-5---configure-and-launch-device-simulator"></a>연습 5 - 디바이스 시뮬레이터 구성 및 시작

이전 연습에서는 개념 증명용 디지털 트윈 모델과 그래프를 만들었습니다. 이 개념 증명을 통해 IoT Hub에서 ADT로 디바이스 메시지 트래픽을 라우팅하는 방법을 시연하려는 경우 디바이스 시뮬레이터를 사용하면 유용합니다. 이 연습에서는 IoT Hub에 원격 분석 데이터를 보내도록 시뮬레이션된 디바이스 앱을 구성합니다.

#### <a name="task-1-open-the-device-simulator-project"></a>작업 1: 디바이스 시뮬레이터 프로젝트 열기

이 작업에서는 구성 준비를 위해 Visual Studio Code에서 Cheese Cave Device 시뮬레이터 앱을 엽니다.

1. **Visual Studio Code** 를 사용하여 ASP.NET 5 API 앱을 만드는 방법을 보여줍니다.

1. **파일** 메뉴에서 **폴더 열기** 를 클릭합니다.

1. 폴더 열기 대화 상자에서 랩 19 Starter 폴더로 이동합니다.

    랩 3: 개발 환경 설정에서 ZIP 파일을 다운로드하고 콘텐츠를 로컬로 추출하여 랩 리소스를 포함하는 GitHub 리포지토리를 복제했습니다. 추출된 폴더 구조에는 다음 폴더 경로가 포함됩니다.

    * Allfiles
        * 랩
            * 19-Azure Digital Twins
                * Starter
                    * CheeseCaveDevice

1. **CheeseCaveDevice** 를 클릭하고 **폴더 선택** 을 클릭합니다.

    Visual Studio Code의 EXPLORER 창에 다음 파일이 나열되어야 합니다.

    * CheeseCaveDevice.csproj
    * Program.cs

1. 코드 파일을 열려면 **Program.cs** 를 클릭합니다.

    이 애플리케이션은 이전 랩의 작업에서 사용했던 시뮬레이션된 디바이스 애플리케이션과 매우 비슷함을 쉽게 확인할 수 있습니다. 이 버전은 대칭 키 인증을 사용하며 원격 분석 및 로깅 메시지를 IoT Hub로 전송합니다. 그리고 이전 랩의 애플리케이션보다 더 복잡한 센서가 구현되어 있습니다.

1. **터미널** 메뉴에서 **새 터미널** 을 클릭합니다.

    명령 프롬프트의 일부로 표시된 디렉터리 경로를 확인합니다. 이전 랩 프로젝트의 폴더 구조 내에서 이 프로젝트를 빌드하지 않으려고 합니다.

1. 터미널 명령 프롬프트에서 애플리케이션 빌드를 확인하려면 다음 명령을 입력합니다.

    ```bash
    dotnet build
    ```

    다음과 유사하게 출력됩니다.

    ```text
    > dotnet build
    Microsoft (R) Build Engine version 16.5.0+d4cbfca49 for .NET Core
    Copyright (C) Microsoft Corporation. All rights reserved.

    Restore completed in 39.27 ms for D:\Az220\AllFiles\Labs\19-Azure Digital Twins\Starter\CheeseCaveDevice\CheeseCaveDevice.csproj.
    CheeseCaveDevice -> D:\Az220\AllFiles\Labs\19-Azure Digital Twins\Starter\CheeseCaveDevice\bin\Debug\netcoreapp3.1\CheeseCaveDevice.dll

    Build succeeded.
        0 Warning(s)
        0 Error(s)

    Time Elapsed 00:00:01.16
    ```

다음 작업에서 연결 문자열을 구성하고 애플리케이션을 검토합니다.

#### <a name="task-2-configure-connection-and-review-code"></a>작업 2: 연결 구성 및 코드 검토

이 작업에서 빌드하는 시뮬레이션된 디바이스 앱은 온도와 습도를 모니터링하는 IoT 디바이스를 시뮬레이트합니다. 랩 15에서 빌드했던 것과 동일한 이 앱은 2초마다 센서 판독값을 시뮬레이션하고 센서 데이터와 통신합니다.

1. **Visual Studio Code** 에서 Program.cs 파일이 열려 있는지 확인합니다.

1. 코드 편집기에서 다음 코드 줄을 찾습니다.

    ```csharp
    private readonly static string deviceConnectionString = "<your device connection string>";
    ```

1. **\<your device connection string\>** 을 랩 설정 연습의 끝부분에서 저장해 둔 디바이스 연결 문자열로 바꿉니다.

    IoT Hub로 원격 분석을 보내기 전에 구현해야 하는 변경은 이 항목뿐입니다.

    > **참고**: 랩 설정 연습에서는 디바이스 및 서비스 연결 문자열을 모두 저장했습니다. 여기서는 디바이스 연결 문자열을 입력해야 합니다.

1. **파일** 메뉴에서 **저장** 을 클릭합니다.

#### <a name="task-3-test-your-code-to-send-telemetry"></a>작업 3: 원격 분석 전송 코드 테스트

이 작업에서는 구성한 시뮬레이션된 앱을 시작하고 원격 분석이 정상적으로 전송되는지 확인합니다.

1. Visual Studio Code에서 터미널이 아직 열려 있는지 확인합니다.

1. 시뮬레이션된 디바이스 앱을 실행하려면 터미널 명령 프롬프트에서 다음 명령을 입력합니다.

    ```bash
    dotnet run
    ```

   이 명령은 현재 폴더의 **Program.cs** 파일을 실행합니다.

1. 터미널로 전송되는 출력을 확인합니다.

    다음과 유사한 콘솔 출력을 빠르게 확인해야 합니다.

    ![콘솔 출력](media/LAB_AK_19-cheesecave-telemetry.png)

    > **참고**:  녹색 텍스트는 실행이 정상적으로 완료되었음을 나타냅니다. 빨간색 텍스트는 문제가 발생했음을 나타냅니다. 위의 이미지와 비슷한 화면이 표시되지 않으면 디바이스 연결 문자열부터 확인합니다.

1. 이 앱을 실행 상태로 둡니다.

    이 랩의 후반부에 IoT Hub로 원격 분석을 보내야 합니다.

### <a name="exercise-6----set-up-azure-function-to-ingest-data"></a>연습 6 - 데이터를 수집하도록 Azure 함수 설정

개념 증명에서 진행해야 하는 중요한 작업 중  하나는 디바이스의 데이터를 Azure Digital Twins로 제공하는 방법을 시연하는 것입니다. Virtual machines, Azure Functions 및 Logic Apps와 같은 외부 컴퓨팅 리소스를 통해 Azure Digital Twins로 데이터를 수집할 수 있습니다. 이 연습에서는 IoT Hub의 기본 제공 Event Grid를 통해 함수 앱을 호출합니다. 함수 앱은 데이터를 수신하고 Azure Digital Twins API를 사용하여 적절하 디지털 트윈 인스턴스에서 속성을 설정합니다.

#### <a name="task-1---create-and-configure-a-function-app"></a>작업 1 - 함수 앱 만들기 및 구성

IoT Hub Event Grid 엔드포인트가 Azure 함수로 원격 분석을 라우팅하도록 구성하려면 먼저 Azure 함수를 만들어야 합니다. 이 작업에서는 개별 Azure Functions가 실행되는 실행 컨텍스트를 제공하는 Azure 함수 앱을 만듭니다.

Azure Digital Twins와 해당 API에 액세스하려면 적절한 권한이 있는 서비스 주체를 사용해야 합니다. 이 작업에서는 함수 앱용으로 서비스 주체를 만든 후에 적절한 권한을 할당합니다. 함수 앱에 적절한 권한이 할당되면 함수 앱 컨텍스트 내에서 실행되는 모든 Azure 함수는 해당 서비스 주체를 사용하므로 ADT 액세스 권한이 부여됩니다.

함수 앱 컨텍스트에서는 함수 하나 이상의 앱 설정을 관리할 수 있는 환경도 제공됩니다. 이 작업에서는 해당 기능을 사용하여 ADT 연결 문자열이 포함된 설정을 정의합니다. 그러면 Azure Functions에서 해당 연결 문자열을 읽을 수 있습니다. 함수 코드에 값을 하드 코드하는 방식보다는 앱 설정에서 연결 문자열 및 기타 구성 값을 캡슐화하는 방식이 훨씬 효율적입니다.

1. Azure Portal이 표시된 브라우저 창을 열고 Azure Cloud Shell을 엽니다.

1. Cloud Shell 명령 프롬프트에서 Azure 함수 앱을 만들려면 다음 명령을 입력합니다.

    ```bash
    az functionapp create --resource-group rg-az220 --consumption-plan-location {your-location} --name func-az220-hub2adt-training-{your-id} --storage-account staaz220training{your-id} --functions-version 3
    ```

    > **참고**: 위의 **{사용자 위치}** 및 **{사용자 ID}** 토큰은 적절한 값으로 바꿔야 합니다.

    Azure 함수가 Azure Digital Twins에 인증을 하려면 전달자 토큰을 함수에 전달해야 합니다. 이 토큰이 전달되었는지 확인하려면 함수 앱에 대한 관리 ID를 만들어야 합니다.

1. 함수 앱용 시스템 관리 ID를 작성(할당)하고 연결된 서비스 주체 ID를 표시하려면 다음 명령을 입력합니다.

    ```bash
    az functionapp identity assign -g rg-az220 -n func-az220-hub2adt-training-{your-id} --query principalId -o tsv
    ```

    > **참고**: 위의 **{사용자 ID}** 토큰은 적절한 값으로 바꿔야 합니다.

    그러면 다음과 같은 출력이 표시됩니다.

    ```bash
    1179da2d-cc37-48bb-84b3-544cbb02d194
    ```

    이 출력이 함수 앱에 할당된 서비스 주체 ID입니다. 다음 단계에서 이 서비스 주체 ID가 필요합니다.

1. 함수 앱 서비스 주체에 **Azure Digital Twins 데이터 소유자** 역할을 할당하려면 다음 명령을 입력합니다.

    ```bash
    az dt role-assignment create --dt-name adt-az220-training-{your-id} --assignee {principal-id} --role "Azure Digital Twins Data Owner"
    ```

    > **참고**: 위의 **{사용자 ID}** 및 **{서비스 주체 ID}** 토큰은 적절한 값으로 바꿔야 합니다. **{서비스 주체 ID}** 값은 이전 단계의 출력으로 표시되었던 값입니다.

    이제 Azure 함수 앱에 서비스 주체가 할당되었으므로 해당 서비스 주체가 Azure Digital Twins 인스턴스에 액세스할 수 있도록 **Azure Digital Twins 데이터 소유자** 역할을 할당해야 합니다.

1. Azure 함수 앱에 Azure Digital Twins 인스턴스 URL을 환경 변수로 제공하려면 다음 명령을 입력합니다.

    ```bash
    az functionapp config appsettings set -g rg-az220 -n func-az220-hub2adt-training-{your-id} --settings "ADT_SERVICE_URL={adt-url}"
    ```

    > **참고**: 위의 **{사용자 ID}** 및 **{ADT URL}** 토큰은 적절한 값으로 바꿔야 합니다. 이전 작업에서 **{adt-url}** 값이 **adt-connection.txt** 파일에 저장되었습니다. 이는 `https://adt-az220-training-dm030821.api.eus.digitaltwins.azure.net`과 비슷합니다.

    명령 실행이 완료되면 사용 가능한 모든 설정의 목록이 표시됩니다. 이제 Azure 함수가 **ADT_SERVICE_URL** 값을 읽어 ADT 서비스 URL을 가져올 수 있습니다.

#### <a name="task-2---review-contosoadtfunctions-project"></a>작업 2 - Contoso.AdtFunctions 프로젝트 검토

이 작업에서는 연결된 Event Grid에서 이벤트가 발생할 때마다 실행되는 Azure 함수를 검토합니다. 이 함수가 실행되면 이벤트가 처리되며 메시지와 원격 분석이 ADT로 라우팅됩니다.

1. **Visual Studio Code** 에서 **Contoso.AdtFunctions** 폴더를 엽니다.

1. **Contoso.AdtFunctions.csproj** 파일을 엽니다.

> **참고**: 랩 3: 개발 환경 설정에서 ZIP 파일을 다운로드하고 콘텐츠를 로컬로 추출하여 랩 리소스를 포함하는 GitHub 리포지토리를 복제했습니다. 추출된 폴더 구조에는 다음 폴더 경로가 포함됩니다.
>
> * Allfiles
>   * 랩
>       * 19-Azure Digital Twins
>           * 최종
>               * Contoso.AdtFunctions

    Notice the project references the following NuGet Packages:

    * **Azure.DigitalTwins.Core** 패키지에는 Azure Digital Twins 서비스용 SDK가 포함되어 있습니다. 이 라이브러리는 트윈, 모델, 관계 등을 관리할 수 있도록 Azure Digital Twins 서비스 액세스 권한을 제공합니다.
    * **Microsoft.Azure.WebJobs.Extensions.EventGrid** 패키지는 Azure Functions에서 Event Grid 웹후크 호출을 받기 위한 기능을 제공하므로 Event Grid에 게시된 모든 이벤트에 응답하는 함수를 쉽게 작성할 수 있습니다.
    * **Microsoft.Azure.WebJobs.Extensions.EventHubs** 패키지는 Azure Functions에서 Event Hub 웹후크 호출을 받기 위한 기능을 제공하므로 Event Hub에 게시된 모든 이벤트에 응답하는 함수를 쉽게 작성할 수 있습니다.
    * **Microsoft.NET.Sdk.Functions** 패키지는 .NET 함수 프로젝트를 빌드하기 위한 빌드 작업을 포함합니다.
    * **Azure.identity**  패키지에는 Azure Identity용 Azure SDK 클라이언트 라이브러리 구현이 포함되어 있습니다. Azure Identity 라이브러리는 Azure SDK 전체에서 Azure Active Directory 토큰 인증을 지원합니다. 즉, 이 라이브러리는 AAD 토큰 인증을 지원하는 Azure SDK 클라이언트를 생성하는 데 사용할 수 있는 TokenCredential 구현 집합을 제공합니다.
    * **System.Net.Http** 패키지는 최신 HTTP 애플리케이션용 프로그래밍 인터페이스를 제공합니다. 이러한 인터페이스에는 애플리케이션이 HTTP를 통해 웹 서비스를 사용할 수 있도록 하는 HTTP 클라이언트 구성 요소, 그리고 클라이언트와 서버가 모두 HTTP 헤더를 구문 분석하는 데 사용할 수 있는 HTTP 구성 요소 등이 포함됩니다.

1. Visual Studio Code에서 **HubToAdtFunction.cs** 파일을 엽니다.

1. 함수의 멤버 변수를 검토하려면 `// INSERT member variables below here` 주석을 찾아서 그 아래의 코드를 검토합니다.

    ```csharp
    //Your Digital Twins URL is stored in an application setting in Azure Functions.
    private static readonly string adtInstanceUrl = Environment.GetEnvironmentVariable("ADT_SERVICE_URL");
    private static readonly HttpClient httpClient = new HttpClient();
    ```

    이 연습 앞부분에서 정의한 **ADT_SERVICE_URL** 환경 변수의 값이 **adtInstanceUrl** 변수에 할당되어 있습니다. 또한 이 코드는 **HttpClient** 의 단일 정적 인스턴스를 사용하는 모범 사례를 따릅니다.

1. **Run** 메서드 선언을 찾아서 다음 주석을 검토합니다.

    ```csharp
    [FunctionName("HubToAdtFunction")]
    public async static Task Run([EventGridTrigger] EventGridEvent eventGridEvent, ILogger log)
    ```

    위의 코드에서는 **FunctionName** 특성을 사용하여 **Run** 메서드를 **HubToAdtFunction** 의 진입점 **Run** 으로 표시했습니다. Azure Digital Twin을 업데이트하는 코드는 비동기적으로 실행되므로 이 메서드는 `async`로도 선언되어 있습니다.

    **eventGridEvent** 매개 변수에는 함수 호출을 트리거한 Event Grid 이벤트가 할당되어 있습니다. 그리고 **log** 매개 변수는 디버그에 사용 가능한 로거 액세스 권한을 제공합니다.

    > **팁**: Azure Functions용 Azure Event Grid 트리거에 대해 자세히 알아보려면 아래 리소스를 검토하세요.
    > * [Azure Functions의 Azure Event Grid 트리거](https://docs.microsoft.com/azure/azure-functions/functions-bindings-event-grid-trigger?tabs=csharp%2Cbash)

1. 정보 데이터를 기록하는 방법을 검토하려면 다음 코드를 찾습니다.

    ```csharp
    log.LogInformation(eventGridEvent.Data.ToString());
    ```

    **Microsoft.Extensions.Logging** 네임스페이스에서 정의되는 **ILogger** 인터페이스는 대다수 로깅 패턴을 단일 메서드 호출로 집계합니다. 여기서는 로그 항목이 **정보** 수준에서 생성되어 있습니다. 이 밖에도 위험, 오류와 같은 여러 수준을 위한 메서드가 있습니다. 기타 Azure Function이 클라우드에서 실행 중이므로 개발 및 프로덕션 중에는 로깅이 필수적입니다.

    > **팁:** **Microsoft.Extensions.Logging** 기능에 대해 자세히 알아보려면 다음 리소스를 검토하세요.
    > * [.NET의 로깅](https://docs.microsoft.com/dotnet/core/extensions/logging?tabs=command-line)
    > * [Microsoft.Extensions.Logging 네임스페이스](https://docs.microsoft.com/dotnet/api/microsoft.extensions.logging?view=dotnet-plat-ext-5.0&viewFallbackFrom=netcore-3.1)
    > * [ILogger 인터페이스](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.logging.ilogger?view=dotnet-plat-ext-5.0&viewFallbackFrom=netcore-3.1)

1. 다음 코드를 찾아서 **adtInstanceUrl** 변수가 확인되는 방식을 살펴봅니다.

    ```csharp
    if (adtInstanceUrl == null)
    {
        log.LogError("Application setting \"ADT_SERVICE_URL\" not set");
        return;
    }
    ```

    이 코드는 **adtInstanceUrl** 변수가 설정되었는지 확인하고, 설정되지 않은 경우 오류가 로깅되고 함수가 종료됩니다. 이 연습에서는 로깅을 수행하여 함수가 잘못 구성되었음을 파악하는 방법을 확인했습니다.

1. 예외가 모두 로깅되도록 하기 위해 `try..catch` 루프가 사용됩니다.

    ```csharp
    try
    {
        // ... main body of code
    }
    catch (Exception e)
    {
        log.LogError(e.Message);
    }
    ```

    예외 메시지가 로깅됩니다.

1. ADT를 인증하고 클라이언트 인스턴스를 만드는 데 함수 앱 주체가 어떻게 사용되는지 살펴보려면 `// REVIEW authentication code below here` 주석을 찾아서 다음 코드를 검토합니다.

    ```csharp
    ManagedIdentityCredential cred = new ManagedIdentityCredential("https://digitaltwins.azure.net");
    DigitalTwinsClient client = new DigitalTwinsClient(new Uri(adtInstanceUrl), cred, new DigitalTwinsClientOptions { Transport = new HttpClientTransport(httpClient) });
    log.LogInformation($"Azure digital twins service client connection created.");
    ```

    위의 코드에서는 **ManagedIdentityCredential** 클래스가 사용되었습니다. 이 클래스는 앞에서 배포 환경에 할당했던 관리 ID를 사용하여 인증을 시도합니다. 반환된 자격 증명은 **DigitalTwinsClient** 인스턴스를 생성하는 데 사용됩니다. 클라이언트에는 모델, 구성 요소, 속성, 관계 등의 디지털 트윈 정보를 검색하고 업데이트하는 메서드가 포함되어 있습니다.

1. Event Grid 이벤트의 처리를 시작하는 코드를 검토하려면 `// REVIEW event processing code below here` 주석을 찾아서 그 아래의 다음 코드를 검토합니다.

    ```csharp
    if (eventGridEvent != null && eventGridEvent.Data != null)
    {
        // Read deviceId and temperature for IoT Hub JSON.
        JObject deviceMessage = (JObject)JsonConvert.DeserializeObject(eventGridEvent.Data.ToString());
        string deviceId = (string)deviceMessage["systemProperties"]["iothub-connection-device-id"];
        var fanAlert = (bool)deviceMessage["properties"]["fanAlert"]; // cast directly to a bool
        var temperatureAlert = deviceMessage["properties"].SelectToken("temperatureAlert") ?? false; // JToken object
        var humidityAlert = deviceMessage["properties"].SelectToken("humidityAlert") ?? false; // JToken object
        log.LogInformation($"Device:{deviceId} fanAlert is:{fanAlert}");
        log.LogInformation($"Device:{deviceId} temperatureAlert is:{temperatureAlert}");
        log.LogInformation($"Device:{deviceId} humidityAlert is:{humidityAlert}");

        var bodyJson = Encoding.ASCII.GetString((byte[])deviceMessage["body"]);
        JObject body = (JObject)JsonConvert.DeserializeObject(bodyJson);
        log.LogInformation($"Device:{deviceId} Temperature is:{body["temperature"]}");
        log.LogInformation($"Device:{deviceId} Humidity is:{body["humidity"]}");
    }
    ```

    위의 코드에서는 JSON deserialization을 사용하여 이벤트 데이터에 액세스합니다. 이벤트 데이터 JSON은 다음과 같습니다.

    ```JSON
    {
        "properties": {
            "sensorID": "S1",
            "fanAlert": "false",
            "temperatureAlert": "true",
            "humidityAlert": "true"
        },
        "systemProperties": {
            "iothub-connection-device-id": "sensor-th-0055",
            "iothub-connection-auth-method": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
            "iothub-connection-auth-generation-id": "637508617957275763",
            "iothub-enqueuedtime": "2021-03-11T03:27:21.866Z",
            "iothub-message-source": "Telemetry"
        },
        "body": "eyJ0ZW1wZXJhdHVyZSI6OTMuOTEsImh1bWlkaXR5Ijo5OC4wMn0="
    }
    ```

    인덱서 방식을 사용하면 메시지 **properties** 및 **systemProperties** 에 쉽게 액세스할 수 있습니다. 그러나 **temperatureAlert**, **humidityAlert** 과 같이 속성이 선택 사항인 경우에는 예외가 throw되지 않도록 `SelectToken`을 사용해야 하며 null 병합 작업을 수행해야 합니다.

    > **팁**: null 병합 연산자 `??`에 대해 자세히 알아보려면 다음 콘텐츠를 검토하세요.
    > * [?? 및 ??= 연산자(C# 참조)](https://docs.microsoft.com/dotnet/csharp/language-reference/operators/null-coalescing-operator)

    ASCII로 인코딩된 JSON인 메시지 **body** 에는 원격 분석 페이로드가 포함됩니다. 그러므로 body를 먼저 디코드한 후 역직렬화해야 원격 분석 속성에 액세스할 수 있습니다.

    > **팁**: 이벤트 스키마에 대해 자세히 알아보려면 다음 리소스를 검토하세요.
    > * [이벤트 스키마](https://docs.microsoft.com/azure/azure-functions/functions-bindings-event-grid-trigger?tabs=csharp%2Cbash#event-schema)

1. ADT 트윈을 업데이트하는 코드를 검사하려면 `// REVIEW ADT update code below here` 주석을 찾아서 그 아래의 다음 코드를 검토합니다.

    ```csharp
    //Update twin
    var patch = new Azure.JsonPatchDocument();
    patch.AppendReplace<bool>("/fanAlert", fanAlert); // already a bool
    patch.AppendReplace<bool>("/temperatureAlert", temperatureAlert.Value<bool>()); // convert the JToken value to bool
    patch.AppendReplace<bool>("/humidityAlert", humidityAlert.Value<bool>()); // convert the JToken value to bool

    await client.UpdateDigitalTwinAsync(deviceId, patch);

    // publish telemetry
    await client.PublishTelemetryAsync(deviceId, null, bodyJson);
    ```

    위의 코드에서는 두 가지 방식으로 디지털 트윈에 데이터를 적용합니다. 첫 번째 방식은 JSON 패치를 사용해 속성을 업데이트하는 것이고, 두 번째 방식은 원격 분석 데이터를 게시하는 것입니다.

    ADT 클라이언트는 JSON Patch 문서를 사용해 디지털 트윈 속성을 추가하거나 업데이트합니다. JSON Patch는 JSON 문서에 적용할 작업 시퀀스를 표현하는 JSON 문서 구조를 정의합니다. 패치에는 여러 값이 추가 또는 바꾸기 작업으로 추가됩니다. 그러면 ADT가 비동기식으로 업데이트됩니다.

   > **팁**: JSON Patch 문서에 대해 자세히 알아보려면 다음 리소스를 검토하세요.
   > * [JSON(Javascript Object Notation) 패치](https://tools.ietf.org/html/rfc6902)
   > * [JSON 패치란?](http://jsonpatch.com/)
   > * [JsonPatchDocument Class](https://docs.microsoft.com/dotnet/api/azure.jsonpatchdocument?view=azure-dotnet)

   > **중요**: `AppendReplace` 작업을 사용하기 전에 디지털 트윈 인스턴스에 기존 값이 있어야 합니다.

   원격 분석 데이터는 속성과는 다른 방식으로 처리됩니다. 즉, 원격 분석 데이터는 디지털 트윈 속성을 설정하는 데 사용되지 않으며 원격 분석 이벤트로 게시됩니다. 이러한 메커니즘이 사용되므로 디지털 트윈 이벤트 경로의 다운스트림 구독자가 원격 분석을 사용할 수 있습니다.

   > **참고**: 원격 분석 메시지를 게시하기 전에 디지털 트윈 이벤트 경로를 정의해야 합니다. 이렇게 하지 않으면 메시지가 사용 가능하도록 라우팅되지 않습니다.

이제 함수를 게시할 수 있습니다.

> **참고**: **TelemetryFunction.cs** 함수는 이후 작업에서 검토합니다.

#### <a name="task-3---publish-functions"></a>작업 3 - 함수 게시

1. Visual Studio Code용 Azure Functions 확장에서 **함수 앱에 배포** 를 선택합니다.

    ![Visual Studio Code 함수 앱에 배포](media/LAB_AK_19-deploy-to-function-app.png)

1. 메시지가 표시되면 다음 항목을 선택합니다.

    * **Azure에 로그인**: 메시지가 표시되면 Azure에 로그인합니다.
    * **구독 선택**: 메시지가 표시되면 이 과정에서 사용하고 있는 구독을 선택합니다.
    * **Azure에서 함수 앱 선택**: **func-az220-hub2adt-training-{사용자 ID}** 를 선택합니다.

    배포를 확인하라는 메시지가 표시되면 **배포** 를 클릭합니다.

    함수가 컴파일되며, 컴파일이 정상적으로 완료되면 함수가 배포됩니다. 어느 정도 시간이 걸릴 수 있습니다.

1. 배포가 완료되면 다음 메시지가 표시됩니다.

    ![Visual Studio Code 배포 완료 - 로그 스트리밍 선택](media/LAB_AK_19-function-stream-logs.png)

    **로그 스트리밍** 를 선택하고 애플리케이션 로깅을 사용할지 확인하라는 대화 상자에서 **예** 를 클릭합니다.

    그러면 **출력** 창에 배포된 함수의 로그 스트림이 표시됩니다. 이 스트림은 2시간 후에 만료됩니다. 상태 정보도 몇 가지 표시됩니다. 그러나 함수 앱을 시작할 때까지는 함수 자체에서 진단 정보가 제공되지는 않습니다. 진단 정보에 대해서는 다음 연습에서 살펴봅니다.

    Visual Studio Code에서 Azure 함수를 마우스 오른쪽 단추로 클릭하고 **스트리밍 로그 시작** 또는 **스트리밍 로그 중지** 를 선택하면 언제든지 스트리밍을 중지하거나 시작할 수 있습니다.

    ![Visual Studio Code Azure 함수 로그 스트리밍 시작](media/LAB_AK_19-start-function-streaming.png)

### <a name="exercise-7---connect-iot-hub-to-the-azure-function"></a>연습 7 - Azure 함수에 IoT Hub 연결

이 연습에서는 설정 스크립트를 사용하여 만든 IoT Hub가 발생하는 이벤트를 이전 연습에서 만든 Azure 함수에 게시하도록 구성합니다. 그러면 앞에서 만든 디바이스 시뮬레이터의 원격 분석이 ADT 인스턴스로 라우팅됩니다.

1. 브라우저를 열고 [Azure Portal](https://portal.azure.com/)로 이동합니다.

1. **iot-az220-training-{사용자 ID}** IoT Hub로 이동합니다.

1. 왼쪽 탐색 영역에서 **이벤트** 를 선택합니다.

1. 이벤트 구독을 추가하려면 **+ 이벤트 구독** 을 클릭합니다.

1. **이벤트 구독 정보** 섹션의 **이름** 필드에 **device-telemetry** 를 입력합니다.

1. **Event Grid 스키마** 드롭다운에서 **Event Grid 스키마** 가 선택되어 있는지 확인합니다.

1. **항목 정보** 섹션에서 **항목 유형** 이 **IoT Hub** 로, **원본 리소스** 가 **iot-az220-training-{사용자 ID}** 로 설정되어 있는지 확인합니다.

1. **시스템 항목 이름** 필드에 **Twin-Topic** 을 입력합니다.

1. **EVENT TYPES** 섹션의 **이벤트 유형 필터** 드롭다운에서 **디바이스 원격 분석** 만 선택합니다.

1. **엔드포인트 정보** 섹션의 **엔드포인트 유형** 드롭다운에서 **Azure 함수** 를 선택합니다.

    UI가 업데이트되어 엔드포인트 선택 항목이 제공됩니다.

1. **엔드포인트** 필드에서 **엔드포인트 선택** 을 클릭합니다.

1. **Azure 함수 선택** 창의 **구독** 아래에서 올바른 구독이 선택되어 있는지 확인합니다.

1. **리소스 그룹** 에서 **rg-az220** 이 선택되어 있는지 확인합니다.

1. 함수 앱에서 **func-az220-hub2adt-training-{your-id}** 를 선택합니다.

1. **슬롯** 아래에 **프로덕션** 이 선택되어 있는지 확인합니다.

1. **함수** 아래에 **HubToAdtFunction** 이 선택되어 있는지 확인합니다.

1. 해당 엔드포인트를 선택하려면 **선택 확인** 을 클릭합니다.

1. 이제 지정된 엔드포인트가 **HubToAdtFunction** 인지 확인합니다.

    **이벤트 구독 만들기** 창의 **엔드포인트 정보** 섹션 **엔드포인트** 필드에 **HubToAdtFunction** 이 표시되어야 합니다.

1. 해당 이벤트 구독을 만들려면 **만들기** 를 클릭합니다.

    구독이 작성되고 나면 이전 연습에서 구성한 Azure Functions 로그 스트림에 메시지가 표시됩니다. Azure Functions 로그 스트림은 Event Grid에서 수신되는 원격 분석을 표시합니다. 또한 Azure Digital Twins에 연결하거나 트윈을 업데이트할 때 발생하는 모든 오류를 표시합니다.

    함수가 정상적으로 호출되면 표시되는 로그 출력은 다음과 같습니다.

    ```log
    2021-03-12T19:14:17.180 [Information] Executing 'HubToAdtFunction' (Reason='EventGrid trigger fired at 2021-03-12T19:14:17.1797847+00:00', Id=88d9f9e8-5cfa-4a20-a4cb-36e07a78acd6)
    2021-03-12T19:14:17.180 [Information] {
    "properties": {
        "sensorID": "S1",
        "fanAlert": "false",
        "temperatureAlert": "true",
        "humidityAlert": "true"
    },
    "systemProperties": {
        "iothub-connection-device-id": "sensor-th-0055",
        "iothub-connection-auth-method": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
        "iothub-connection-auth-generation-id": "637508617957275763",
        "iothub-enqueuedtime": "2021-03-12T19:14:16.824Z",
        "iothub-message-source": "Telemetry"
    },
    "body": "eyJ0ZW1wZXJhdHVyZSI6NjkuNDcsImh1bWlkaXR5Ijo5Ny44OX0="
    }
    2021-03-12T19:14:17.181 [Information] Azure digital twins service client connection created.
    2021-03-12T19:14:17.181 [Information] Device:sensor-th-0055 fanAlert is:False
    2021-03-12T19:14:17.181 [Information] Device:sensor-th-0055 temperatureAlert is:true
    2021-03-12T19:14:17.181 [Information] Device:sensor-th-0055 humidityAlert is:true
    2021-03-12T19:14:17.181 [Information] Device:sensor-th-0055 Temperature is:69.47
    2021-03-12T19:14:17.181 [Information] Device:sensor-th-0055 Humidity is:97.89
    2021-03-12T19:14:17.182 [Information] Executed 'HubToAdtFunction' (Succeeded, Id=88d9f9e8-5cfa-4a20-a4cb-36e07a78acd6, Duration=2ms)
    ```

    디지털 트윈 인스턴스를 찾을 수 없는 경우의 예제 로그는 다음과 같습니다.

    ```log
    2021-03-11T16:35:43.646 [Information] Executing 'HubToAdtFunction' (Reason='EventGrid trigger fired at 2021-03-11T16:35:43.6457834+00:00', Id=9f7a3611-0795-4da7-ac8c-0b380310f4db)
    2021-03-11T16:35:43.646 [Information] {
    "properties": {
        "sensorID": "S1",
        "fanAlert": "false",
        "temperatureAlert": "true",
        "humidityAlert": "true"
    },
    "systemProperties": {
        "iothub-connection-device-id": "sensor-th-0055",
        "iothub-connection-auth-method": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
        "iothub-connection-auth-generation-id": "637508617957275763",
        "iothub-enqueuedtime": "2021-03-11T16:35:43.279Z",
        "iothub-message-source": "Telemetry"
    },
    "body": "eyJ0ZW1wZXJhdHVyZSI6NjkuNzMsImh1bWlkaXR5Ijo5OC4wOH0="
    }
    2021-03-11T16:35:43.646 [Information] Azure digital twins service client connection created.
    2021-03-11T16:35:43.647 [Information] Device:sensor-th-0055 fanAlert is:False
    2021-03-11T16:35:43.647 [Information] Device:sensor-th-0055 temperatureAlert is:true
    2021-03-11T16:35:43.647 [Information] Device:sensor-th-0055 humidityAlert is:true
    2021-03-11T16:35:43.647 [Information] Device:sensor-th-0055 Temperature is:69.73
    2021-03-11T16:35:43.647 [Information] Device:sensor-th-0055 Humidity is:98.08
    2021-03-11T16:35:43.648 [Information] Executed 'HubToAdtFunction' (Succeeded, Id=9f7a3611-0795-4da7-ac8c-0b380310f4db, Duration=2ms)
    2021-03-11T16:35:43.728 [Error] Service request failed.
    Status: 404 (Not Found)

    Content:
    {"error":{"code":"DigitalTwinNotFound","message":"There is no digital twin instance that exists with the ID sensor-th-0055. Please verify that the twin id is valid and ensure that the twin is not deleted. See section on querying the twins http://aka.ms/adtv2query."}}

    Headers:
    Strict-Transport-Security: REDACTED
    traceresponse: REDACTED
    Date: Thu, 11 Mar 2021 16:35:43 GMT
    Content-Length: 267
    Content-Type: application/json; charset=utf-8
    ```

1. 브라우저의 **ADT Explorer** 인스턴스로 돌아와 그래프를 쿼리합니다.

    **fanAlert**, **temperatureAlert** 및 **humidityAlert** 속성이 업데이트된 상태로 표시되어야 합니다.

지금까지 디바이스(여기서는 디바이스 시뮬레이터)에서 ADT로의 데이터 수집 과정을 살펴보았습니다. 다음 연습에서는 원격 분석을 TSI(Time Series Insights)로 스트리밍하는 방법을 보여 줍니다.

### <a name="exercise-8---connect-adt-to-tsi"></a>연습 8 - TSI에 ADT 연결

이 연습에서는 개념 증명의 마지막 과정을 완료합니다. 즉, 시뮬레이터를 통해 Cheese Cave Device sensor-th-0055에서 전송되는 디바이스 원격 분석을 Azure Digital Twins를 통해 Time Series Insights로 스트리밍합니다.

> **참고**: 앞에서 이 랩용 설정 스크립트를 실행할 때 이 연습에서 사용할 Azure Storage 계정, Time Series Insights 환경 및 액세스 정책이 작성되었습니다.

이벤트 데이터는 다음과 같은 기본적인 흐름을 통해 ADT에서 TSI로 라우팅됩니다.

* ADT가 Event Hub(adt2func)에 알림 이벤트를 게시합니다.
* adt2func Event Hub의 이벤트에 의해 Azure 함수가 트리거되어 TSI용으로 새 이벤트를 만들고 파티션 키를 추가한 후 다른 Event Hub(func2tsi)에 새 이벤트를 게시합니다.
* TSI가 func2tsi Event Hub에 게시된 이벤트를 구독합니다.

이 흐름을 구현하려면 (ADT와 TSI 이외에) 다음 리소스를 만들어야 합니다.

* Azure 함수
* Event Hub 네임스페이스
* Event Hub 2개 - ADT에서 Azure 함수로 전송되는 이벤트용 Event Hub 및 Azure 함수가 TSI용으로 만드는 이벤트용 Event Hub

다음과 같은 여러 가지 용도로 Azure 함수를 사용할 수 있습니다.

* 디바이스별 원격 분석 메시지 형식(속성 이름, 데이터 형식 등)을 TSI용 단일 형식에 매핑. 따라서 여러 디바이스의 데이터를 통합 확인할 수 있을 뿐 아니라, TSI 솔루션이 솔루션의 다른 부분을 변경하는 현상도 방지할 수 있습니다.
* 다른 원본의 메시지 보강
* TSI 내에서 시계열 ID 속성으로 사용할 필드를 각 이벤트에 추가

#### <a name="task-1---create-event-hub-namespace"></a>작업 1 - Event Hub 네임스페이스 만들기

Event Hubs 네임스페이스는 DNS 통합 네트워크 엔드포인트와 IP 필터링, 가상 네트워크 서비스 엔드포인트 및 Private Link와 같은 다양한 액세스 제어 및 네트워크 통합 관리 기능을 제공하며, 여러 Event Hub 인스턴스(또는 kafka 용어의 항목) 중 하나에 대한 관리 컨테이너입니다. 이 네임스페이스 내에서 이 솔루션에 필요한 Event Hub 2개를 만듭니다.

1. Azure 계정 자격 증명을 사용하여 [portal.azure.com](https://portal.azure.com)에 로그인합니다.

1. Azure Portal 메뉴에서 **+ 리소스 만들기** 를 클릭합니다.

1. 검색 텍스트 상자에 **Event Hubs** 를 입력하고 검색 결과에서 **Event Hubs** 를 클릭합니다.

1. **Event Hub** 를 만들려면 **만들기** 를 클릭합니다.

    **네임스페이스 만들기** 페이지가 열립니다.

1. **네임스페이스 만들기** 블레이드의 **구독** 드롭다운에서 이 과정에 사용 중인 Azure 구독이 선택되어 있는지 확인합니다.

1. **리소스 그룹** 오른쪽에서 드롭다운을 열고 **rg-az220** 을 클릭합니다.

1. **네임스페이스 이름** 필드에 **adt-az220-training-{사용자 ID}** 를 입력합니다.

    이 리소스는 공개적으로 액세스 가능하며, 고유한 이름을 지정해야 합니다.

1. **위치** 오른쪽의 드롭다운 목록을 열고 리소스 그룹용으로 선택한 것과 같은 위치를 선택합니다.

1. **가격 책정 계층** 오른쪽의 드롭다운 목록을 열고 **표준** 을 선택합니다.

    > **팁**: 이 연습에서는 **기본** 계층을 사용해도 됩니다. 그러나 대다수 프로덕션 시나리오에서는 **표준** 을 선택하는 것이 더 좋습니다.
    >
    > * **기본**
    >   * 소비자 그룹 1개
    >   * 조정된 연결 100개
    >   * 수신 이벤트 - 1백만 개당 $0.028(이 문서 작성 시점의 가격)
    >   * 메시지 보존 - 1일
    > * **Standard**
    >   * 소비자 그룹 20개
    >   * 조정된 연결 1000개
    >   * 수신 이벤트 - 1백만 개당 $0.028(이 문서 작성 시점의 가격)
    >   * 메시지 보존 - 1일
    >   * 추가 저장 - 최대 7일
    >   * 게시자 정책

1. **처리량 단위** 오른쪽의 선택 항목을 **1** 로 유지합니다.

    > **팁**: Event Hubs의 처리량 용량은 처리량 단위로 제어됩니다. 처리량 단위는 미리 구입한 용량의 단위입니다. 단일 처리량을 사용하면 다음을 수행할 수 있습니다.
    >
    > * 수신: 초당 최대 1MB 또는 초당 1,000회 이벤트(둘 중 빠른 쪽 적용).
    > * 송신: 초당 최대 2MB 또는 4096개의 이벤트.

1. **자동 팽창 사용** 을 선택하지 않은 상태로 둡니다.

    자동 팽창은 트래픽이 할당된 처리량 단위의 용량을 초과할 때 표준 계층 이벤트 허브 네임스페이스에 할당된 처리량 단위 수를 자동으로 스케일링합니다. 네임스페이스가 자동으로 크기 조정되는 제한을 지정할 수 있습니다.

1. 입력한 데이터의 유효성 검사를 시작하려면 **검토 + 만들기** 를 클릭합니다.

1. 유효성 검사가 정상적으로 완료되면 **만들기** 를 클릭합니다.

    잠시 후 리소스가 배포됩니다. **리소스로 이동** 을 클릭합니다.

이 네임스페이스에는 디지털 트윈 원격 분석을 Azure 함수와 통합하는 데 사용되는 Event Hub, 그리고 Azure 함수의 출력을 가져와 Time Series Insights와 통합하는 Event Hub가 하나씩 포함됩니다.

#### <a name="task-2---add-an-event-hub-for-adt"></a>작업 2 - ADT용 Event Hub 추가

이 작업에서는 트윈 원격 분석 이벤트를 구독하여 Azure 함수로 전달하는 Event Hub를 만듭니다.

1. **adt-az220-training-{사용자 ID}** 네임스페이스의 **개요** 페이지에서 **+ Event Hub** 를 클릭합니다.

1. **이벤트 허브 만들기** 페이지의 **이름** 아래에 **evh-az220-adt2func** 를 입력합니다.

    > **참고**: 이벤트 허브는 전역적으로 고유한 네임스페이스 내에서 범위가 지정되므로 이벤트 허브 이름 자체는 전역적으로 고유할 필요가 없습니다.

1. **파티션 수** 값은 **1** 로 유지합니다.

    > **팁**: 파티션은 데이터 조직 메커니즘으로서 애플리케이션 소비에 필요한 다운스트림 병렬 처리와 관련되어 있습니다. Event Hub의 파티션 수는 예상되는 동시 판독기의 수와 직접적으로 관련이 있습니다.

1. **메시지 보존** 값은 **1** 로 유지합니다.

    > **팁**: 이벤트의 보존 기간입니다. 보존 기간을 1일에서 7일 사이로 설정할 수 있습니다.

1. **캡처** 에서 값을 **끄기** 로 설정된 상태로 유지합니다.

    > **팁**: Azure Event Hubs Capture를 사용하면 시간 또는 크기 간격을 지정할 수 있는 유연성이 추가되어 Event Hub의 스트리밍 데이터를 선택한 Azure Blob 스토리지 또는 선택한 Azure Data Lake Store 계정으로 자동 전달할 수 있습니다. 캡처는 빠르게 설정할 수 있으며 실행을 위한 관리 비용이 없고 Event Hubs 처리량 단위에 따라 크기가 자동으로 조정됩니다. Event Hubs 캡처는 스트리밍 데이터를 Azure에 로드하는 가장 쉬운 방법이며 데이터 캡처보다 데이터 처리에 집중할 수 있게 해줍니다.

1. Event Hub를 만들려면 **만들기** 를 클릭합니다.

    잠시 후에 Event Hub가 작성되고 Event Hubs 네임스페이스 **개요** 가 표시됩니다. 필요한 경우 페이지 아래쪽으로 스크롤합니다. 그러면 **evh-az220-adt2func** Event Hub가 표시됩니다.

#### <a name="task-3---add-an-authorization-rule-to-the-event-hub"></a>작업 3 - Event Hub에 권한 부여 규칙 추가

각 Event Hubs 네임스페이스와 각 Event Hubs 엔터티(이벤트 허브 인스턴스 또는 Kafka 토픽)에는 규칙으로 구성된 공유 액세스 권한 부여 정책이 있습니다. 네임스페이스 수준에서 정책은 개별 정책 구성에 관계 없이 네임스페이스 내부의 모든 엔터티에 적용됩니다. 각각의 권한 부여 정책 규칙의 경우 이름, 범위 및 권한 등, 3가지 정보를 결정합니다. 이름은 해당 범위 내에서 고유한 이름입니다. 범위는 해당 리소스의 URI입니다. Event Hubs 네임스페이스의 경우 범위는 `https://evhns-az220-training-{your-id}.servicebus.windows.net/` 등과 같이 FQDN(정규화된 도메인 이름)입니다.

정책 규칙에 따른 권한은 다음의 조합일 수 있습니다.

* Send – 엔터티에 메시지를 보낼 수 있는 권한을 부여합니다.
* Listen -엔터티를 수신 대기하거나 받을 수 있는 권한을 부여합니다.
* Manage – 엔터티 만들기 및 삭제를 포함하여 네임스페이스의 토폴로지를 관리할 수 있는 권한을 부여합니다.

이 작업에서는 **수신 대기** 및 **보내기** 권한이 있는 권한 부여 규칙을 만듭니다.

1. **adt-az220-training-{사용자 ID}** 네임스페이스의 **개요** 페이지에서 **evh-az220-adt2func** Event Hub를 클릭합니다.

1. 왼쪽 탐색 영역의 **설정** 에서 **공유 액세스 정책** 을 클릭합니다.

    이 Event Hub와 관련된 빈 정책 목록이 표시됩니다.

1. 새 권한 부여 규칙을 만들려면 **+ 추가** 를 클릭합니다.

1. **SAS 정책 추가** 창의 정책 이름에 **ADTHubPolicy** 를 입력합니다.

1. 권한 목록에서 **보내기** 와 **수신 대기** 만 선택합니다.

1. **만들기** 를 클릭하여 권한 부여 규칙을 만듭니다.

    잠시 후에 창이 닫히고 정책 목록이 새로 고쳐집니다.

1. 권한 부여 규칙의 기본 연결 문자열을 검색하려면 목록에서 **ADTHubPolicy** 를 클릭합니다.

    **SAS 정책: ADTHubPolicy** 창이 열립니다.

1. **연결 문자열 기본 키** 값을 복사하여 새 텍스트 파일 **telemetry-function.txt** 에 추가합니다.

1. **SAS 정책: ADTHubPolicy** 창을 닫습니다.

#### <a name="task-4---add-an-event-hub-endpoint-to-the-adt-instance"></a>작업 4 - ADT 인스턴스에 Event Hub 엔드포인트 추가

Event Hub를 만들었으므로 엔드포인트로 추가해야 합니다. 그러면 ADT 인스턴스가 해당 엔드포인트를 출력으로 사용하여 이벤트를 보낼 수 있습니다.

1. **iot-az229-training-{사용자 ID}** 인스턴스로 이동합니다.

1. 왼쪽 탐색 영역의 **출력 연결** 에서 **엔드포인트** 를 클릭합니다.

    엔드포인트 목록이 표시됩니다.

1. 새 엔드포인트를 추가하려면 **+ 엔드포인트 만들기** 를 클릭합니다.

1. **엔드포인트 만들기** 창의 **이름** 에 **eventhub-endpoint** 를 입력합니다.

1. **엔드포인트 유형** 에서 **Event Hub** 를 선택합니다.

    Event Hub 정보를 지정하는 필드가 포함되어 UI가 업데이트됩니다.

1. **구독** 드롭다운에서 이 과정에 사용하려는 Azure 구독이 선택되어 있는지 확인합니다.

1. **Event Hub 네임스페이스** 드롭다운에서 **adt-az220-training-{사용자 ID}** 를 선택합니다.

1. **Event Hub** 드롭다운에서 **evh-az220-adt2func** 를 선택합니다.

1. **권한 부여 규칙** 드롭다운에서 **ADTHubPolicy** 를 선택합니다.

   이 규칙은 앞에서 만든 권한 부여 규칙입니다.

1. 엔드포인트를 만들려면 **저장** 을 클릭합니다.

    창이 닫히고 잠시 후에 새 엔드포인트가 포함되어 엔드포인트 목록이 업데이트됩니다.

#### <a name="task-5---add-a-route-to-send-telemetry-to-the-event-hub"></a>작업 5 - Event Hub에 원격 분석을 전송하기 위한 경로 추가

ADT 인스턴스에 Event Hub 엔드포인트를 추가했으므로 이제 트윈 원격 분석 이벤트를 해당 엔드포인트로 전송하는 경로를 만들어야 합니다.

1. 왼쪽 탐색 영역의 **출력 연결** 에서 **이벤트 경로** 를 클릭합니다.

    기존 경로 목록이 표시됩니다.

1. 새 이벤트 경로를 추가하려면 **+ 이벤트 경로 만들기** 를 클릭합니다.

1. **이벤트 경로 만들기** 창의 **이름** 에 **eventhub-telemetryeventroute** 를 입력합니다.

1. **엔드포인트** 드롭다운에서 **eventhub-endpoint** 를 선택합니다.

1. **이벤트 경로 필터 추가** 아래에서 **고급 편집기** 를 사용하지 않도록 설정합니다.

    고급 편집기는 특정 필터링 식 입력을 지원하는데, 이 작업에서는 UI만으로도 충분합니다.

1. **이벤트 유형** 드롭다운에서 **원격 분석** 만 선택합니다.

    **필터** 필드에 생성된 필터 식이 표시되는 것을 볼 수 있습니다.

1. 이벤트 경로를 만들려면 **저장** 을 클릭합니다.

    창이 닫히고 잠시 후에 새 경로가 포함되어 이벤트 경로 목록이 업데이트됩니다.

#### <a name="task-6---create-tsi-event-hub-and-policy"></a>작업 6 - TSI Event Hub 및 정책 만들기

이번에는 Azure CLI를 사용하여 Event Hub 및 권한 부여 규칙을 만듭니다.

1. 명령 프롬프트로 돌아와서 다음 명령을 입력해 세션에 아직 로그인되어 있는지 확인합니다.

    ```powershell
    az account list
    ```

    `az login`을 실행하라는 메시지가 표시되면 az login을 실행하여 로그인합니다.

1. Azure 함수와 TSI 간의 Event Hub를 만들려면 다음 명령을 입력합니다.

    ```powershell
    az eventhubs eventhub create --name evh-az220-func2tsi --resource-group rg-az220 --namespace-name evhns-az220-training-{your-id}
    ```

    **{사용자 ID}** 는 적절한 값으로 바꿔야 합니다.

1. 새 Event Hub에 대한 수신 대기 및 보내기 권한이 있는 권한 부여 규칙을 만들려면 다음 명령을 입력합니다.

    ```powershell
    az eventhubs eventhub authorization-rule create --rights Listen Send --resource-group rg-az220  --eventhub-name evh-az220-func2tsi --name TSIHubPolicy --namespace-name evhns-az220-training-{your-id}
    ```

    **{사용자 ID}** 는 적절한 값으로 바꿔야 합니다.

1. 권한 부여 규칙의 기본 연결 문자열을 검색하려면 다음 명령을 입력합니다.

    ```powershell
    az eventhubs eventhub authorization-rule keys list --resource-group rg-az220 --eventhub-name evh-az220-func2tsi --name TSIHubPolicy --namespace-name evhns-az220-training-{your-id} --query primaryConnectionString -o tsv
    ```

    **{사용자 ID}** 는 적절한 값으로 바꿔야 합니다.

    다음과 유사하게 출력됩니다.

    ```text
    Endpoint=sb://evhns-az220-training-dm030821.servicebus.windows.net/;SharedAccessKeyName=TSIHubPolicy;SharedAccessKey=x4xItgUG6clhGR9pZe/U6JqrNV+drIfu1rlvYHEdk9I=;EntityPath=evh-az220-func2tsi
    ```

1. **연결 문자열 기본 키** 값을 복사하여 텍스트 파일 **telemetry-function.txt** 에 추가합니다.

    다음 작업에서 연결 문자열 2개가 필요합니다.

#### <a name="task-7---add-the-endpoint-addresses-as-app-settings-for-azure-function"></a>작업 7 - Azure 함수용 앱 설정으로 엔드포인트 주소 추가

Azure 함수가 Event Hub에 연결하려면 적절한 권한이 있는 정책의 연결 문자열에 액세스할 수 있어야 합니다. 이 시나리오에서는 Event Hub 2개가 사용됩니다. 그 중 하나는 ADT의 이벤트를 게시하는 허브이고, 다른 하나는 Azure 함수에서 변환한 데이터를 TSI에 게시하는 허브입니다.

1. Event Hub 권한 부여 규칙 연결 문자열을 환경 변수로 제공하려면 Azure Portal에서 **func-az220-hub2adt-training-{사용자 ID}** 인스턴스로 이동합니다.

1. 왼쪽 탐색 영역의 **설정** 에서 **구성** 을 클릭합니다.

1. **구성** 페이지의 **애플리케이션 설정** 탭에서 나와 있는 현재 **애플리케이션 설정** 을 검토합니다.

    앞에서 CLI를 통해 추가한 **ADT_SERVICE_URL** 이 목록에 포함되어 있어야 합니다.

1. adt2func 규칙 연결 문자열용 환경 변수를 추가하려면 **+ 새 애플리케이션 설정** 을 클릭합니다.

1. **애플리케이션 설정 추가/편집** 창의 **이름** 필드에 **ADT_HUB_CONNECTIONSTRING** 을 입력합니다.

1. **값** 필드에 이전 작업에서 **telemetry-function.txt** 파일에 저장해 두었고 `EntityPath=evh-az220-adt2func`로 끝나는 권한 부여 규칙 연결 문자열 값을 입력합니다.

    값은 다음과 비슷합니다. `Endpoint=sb://evhns-az220-training-dm030821.servicebus.windows.net/;SharedAccessKeyName=ADTHubPolicy;SharedAccessKey=fHnhXtgjRGpC+rR0LFfntlsMg3Z/vjI2z9yBb9MRDGc=;EntityPath=evh-az220-adt2func`

1. 창을 닫으려면 **확인** 을 클릭합니다.

    > **참고**: 이 설정은 아직 저장되지 않았습니다.

1. func2tsi 규칙 연결 문자열용 환경 변수를 추가하려면 **+ 새 애플리케이션 설정** 을 클릭합니다.

1. **애플리케이션 설정 추가/편집** 창의 **이름** 필드에 **TSI_HUB_CONNECTIONSTRING** 을 입력합니다.

1. **값** 필드에 이전 작업에서 **telemetry-function.txt** 파일에 저장해 두었고 `EntityPath=evh-az220-func2tsi`로 끝나는 권한 부여 규칙 연결 문자열 값을 입력합니다.

    값은 다음과 비슷합니다. `Endpoint=sb://evhns-az220-training-dm030821.servicebus.windows.net/;SharedAccessKeyName=TSIHubPolicy;SharedAccessKey=x4xItgUG6clhGR9pZe/U6JqrNV+drIfu1rlvYHEdk9I=;EntityPath=evh-az220-func2tsi`

1. 창을 닫으려면 **확인** 을 클릭합니다.

    > **참고**: 이 설정은 아직 저장되지 않았습니다.

1. 새 설정 2개를 모두 저장하려면 **저장**, **계속** 을 차례로 클릭합니다.

    > **참고**: 애플리케이션 설정을 변경하면 함수가 다시 시작됩니다.

#### <a name="task-8---review-a-telemetry-azure-function"></a>작업 8 - 원격 분석 Azure 함수 검토

이 작업에서는 두 번째 Azure 함수를 검토합니다. 이 함수는 TSI용 대체 형식에 디바이스 원격 분석 메시지를 매핑합니다. 이 방식을 사용하는 경우 TSI 솔루션을 변경하지 않고도 디바이스 원격 분석 형식의 변경을 처리할 수 있습니다.

1. Visual Studio Code에서 **Contoso.AdtFunctions** 프로젝트를 엽니다.

1. **TelemetryFunction.cs** 파일을 엽니다.

1. **Run** 메서드 정의를 찾아서 코드를 검토합니다.

    ```csharp
    public static class TelemetryFunction
    {
        [FunctionName("TelemetryFunction")]
        public static async Task Run(
            [EventHubTrigger("evh-az220-adt2func", Connection = "ADT_HUB_CONNECTIONSTRING")] EventData[] events,
            [EventHub("evh-az220-func2tsi", Connection = "TSI_HUB_CONNECTIONSTRING")] IAsyncCollector<string> outputEvents,
            ILogger log)
        {
            var exceptions = new List<Exception>();

            foreach (EventData eventData in events)
            {
                try
                {
                    // main processing code here
                }
                catch (Exception e)
                {
                    // We need to keep processing the rest of the batch - capture this exception and continue.
                    // Also, consider capturing details of the message that failed processing so it can be processed again later.
                    exceptions.Add(e);
                }
            }

            // Once processing of the batch is complete, if any messages in the batch failed processing throw an exception so that there is a record of the failure.

            if (exceptions.Count > 1)
                throw new AggregateException(exceptions);

            if (exceptions.Count == 1)
                throw exceptions.Single();
        }
    }
    ```

    **Run** 메서드 정의를 잠시 살펴보겠습니다. **events** 매개 변수는 **EventHubTrigger **특성을 사용합니다. 이 특성의 생성자는 Event Hub의 이름, 소비자 그룹의** 선택적** 이름(생략하는 경우 **$Default** 가 사용됨), 그리고 연결 문자열을 포함하는 앱 설정의 이름을 받습니다. 그러면 Event Hub 이벤트 스트림으로 전송된 이벤트에 응답하는 함수 트리거가 구성됩니다. **events** 는 EventData 배열로 정의되므로 이벤트 배치를 입력할 수 있습니다.

    > **팁** **EventHubTrigger** 에 대해 자세히 알아보려면 다음 리소스를 검토하세요. [Azure Functions의 Azure Event Hubs 트리거](https://docs.microsoft.com/azure/azure-functions/functions-bindings-event-hubs-trigger?tabs=csharp)

    다음 매개 변수인 **outputEvents** 에는 **EventHub** 특성이 있습니다. 이 특성의 생성자는 Event Hub의 이름, 그리고 연결 문자열이 포함되어 있는 앱 설정의 이름을 가져옵니다. **outputEvents** 변수에 추가하는 데이터는 연결된 Event Hub에 게시됩니다.

    이 함수는 이벤트 배치를 처리하므로, 예외를 포함할 컬렉션을 만들어서 오류를 처리할 수 있습니다. 그러면 함수가 배치의 각 이벤트에 대해 반복 실행되어 예외를 catch한 후 컬렉션에 추가합니다. 메서드의 끝부분으로 넘어가면 예외가 여러 개인 경우 컬렉션과 함께 **AggregaeException** 이 작성되고 생성된 예외가 한 개인 경우 단일 예외가 throw되는 것을 볼 수 있습니다.

1. 이벤트가 Cheese Cave Device 원격 분석을 포함하는지 확인하는 코드를 검토하려면 `// REVIEW check telemetry below here` 주석을 찾아서 그 아래의 다음 코드를 검토합니다.

    ```csharp
    if ((string)eventData.Properties["cloudEvents:type"] == "microsoft.iot.telemetry" &&
        (string)eventData.Properties["cloudEvents:dataschema"] == "dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave_device;1")
    {
        // REVIEW TSI Event creation below here
    }
    else
    {
        log.LogInformation($"Not Cheese Cave Device telemetry");
        await Task.Yield();
    }
    ```

    이 코드는 현재 이벤트가 Cheese Cave Device ADT 트윈에서 보낸 원격 분석인지를 확인하여 아닌 경우 Cheese Cave Device 트윈의 원격 분석이 아니라는 내용을 로깅합니다. 그런 다음 메서드를 비동기식으로 강제 완료합니다. 이렇게 메서드를 강제 완료하면 리소스 사용 효율성을 높일 수 있습니다.

    > **팁**: `await Task.Yield();`의 사용에 대해 자세히 알아보려면 다음 리소스를 검토하세요.
    > * [Task.Yield 메서드](https://docs.microsoft.com/dotnet/api/system.threading.tasks.task.yield?view=net-5.0)

1. 이벤트를 처리하고 TSI에 대해 메시지를 생성하는 코드를 검토하려면 `// REVIEW TSI Event creation below here` 주석을 찾아서 그 아래의 다음 코드를 검토합니다.

    ```csharp
    // The event is Cheese Cave Device Telemetry
    string messageBody = Encoding.UTF8.GetString(eventData.Body.Array, eventData.Body.Offset, eventData.Body.Count);
    JObject message = (JObject)JsonConvert.DeserializeObject(messageBody);

    var tsiUpdate = new Dictionary<string, object>();
    tsiUpdate.Add("$dtId", eventData.Properties["cloudEvents:source"]);
    tsiUpdate.Add("temperature", message["temperature"]);
    tsiUpdate.Add("humidity", message["humidity"]);

    var tsiUpdateMessage = JsonConvert.SerializeObject(tsiUpdate);
    log.LogInformation($"TSI event: {tsiUpdateMessage}");

    await outputEvents.AddAsync(tsiUpdateMessage);
    ```

    **eventData.Body** 는 단순한 배열이 아닌 **ArraySegment** 로 정의되므로 **messageBody** 가 포함된 기본 배열 부분을 추출한 후에 역직렬화해야 합니다.

    > **팁**: **ArraySegment** 에 대해 자세히 알아보려면 다음 리소스를 검토하세요.
    > * [ArraySegment&lt;T&gt; 구조체](https://docs.microsoft.com/dotnet/api/system.arraysegment-1?view=net-5.0)

    그리고 나면 키/값 쌍을 저장할 **Dictionary** 가 인스턴스화됩니다. TSI 이벤트 내에서 전송되는 속성은 이 키/값 쌍으로 구성됩니다. (`adt-az220-training-dm030821.api.eus.digitaltwins.azure.net/digitaltwins/sensor-th-0055`와 같은 정규화된 트윈 ID를 포함하는) **cloudEvents:source** 속성이 **\$dtId** 키에 할당된 것을 볼 수 있습니다. 이 키가 특별한 이유는, 설정 과정에서 생성된 Time Series Insights 환경이 **\$dtId** 를 **시계열 ID 속성** 으로 사용하기 때문입니다.

    **temperature** 및 **humidity** 값은 메시지에서 추출되어 TSI 업데이트에 추가됩니다.

    그런 후에 업데이트가 JSON으로 직렬화된 후 **outputEvents** 에 추가되며, 그러면 Event Hub에 업데이트가 게시됩니다.

    이제 함수를 게시할 수 있습니다.

#### <a name="task-9---publish-the-function-app-to-azure"></a>작업 9 - Azure에 함수 앱 게시

1. Visual Studio Code용 Azure Functions 확장에서 **함수 앱에 배포** 를 선택합니다.

    ![Visual Studio Code 함수 앱에 배포](media/LAB_AK_19-deploy-to-function-app.png)

1. 메시지가 표시되면 다음 항목을 선택합니다.

    * **구독 선택**: 메시지가 표시되면 이 과정에서 사용하고 있는 구독을 선택합니다.
    * **Azure에서 함수 앱 선택**: **func-az220-hub2adt-training-{사용자 ID}** 를 선택합니다.

    배포를 확인하라는 메시지가 표시되면 **배포** 를 클릭합니다.

    함수가 컴파일되며, 컴파일이 정상적으로 완료되면 함수가 배포됩니다. 어느 정도 시간이 걸릴 수 있습니다.

1. 배포가 완료되면 다음 메시지가 표시됩니다.

    ![Visual Studio Code 배포 완료 - 로그 스트리밍 선택](media/LAB_AK_19-function-stream-logs.png)

    **로그 스트리밍** 를 선택하고 애플리케이션 로깅을 사용할지 확인하라는 대화 상자에서 **예** 를 클릭합니다.

    그러면 **출력** 창에 배포된 함수의 로그 스트림이 표시됩니다. 이 스트림은 2시간 후에 만료됩니다. 상태 정보도 몇 가지 표시됩니다. 그러나 함수 앱을 시작할 때까지는 함수 자체에서 진단 정보가 제공되지는 않습니다. 진단 정보에 대해서는 다음 연습에서 살펴봅니다.

    Visual Studio Code에서 Azure 함수를 마우스 오른쪽 단추로 클릭하고 **스트리밍 로그 시작** 또는 **스트리밍 로그 중지** 를 선택하면 언제든지 스트리밍을 중지하거나 시작할 수 있습니다.

    ![Visual Studio Code Azure 함수 로그 스트리밍 시작](media/LAB_AK_19-start-function-streaming.png)

#### <a name="task-10---configure-tsi"></a>작업 10 - TSI 구성

1. 브라우저에서 Azure Portal에 연결한 후 **tsi-az220-training-{사용자 ID}** 리소스를 찾습니다.

    > **참고**: 이 리소스는 설정 스크립트에 의해 작성된 것입니다. 설정 스크립트를 실행하지 않았다면 지금 실행하세요. 기존 리소스에는 아무런 영향이 없습니다.

1. **개요** 창의 **필수** 섹션에서 **시계열 ID 속성** 필드와 값 **$dtid** 를 찾습니다. TSI 환경을 만들 때 지정된 이 값은 TSI로 스트리밍하는 이벤트 데이터의 필드 값과 일치해야 합니다.

    > **중요**: **시계열 ID 속성** 값은 TSI 환경을 만들 때 지정되며, 그 이후에는 변경할 수 없습니다.

1. 왼쪽 탐색 영역의 **설정** 에서 **이벤트 원본** 을 클릭합니다.

    **이벤트 원본** 목록이 표시됩니다. 현재 해당 목록은 비어 있습니다.

1. 새 이벤트 원본을 추가하려면 **+ 추가** 를 클릭합니다.

1. **새 이벤트 원본** 창의 **이벤트 원본 이름** 에 **adt-telemetry** 를 입력합니다.

1. **원본** 에서 **이벤트 허브** 를 선택합니다.

1. **가져오기 옵션** 에서 **사용 가능한 구독의 이벤트 허브 사용** 을 선택합니다.

1. **구독 ID** 에서 이 과정에 사용 중인 구독을 선택합니다.

1. **Event Hub 네임스페이스** 에서 **evhns-az220-training-{사용자 ID}** 를 선택합니다.

1. **Event Hub 이름** 에서 **evh-az220-func2tsi** 를 선택합니다.

1. **Event Hub 정책 값** 에서 **TSIHubPolicy** 를 선택합니다.

    읽기 전용 필드인 **이벤트 허브 정책 키** 에 값이 자동으로 입력됩니다.

1. **Event Hub 소비자 그룹** 에서 **$Default** 를 선택합니다.

    > **팁**: 여기서는 **evh-az220-func2tsi** Event Hub의 이벤트 읽기 권한자가 한 명뿐이므로 **$Default** 소비자 그룹을 선택하면 됩니다. 읽기 권한자를 더 추가하려면 읽기 권한자당 소비자 그룹을 하나씩 사용하는 것이 좋습니다. 소비자 그룹은 Event Hub에 작성됩니다.

1. **시작 시간** 에서 **내 모든 데이터** 를 선택합니다.

    그 밖에도 프로덕션 환경에 적합한 **지금 시작(기본값)** 과 같은 여러 옵션이 있는 것을 볼 수 있습니다.

1. **이벤트 Serialization 형식** 에서 읽기 전용 값이 **JSON** 인지 확인합니다.

1. **타임스탬프 속성 이름** 값은 비워 둡니다.

    > **팁**: 이 속성은 이벤트 타임스탬프로 사용해야 하는 이벤트 속성의 이름을 지정합니다. 이 속성을 지정하지 않으면 이벤트 원본 내의 이벤트 인큐 시간이 이벤트 타임스탬프로 사용됩니다.

1. 이벤트 원본을 만들려면 **저장** 을 클릭합니다.

#### <a name="task-11---visualize-the-telemetry-data-in-time-series-insights"></a>작업 11 - Time Series Insights에서 원격 분석 데이터 시각화

이제 데이터가 Time Series Insights 인스턴스로 이동하여 분석할 준비가 되어야 합니다. 다음 단계에 따라 들어오는 데이터를 탐색합니다.

1. 브라우저에서 **tsi-az220-training-{사용자 ID}** 리소스의 **개요** 창으로 돌아옵니다.

1. **TSI 탐색기** 로 이동하려면 **TSI 탐색기로 이동** 을 클릭합니다.

    브라우저의 새 탭에서 **TSI 탐색기** 가 열립니다.

1. 탐색기의 왼쪽에는 Azure Digital Twins가 표시됩니다.

    ![TSI 탐색기 데이터](media/LAB_AK_19-tsi-explorer.png)

1. 그래프에 원격 분석을 추가하려면 트윈을 클릭하고 **EventCount**, **humidity** 및 **temperature** 를 선택합니다.

    적절한 시간 범위를 선택하면 다음과 같이 데이터가 표시됩니다.

    ![시계열 데이터가 표시된 TSI 탐색기](media/LAB_AK_19-tsi-explorer-data.png)

    > **참고**: 충분한 데이터가 표시되려면 다소 시간이 걸릴 수 있습니다.

Time Series Insights에서는 기본적으로 디지털 트윈이 단순 계층 구조로 저장됩니다. 그러나 여러 수준이 포함된 조직의 계층 구조와 모델 정보로 디지털 트윈을 보강할 수 있습니다. Azure Digital Twins에 이미 저장된 모델 및 그래프 데이터를 사용하여 이 정보를 자동으로 제공하는 사용자 지정 논리를 작성할 수 있습니다.

축하합니다. 이제 디바이스 원격 분석 데이터를 Time Series Insights에 전달하고 있습니다.
