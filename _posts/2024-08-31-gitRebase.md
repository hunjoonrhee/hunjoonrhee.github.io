---
layout: single
title:  "Git Rebase & Git Merge"
categories: [Version Control, Git]
tags: [Version-Control, rebase, merge]
search: true
---

# Introduction
개발자들은 개발 프로세스에서 브랜치를 체계적으로 관리하기 위해 특정한 *워크플로우 (work flow)* 를 정의하는데, 이 플로우가 바로 *깃 플로우 (git flow)* 이다.  
*Git Flow* 는 다음과 같은 주요 브랜치와 그들 간의 관계를 중심으로 운영된다.
1. Main (default) branches (주요 브랜치)
   - `main`: 릴리즈 된 버전의 코드가 저장되는 브랜치다. 이 브랜치에는 항상 안정적인 코드만 포함된다. 배포가 가능한 코드만 이 브랜치에 머물러야 한다.
   - `develop`: 최신 개발이 이루어지는 브랜치이다. 보통 이 브랜치를 디폴트 브랜치로 정의하고 모든 개발이 이 브랜치를 기준으로 이루어지는 경우가 많다. 이 브랜치도 항상 작동이 되는 코드만 포함해야한다.

2. Supporting branches (보조 브랜치)
   - `feature` branches: 새로운 기능 개발을 위한 브랜치다. `develop` 브랜치에서 분기하여 기능을 개발하고, 개발이 완료되면 다시 `develop` 브랜치로 병합된다. 브랜치 이름은 일반적으로 `feature/<feature-name>`

이 밖에도 보조 브랜치에는 릴리즈 브랜치, 핫픽스 (hotfix) 브랜치들이 있을 수 있지만 이 글에서는 다루지 않겠다. 이 글에서는 이렇게 브랜치 작업을 하다가 생길 수 있는 여러가지 문제들 중에서 무조건 접하게 되는 문제인 **리베이스** 와 **병합** 에 대해서 알아보겠다.

# Problem
`feature` branch는 통상적으로 `develop` 브랜치에서 분기된다고 위에서 언급했다. 이때 여러개의 `feature` branch 들이 같은 `develop` 브랜치에서 생성될 수 있다. 그리고 그 중 하나의 기능이 완료가 되어서 `develop` 브랜치로 병합이 되었다고 생각해보자. 이렇게 되면 `develop` 브랜치에는 한번의 업데이트가 일어나게 되고, 다른 `feature` branch의 base가 되는 `develop` 브랜치와 차이가 생기게 된다. 아래의 그림을 보면 좀 더 이해하기 쉬울 것이다.

<img src="../../images/2024-08-31/git1.png" alt="git1" style="zoom:50%;" />

위의 그림을 보면 `feature 1` 과 `feature 2` 는 같은 `develop` 브랜치에서 생성이 되었고, 그 중 `feature 1` 브랜치가 완성이 되어 `develop` 브랜치로 병합이 되었다. 즉, `develop` 브랜치는 `feature 1` 의 기능이 새로 업데이트 된 것이다. 하지만 `feature 2` 는 아직 `feature 1` 의 기능을 가지고 있지 않다. 이렇게 되면 나중에 `feature 2` 브랜치가 완성이 되어서 `develop` 브랜치로 병합을 하려고 할 때 깃헙과 같은 플랫폼에서 conflict 때문에 병합을 할 수 없다고 한다. `feature 2`가 생성될 때 `develop` 브랜치는 `feature 1` 기능이 없었지만, `feature 2` 가 병합하려고 하는 `develop` 브랜치는 `feature 1` 기능이 있기 때문이다. 병합을 해야할 때는 기준점이 되는 브랜치의 상태는 똑같아야 한다.

이런 문제가 생겼을 때 해결하는 법 두가지를 소개하려고 한다. 첫번째는 `Merge`, 두번째는 `Rebase` 이다.

## `Merge`
위의 문제를 해결하기 위한 첫번째 방법은 업데이트된 `develop`을 `feature 2` 브랜치로 병합을 하는 것이다. 말하자면 두 브랜치를 합치는 것인데, `develop` 브랜치의 커밋 이력 (`feature 1`이 병합된 것까지 모두)을 살리고 `feature 2` 브랜치에 새로운 *머지 커밋* 을 생성하는 방식이다.

