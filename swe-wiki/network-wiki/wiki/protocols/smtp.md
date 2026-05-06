---
tags: [protocol, application-layer]
layer: application
rfc: 5321
related: [TCP, DNS, http]
source_count: 1
last_updated: 2026-04-08
---

# SMTP (Simple Mail Transfer Protocol)

인터넷 전자 메일의 핵심 프로토콜. 메일 서버 간 메시지를 전송하는 **push** 프로토콜이다.

## 한 줄 요약

TCP 포트 25를 사용하여 송신 메일 서버에서 수신 메일 서버로 메일을 **직접 전송**하는 프로토콜.

## 이메일 시스템 구성 요소

1. **사용자 에이전트(User Agent)**: 메일 작성/읽기 (Outlook, Gmail 앱 등)
2. **메일 서버**: 메일박스 관리, 발송 메시지 큐 유지
3. **SMTP**: 서버 간 메일 전송 프로토콜

## 동작 원리

1. Alice의 사용자 에이전트 → Alice의 메일 서버 (SMTP 또는 HTTP)
2. Alice의 메일 서버 → Bob의 메일 서버 (**SMTP**, TCP 포트 25 직접 연결)
3. Bob의 메일 서버 → Bob의 사용자 에이전트 (**HTTP** 또는 **IMAP**, pull 방식)

> 중간 메일 서버를 거치지 않고 **송신 서버에서 수신 서버로 직접** TCP 연결한다. 수신 서버가 다운이면 송신 서버 큐에서 약 30분마다 재시도.

## 프로토콜 특성

- **Push 프로토콜**: 송신 측이 수신 측에 밀어넣음 (HTTP는 pull)
- **지속 연결**: 같은 수신 서버로 여러 메시지를 하나의 TCP 연결로 전송
- **7비트 ASCII 제한**: 본문(헤더 포함)이 7비트 ASCII여야 함 → 바이너리 첨부 시 인코딩 필요
- 메시지 종료: 줄의 처음에 마침표(`.`)만 있는 라인으로 표시 (CRLF.CRLF)

## SMTP 명령 예시

```
S: 220 hamburger.edu
C: HELO crepes.fr
S: 250 Hello crepes.fr
C: MAIL FROM: <alice@crepes.fr>
S: 250 alice@crepes.fr ... Sender ok
C: RCPT TO: <bob@hamburger.edu>
S: 250 bob@hamburger.edu ... Recipient ok
C: DATA
S: 354 Enter mail, end with "." on a line by itself
C: (메시지 본문)
C: .
S: 250 Message accepted for delivery
C: QUIT
S: 221 hamburger.edu closing connection
```

## 메일 메시지 형식 (RFC 5322)

```
From: alice@crepes.fr
To: bob@hamburger.edu
Subject: Searching for the meaning of life.

(메시지 본문)
```

헤더 라인(From, To, Subject 등)과 본문 사이에 빈 줄로 구분. SMTP 명령(MAIL FROM 등)과 메시지 헤더(From:)는 별개.

## HTTP와의 비교

| 비교 항목 | SMTP | HTTP |
|---|---|---|
| 방향 | Push | Pull |
| ASCII 제한 | 7비트 ASCII 필수 | 제한 없음 |
| 객체 처리 | 모든 객체를 한 메시지에 포함 | 각 객체별 별도 응답 |
| 포트 | 25 | 80 (443) |

## 메일 접근 프로토콜

SMTP는 push 프로토콜이므로 수신자가 메일을 **가져오는(pull)** 데는 사용 불가:
- **HTTP**: 웹 기반 이메일 (Gmail 등)
- **IMAP** (RFC 3501): 서버에 메시지를 유지하며 폴더 관리 가능

## 출처

- [[kurose-ch2]] — Kurose 8판 2.3절
