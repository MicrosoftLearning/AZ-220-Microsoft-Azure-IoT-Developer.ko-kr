---
ms.openlocfilehash: 8f8fa02137ff2e71ac82349a59d0eca8998f3c25
ms.sourcegitcommit: 5e9f89d47b27285feaf13cacfa097ddd4b888a90
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/14/2022
ms.locfileid: "137893074"
---
# <a name="az-220-microsoft-azure-iot-developer"></a>AZ-220: Microsoft Azure IoT Developer

- **[최신 학생 핸드북 및 AllFiles 콘텐츠 다운로드](../../releases/latest)**
- **MCT이신가요?** - [MCT를 위한 GitHub 사용자 가이드](https://microsoftlearning.github.io/MCT-User-Guide/)를 살펴보세요.
- **랩 지침을 수동으로 빌드해야 하나요?** - 지침은 [MicrosoftLearning/Docker-Build](https://github.com/MicrosoftLearning/Docker-Build) 리포지토리에서 확인할 수 있습니다.
- **[랩 VM에 설치된 소프트웨어](lab.md)**

## <a name="what-are-we-doing"></a>Microsoft의 역할

- 이 과정을 지원하려면 과정 콘텐츠를 자주 업데이트하여, 과정에서 사용하는 Azure 서비스를 이용해 콘텐츠를 최신 상태로 유지해야 합니다.  과정 작성자와 MCT 사이의 개방된 기여를 통해 Azure 플랫폼의 변경 내용을 적용하여 콘텐츠를 최신 상태로 유지할 수 있도록, 랩 지침과 랩 파일을 GitHub에 게시합니다.

- 이를 통해 이전에 경험하지 못한 방식으로 랩에 대한 협업을 경험하기 바랍니다. Azure가 변경하고 실시간 제공 중에 이를 먼저 발견하면 랩 소스에서 바로 개선 작업을 진행합니다.  동료 MCT를 도와주세요.

## <a name="how-should-i-use-these-files-relative-to-the-released-moc-files"></a>이러한 파일을 릴리스된 MOC 파일과 관련하여 어떻게 사용해야 하나요?

- 강사 핸드북과 PowerPoint는 여전히 과정 콘텐츠 교육의 기본 소스입니다.

- GitHub에서 이러한 파일은 학생 핸드북과 함께 사용하도록 설계되었지만, 중앙 리포지토리 역할을 하는 GitHub에 있기 때문에 MCT와 과정 작성자는 최신 랩 파일의 소스를 공유할 수 있습니다.

- 강사는 전달이 수행될 때마다 GitHub를 확인하여 최신 Azure 서비스를 지원하기 위한 변경 내용이 적용되었는지 확인하고, 전달할 최신 파일을 확보하는 것이 좋습니다.

## <a name="what-about-changes-to-the-student-handbook"></a>학생 핸드북의 변경 내용은 어떻게 해야 할까요?

- 학생 핸드북을 분기별로 검토하고 필요하다면 일반 MOC 릴리스 채널을 통해 업데이트합니다.

## <a name="how-do-i-contribute"></a>기고하려면 어떻게 해야 하나요?

- 과정 랩의 업데이트는 지속적으로 개발되고 있습니다. 이러한 업데이트의 일환으로 사소한 변경이 구현되는 동시에 전역 업데이트 개발도 진행되고 있습니다. 콘텐츠를 제공하려는 경우 끌어오기 요청을 작성하기 전에 문제를 제기하여 변경을 제안하는 것이 좋습니다.  

- 모든 MCT는 GitHub 리포지토리의 코드 또는 콘텐츠에 대한 끌어오기 요청을 제출(Master 분기 대상)할 수 있습니다. Microsoft와 과정 작성자는 콘텐츠 및 랩 코드 변경을 선별하고 필요에 따라 포함합니다. 랩 진행을 어렵게 만드는 문제가 확인되면 즉시 문제를 제기해 주시기 바랍니다.

- MCT는 버그, 변경 사항, 개선 사항 및 아이디어를 제출할 수 있습니다. 우리보다 먼저 새 Azure 기능을 찾으셨나요? 새로운 데모를 제출해 주세요!

## <a name="how-do-i-use-these-labs-for-self-study"></a>자가 학습용으로 이러한 랩을 사용하려면 어떻게 해야 하나요?

이러한 랩은 강사가 진행하는 AZ-220 과정의 도우미 자료로 작성되었습니다.  랩을 자가 학습용 지침 도구로 활용할 수 있도록 제작 과정에서 각 단계의 설명을 포함하기 위해 최선을 다했지만, 랩을 적절하게 완료하기 위한 모든 요구 사항이 작성된 것은 아닙니다.  예를 들어 강의실에서는 수강생이 비교적 제한 수준이 낮은 Azure 구독에 액세스할 수 있으므로 랩에 사용되는 리소스 공급자의 전체 목록은 제공되지 않습니다.  즉, 기능이 제한되는 구독(예: [Azure Policy](https://docs.microsoft.com/azure/governance/policy/overview) 제한이 적용되는 구독)에서 수행하는 랩 단계는 실패할 수도 있습니다.  또한 강사 진행 환경 외부에서는 일대일 랩 지원이 제공되지 않으므로 일대일 랩 지원 요청 관련 문제는 제출하지 마시기 바랍니다.

Microsoft Learn에서는 샌드박스 랩 환경이 포함된 다양한 IoT용 [학습 경로](https://docs.microsoft.com/en-us/learn/browse/?resource_type=learning%20path&products=azure-iot&roles=developer)를 제공합니다.  IoT 관련 자가 학습을 진행하려는 대다수 수강생은 이러한 학습 경로를 진행하는 것이 더 적절합니다.

## <a name="notes"></a>참고

### <a name="classroom-materials"></a>수업 자료

MCT와 파트너는 이러한 자료에 액세스하여 학생에게 개별적으로 제공하는 것이 좋습니다.  진행 중인 수업의 일환으로 학생들에게 GitHub로 직접 이동하여 랩 단계에 액세스하게 하면, 과정에서 다른 UI에 액세스해야 하므로 학생들은 혼란을 겪게 됩니다. 별도의 랩 지침이 제공되는 이유를 학생들에게 설명하면 클라우드 기반 인터페이스와 플랫폼의 끊임없이 변하는 특성을 강조할 수 있습니다. GitHub 파일 액세스와 GitHub 사이트 탐색에 대한 Microsoft Learning 지원은 이 과정을 가르치는 MCT에게만 제공됩니다.

https://aka.ms/az220labs 에서 Markdown 파일의 처리되고 서식 있는 버전을 확인할 수 있습니다.

### <a name="known-issues-in-the-current-release"></a>현재 릴리스에 대해 알려진 문제

같은 문제를 반복해서 생성하지 않도록 현재 공개된 문제를 참조해 주세요.  알려져 있고 현재 작업 중이나 공개되지 않은 문제는 다음과 같습니다.

* 대부분의 랩에서는 설치 스크립트를 실행해야 특별히 명명된 IoT 디바이스를 만들 수 있습니다. 랩 내부에서 그리고 랩 전체에 걸쳐서 디바이스 리소스를 "정리"하고 있습니다.

* 많은 랩 연습이 지정된 작업으로 분류되어 있지 않아서 헷갈리거나 단계를 놓칠 수 있습니다. 이 서식 설정을 해결하기 위해 작업 중에 있습니다.

* 랩 단계에는 이미지를 통해 지침을 명확히 이해할 수 있도록 마련된 곳이 있습니다. 현재 필요하다고 생각되는 랩에 이미지를 추가하고 있습니다. 그러나, Azure UI의 유연한 특성을 고려하여 이미지 수를 적게 유지하려고 합니다.