<img src="../../images/2024-08-31/merge.png" alt="merge" style="zoom:50%;" />

위의 그림을 보면 `develop` 브랜치가 `feature 1` 의 기능이 더해져서 업데이트가 되었다. 그리고 이 업데이트된 `develop` 브랜치를 `feature 2` 브랜치로 병합을 하는 것이다. 이렇게 되면 `feature 2` 브랜치의 기준점이 되었던 `develop` 브랜치에 업데이트된 `develop` 브랜치가 병합이 되면서 `feature` 1 의 기능이 `feature 2` 브랜치에 새롭게 들어와 이른바 *Merge Commit* 이 생기는 것이다.

이 방식의 특징은 다음과 같다.
- `develop` 브랜치의 모든 변경 사항을 그대로 `feature` 브랜치에 반영한다.
- 커밋 이력이 상대적으로 복잡해질 수 있다. `feature` 브랜치는 더이상 해당 기능의 커밋 이력만 갖고 있지 않게 되고 변경된 `develop`의 커밋 이력도 함께 갖고 있게 되기 때문이다.
- 하지만 이 때문에 각 브랜치의 커밋 이력이 유지가 된다는 장점도 있다.

## `Rebase`
첫 번째 방법인 `Merge`는 `develop`을 `feature` 브랜치로 병합을 시켰다면, `Rebase`는 작업중인 `feature` 브랜치를 새롭게 업데이트 된 브랜치 위로 옮겨서 마치 해당 `feature` 브랜치가 새롭게 업데이트 된 `develop` 브랜치에서 *새롭게* 작업한 것처럼 커밋 이력을 다시 쓰는 방식이다.

<img src="../../images/2024-08-31/rebase.png" alt="rebase" style="zoom:50%;" />

`rebase` 라는 이름으로 봐서 알 수 있듯이, 작업 중이던 `feature` 브랜치의 베이스를 *다시 새롭게* 한다고 생각하면 이해하기가 쉬울 것이다. `feature` 브랜치의 그동안의 커밋들이 `develop` 브랜치의 최신 커밋들 뒤로 이동하게 된다. 시간 상으로는 뒤죽박죽이 될 수 있지만, `merge` 와는 다르게 새로운 머지 커밋이 생성되지 않고 커밋 이력이 깔끔해진다는 특징이 있다. 이렇게 되면 깃 그래프에서는 해당 `feature` 브랜치와 `develop` 브랜치가 한 줄로 놓이게 되어 마치 `develop` 브랜치에서 파생되지 않은 것 처럼 보이게 된다. 

# Conclusion
오늘은 깃 플로우 작업 중에 `develop`과 `feature` 브랜치 사이에 흔히 생길 수 있는 문제에 대해서 알아보았다. 모든 `feature` 브랜치들은 보통 하나의 `develop` 브랜치에서 파생되지만, 그들이 완성되는 시점과 다시 `develop` 브랜치로 병합되는 시점은 다 다르기 때문에, 항상 개발자들은 `develop` 브랜치의 최신 상황을 알고 있어야 하며, 자신의 `feature` 브랜치의 기준점을 그에 맞춰 같이 업데이트를 해줘야 한다. 그리고 이 글에서는 그 업데이트 하는 방법 두 가지를 소개하였다.
1. `develop` 브랜치를 `feature` 브랜치로 병합: `Merge`
2. `feature` 브랜치의 베이스를 최신 `develop` 브랜치로 업데이트: `Rebase`

각각의 방식은 서로 장단점이 있고, 어떤것이 좋고 나쁘다를 말하기는 힘들다. 내 경험상 `Rebase`를 선호하는 시니어 개발자가 있었는가 하면, `Merge`를 선호하는 시니어 개발자도 있었다. 일단 우리 주니어 개발자들은 시니어가 원하는대로 가야하기 때문에 "왜 저걸 더 선호하지" 라는 질문 보단 각 방법의 특징들을 잘 이해하는게 중요하다고 생각한다.