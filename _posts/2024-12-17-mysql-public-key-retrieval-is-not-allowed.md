---
title: "[mysql/trubleshooting] Public Key Retrieval is not allowed"
date: 2024-12-17 00:33:00 +0900
categories: [ "programming", "database" ]
tags: [ "database", "mysql", "trubleshooting" ]
pin: false
math: false
mermaid: false
---

> Public key retrieval is not allowed
{: .prompt-danger }

사용하는 버전의 mysql 이 8.x 버전 이상 일 경우 아래와 같은 옵션을 확인해야 합니다.

| **옵션**                  | **설명**                      |
|-------------------------|-----------------------------|
| useSSL                  | DB 에 SSL 로 연결하는가            |
| allowPublicKeyRetrieval | 서버에서 RSA 공개키를 검색하거나 가져와야하는가 |

일반적으로 `useSSL=false`로 설정하고, `allowPublicKeyRetrieval`를 설정하지 않은 경우 발생

## 접속 URL 변경

```
jdbc:mysql://localhost:3306/{db_name}?useSSL=false&allowPublicKeyRetrieval=true
```
