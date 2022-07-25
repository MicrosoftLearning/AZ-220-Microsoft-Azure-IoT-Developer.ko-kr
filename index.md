---
title: 온라인 호스팅 지침
permalink: index.html
layout: home
ms.openlocfilehash: c906809bcfcfd6cfac0139d9c6d9432cff05fbf1
ms.sourcegitcommit: 5e9f89d47b27285feaf13cacfa097ddd4b888a90
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/14/2022
ms.locfileid: "137893075"
---
# <a name="content-directory"></a>콘텐츠 디렉터리

다음은 각 랩 연습 및 데모의 하이퍼링크입니다.

## <a name="labs"></a>랩

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| 모듈 | 랩 |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

{% comment %}
<!-- Comment out the Jekyll template that lists the placeholder demo -->

## <a name="demos"></a>데모

{% assign demos = site.pages | where_exp:"page", "page.url contains '/Instructions/Demos'" %}
| 모듈 | 데모 |
| --- | --- | 
{% for activity in demos  %}| {{ activity.demo.module }} | [{{ activity.demo.title }}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

{% endcomment %}
