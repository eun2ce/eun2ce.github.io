---
title: "[멋쟁이사자처럼 백엔드 스쿨] 은행 시스템 개선: Scanner 를 BufferedReader 로 변환"
date: 2024-12-24 16:24:00 +0900
categories: [ "etc", "project" ]
tags: [ "bootcamp", "java", "멋쟁이사자처럼", "부트캠프", "프로그래밍" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/2024-12-24-likelion-grow-up-lionbank-proj-2-2024-12-24-16-24-59.wepb
  alt: "은행 시스템 개선: Scanner 를 BufferedReader 로 변환"
---

> 학습을 목표로 자바 프로젝트를 진행하고 있습니다.
{: .prompt-info }

`Scanner`와 `BufferedReader`에 대해 아래와 같은 차이점을 알게 되었습니다.

| **특징**     | **Scanner**      | **BufferedReader**    |
|------------|------------------|-----------------------|
| **속도**     | 느림 (정규식 처리가 추가됨) | 빠름 (단순 읽기 중심)         |
| **메모리 효율** | 상대적으로 무거움        | 더 가볍고 효율적             |
| **사용 용도**  | 간단한 입력 처리에 적합    | 대량 데이터나 고성능 입력 처리에 적합 |
| **유연성**    | 정규식 기반 파싱 지원     | 직접 파싱 필요 (더 유연함)      |

### 비교

```java
import java.io.*;
import java.util.Scanner;

public class InputSpeedComparison {

  public static void main(String[] args) throws IOException {
    // 입력 데이터를 생성
    StringBuilder mockInput = new StringBuilder();
    for (int i = 1; i <= 100000; i++) {
      mockInput.append(i).append("\n");
    }
    String inputData = mockInput.toString();

    // Scanner 방식
    long scannerStartTime = System.currentTimeMillis();
    Scanner scanner = new Scanner(new StringReader(inputData));
    while (scanner.hasNextLine()) {
      int num = Integer.parseInt(scanner.nextLine());
    }
    scanner.close();
    long scannerEndTime = System.currentTimeMillis();

    // BufferedReader 방식
    long bufferedReaderStartTime = System.currentTimeMillis();
    BufferedReader bufferedReader = new BufferedReader(new StringReader(inputData));
    String line;
    while ((line = bufferedReader.readLine()) != null) {
      int num = Integer.parseInt(line);
    }
    bufferedReader.close();
    long bufferedReaderEndTime = System.currentTimeMillis();

    // 결과 출력
    System.out.println("Scanner 방식 처리 시간: " + (scannerEndTime - scannerStartTime) + "ms");
    System.out.println(
      "BufferedReader 방식 처리 시간: " + (bufferedReaderEndTime - bufferedReaderStartTime) + "ms");
  }
}
```

#### 결과

```
Scanner 방식 처리 시간: 45ms
BufferedReader 방식 처리 시간: 30ms
```

### 결론

간단한 작업엔 `Scanner`, 성능과 유연성을 원하면 `BufferedReader`

현재 lion bank 는 반복적인 입력 상황이 많기때문에 변경하기로 결정

### 변경 사항

#### 입력 방식

* 기존

```java
      Scanner scanner = new Scanner(System.in);
```

* 현재

```java
      BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
```

> 자세한 변경 사항은 [이 곳](https://github.com/eun2ce/likelion/commit/f7bbbb45328aaf07677353e390677d06bda5cf8e)을 참고해주세요.
{: .prompt-info}
