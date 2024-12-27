---
title: "[ 멋쟁이사자처럼 백엔드 스쿨 ] 은행 시스템 개선: thread & logging 기능 추가"
date: 2024-12-24 16:24:00 +0900
categories: [ "external-experience", "likelion" ]
tags: [ "bootcamp", "java", "멋쟁이사자처럼", "부트캠프", "프로그래밍" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/external-experience/likelion/2024-12-24-likelion-grow-up-lionbank-proj-2/2024-12-24-16-24-59.png
  alt: "은행 시스템 개선: thread & logging 기능 추가"
---

> 학습을 목표로 자바 프로젝트를 진행하고 있습니다.
{: .prompt-info }

file read, write 기능을 활용한 은행 오픈, 마감 기능 추가

## 파일 입출력 유틸 작성

파일 입출력 작성 시 
[https://github.com/eun2ce/likelion/blob/main/lionbank/src/main/java/org/lionbank/util/FileReaderTask.java](https://github.com/eun2ce/likelion/blob/main/lionbank/src/main/java/org/lionbank/util/FileReaderTask.java)

```java
package org.lionbank.util;

import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class FileReaderTask extends Thread {
  private String logMessage;  // 로그 메시지
  private static final String LOG_FILE = "log.txt";  // 로그 파일 경로

  // 생성자, 로그 메시지를 받아서 설정
  public FileReaderTask(String logMessage) {
    this.logMessage = logMessage;
  }

  // 쓰레드 실행 시 로그를 파일에 기록
  @Override
  public void run() {
    writeLogToFile(logMessage);
  }

  // 로그를 파일에 작성하는 메서드
  private void writeLogToFile(String message) {
    try (BufferedWriter writer = new BufferedWriter(new FileWriter(LOG_FILE, true))) {
      // 현재 시간 얻기
      String timestamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
      // 로그 내용 작성
      writer.write("[" + timestamp + "] " + message);
      writer.newLine();
    } catch (IOException e) {
      System.out.println("로그 파일을 작성하는 중 오류가 발생했습니다: " + e.getMessage());
    }
  }
}
```

## 결과

아래와 같은 내용이 `log.txt` 파일에 남는다.

![fetch logfile](/assets/img/posts/external-experience/likelion/2024-12-24-likelion-grow-up-lionbank-proj-3/2024-12-27-15-34-07.png)

> 자세한 패치 내용은 [이 곳](https://github.com/eun2ce/likelion/commit/d6aeb14027b493a5b92c5ea5361933f6854f9173)을 참고해주세요.
{ : .prompt-info }
