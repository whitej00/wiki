---
tags: [concept, security, operational-security]
layer: network
related: [firewall]
source_count: 1
last_updated: 2026-04-09
---

# 침입 탐지/방지 시스템 (IDS/IPS)

패킷 필터(방화벽)가 헤더만 검사하는 것과 달리, **심층 패킷 검사(Deep Packet Inspection)**를 수행하여 악의적 활동을 탐지하거나 차단하는 시스템.

## IDS vs IPS

| | IDS (Intrusion Detection System) | IPS (Intrusion Prevention System) |
|---|---|---|
| **동작** | 의심스러운 트래픽 탐지 → **경보** 생성 | 의심스러운 트래픽 **차단** |
| **배치** | 트래픽 경로에서 관찰 (패시브) | 트래픽 경로에 인라인 (액티브) |

## 탐지 가능한 공격

- 네트워크 매핑 (nmap 등)
- 포트 스캔, TCP 스택 스캔
- DoS 대역폭 플러딩
- 웜과 바이러스
- OS 취약점 공격
- 애플리케이션 취약점 공격

## 배치 구조

조직 네트워크에 **다수의 IDS 센서**를 분산 배치하여 중앙 IDS 프로세서로 보고:

```
[내부 네트워크]
    |
[앱 게이트웨이] ─ [패킷 필터] ─── 인터넷
    |                  |
[IDS 센서]        [IDS 센서]
    |
[고보안 영역]    [DMZ: 웹서버, FTP, DNS]
                     |
                [IDS 센서]
```

- **고보안 영역**: 패킷 필터 + 앱 게이트웨이 + IDS 센서로 보호
- **DMZ (비무장지대, Demilitarized Zone)**: 외부 접속이 필요한 서버(웹, DNS) 배치. 패킷 필터 + IDS 센서로만 보호

다수 센서를 배치하는 이유: 각 센서가 전체 트래픽의 일부만 처리하여 심층 검사의 처리 부담을 분산.

## 탐지 방식

### 1. 시그니처 기반 (Signature-Based)

**공격 시그니처 데이터베이스**를 유지하고, 모든 패킷을 시그니처와 대조한다.

시그니처란:
- 특정 패킷의 특성 (소스/목적지 포트, 프로토콜, 페이로드 내 특정 비트열)
- 패킷 시리즈의 패턴

**Snort 시그니처 예시**:
```
alert icmp $EXTERNAL_NET any -> $HOME_NET any
(msg:"ICMP PING NMAP"; dsize: 0; itype: 8;)
```
→ 외부→내부, ICMP type 8(ping), 페이로드 크기 0 (nmap 특성) → "ICMP PING NMAP" 경보

**장점**: 알려진 공격에 대해 정확한 탐지  
**단점**:
- 새로운(알려지지 않은) 공격 탐지 불가
- 시그니처 매칭 시 오탐(false positive) 가능
- 대규모 시그니처 DB 대조에 처리 비용 높음

### 2. 이상 기반 (Anomaly-Based)

**정상 트래픽 프로파일**을 학습한 뒤, 통계적으로 이상한 패턴을 탐지한다.

예: ICMP 패킷 비율 급증, 포트 스캔의 지수적 증가 등

**장점**: 새로운, 문서화되지 않은 공격 탐지 가능  
**단점**: 정상과 비정상의 구별이 매우 어려움 → 오탐/미탐 빈도 높음

> 현재 대부분의 IDS 배포는 시그니처 기반이 주류이며, 일부 이상 기반 기능을 보완적으로 포함한다.

### Snort

가장 널리 사용되는 오픈소스 IDS. Linux/UNIX/Windows 지원. libpcap(Wireshark와 동일) 사용. 100 Mbps 이상 처리 가능. 커뮤니티가 새로운 공격 발견 시 수 시간 내에 시그니처를 배포한다.

## 출처

- [kurose-ch8](../sources/kurose-ch8.md) — Kurose 8판 8.9절
