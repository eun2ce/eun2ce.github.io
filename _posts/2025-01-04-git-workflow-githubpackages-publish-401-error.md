---
title: "[github actions] Could not PUT 'https://maven.pkg.github.com/xxx....jar'. Received status code 401 from server: Unauthorized"
description: "git Actions 에서 배포 시 401 에러에 대해 다룹니다."
date: 2024-12-18 10:47:00 +0900
categories: [ "etc", "git" ]
tags: [ "401", "git", "github actions", "unauthorized", "깃", "깃헙", "오류" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/2025-01-04-git-workflow-githubpackages-publish-401-error-2025-01-04-13-38-09.webp
  alt: "Could not PUT 'https://maven.pkg.github.com/xxx....jar'. Received status code 401 from server: Unauthorized"
---

## 원인

`maven central repository` 에 수동으로 직접 배포하던 패키지를 `github actions`와 연동시켜 `github packages` 에 배포하도록 수정하는
작업에서 아래와 같이 계정정보를 제대로 받아오지 못하는 문제가 발생했습니다.

```markdown
Run ./gradlew publish

> Task :compileJava UP-TO-DATE
> Task :processResources UP-TO-DATE
> Task :classes UP-TO-DATE
> Task :jar UP-TO-DATE
> Task :generateMetadataFileForGprPublication
> Task :generatePomFileForGprPublication
> Task :publishGprPublicationToGitHubPackagesRepository FAILED
FAILURE: Build failed with an exception.

* What went wrong:
  Execution failed for task ':publishGprPublicationToGitHubPackagesRepository'.

> Failed to publish publication 'gpr' to repository 'GitHubPackages'
> Could not
PUT 'https://maven.pkg.github.com/eun2ce/text2emoji/io/github/eun2ce/text2emoji/0.2.0/text2emoji-0.2.0.jar'.
Received status code 401 from server: Unauthorized

* Try:

> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.
> Get more help at https://help.gradle.org.
BUILD FAILED in 1s
6 actionable tasks: 3 executed, 3 up-to-date
Error: Process completed with exit code 1.
```

## 해결

`build.gradle` 에서 publishing 구문을 작성할 때, github 계정 정보가 제대로 입력되지 않아 발생한 문제였습니다.

### SSH Keys 정보 확인

[https://github.com/settings/keys](https://github.com/settings/keys)에서 `SSH Keys`가 활성화 중이 맞는지 확인

### `workflow` 파일

`workflow`에서는 아래와 같은 방법으로 GITHUB_TOKEN 을 이용할 수 있고,
`gradle-publish.yml`에서는 그 정보들을 USERNAME, TOKEN 이라는 환경변수로 제공합니다.

* [Automatic token authentication](https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication)

```yaml
# .github/workflows/gradle-publish.yml

  ...중략...
    - name: Publish to GitHub Packages
  run: ./gradlew publish
  env:
    USERNAME: ${{ github.actor }}
    TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### build.gradle 확인

위와 같이 작성되어 있어야 아래 파일에서 USERNAME, TOKEN 정보를 받아 `workflow`가 정상동작 할 수 있습니다.

```groovy
// build.gradle
publishing {
  repositories {
    maven {
      // ... 중략 ...
      credentials {
        username = project.findProperty("gpr.user") ?: System.getenv("USERNAME") 
        password = project.findProperty("gpr.key") ?: System.getenv("TOKEN")
      }
    }
  }
}
```

## 결과

![결과](/assets/img/posts/2025-01-04-git-workflow-githubpackages-publish-401-error-2025-01-04-13-38-09.webp)

> 혹시 다른 이유로 동일한 문제를 겪으신 분들은 댓글로 알려주세요 !
