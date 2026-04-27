# Reward Campaign API guide for advertiser

### Getting Started

- 본 가이드는 귀사와 TIP-BOX와의 API 연동을 위한 가이드입니다
- 귀사의 캠페인을 TIP-BOX에게 제공함으로 TIP-BOX의 매체사들에게 캠페인을 노출시킬 수 있습니다

### 연동 흐름도

```mermaid
sequenceDiagram
    participant 광고주
    participant TIP-BOX
    participant 매체사

    rect rgb(240, 248, 255)
    Note over TIP-BOX, 광고주: 1. 캠페인 동기화
    TIP-BOX->>광고주: 캠페인 리스트 조회 API 호출
    광고주-->>TIP-BOX: 캠페인 리스트 응답
    end

    rect rgb(245, 245, 255)
    Note over 매체사, 광고주: 2~3. 캠페인 참여
    매체사->>TIP-BOX: 유저 캠페인 참여 요청
    TIP-BOX->>광고주: 참여 요청 전달 (클릭)
    광고주-->>TIP-BOX: 랜딩 URL 응답
    TIP-BOX-->>매체사: 랜딩 URL 전달
    end

    rect rgb(255, 245, 240)
    Note over 광고주, 매체사: 4~5. 전환 처리
    광고주->>TIP-BOX: 유저 미션 완료 → 실적 포스트백 전송
    TIP-BOX->>매체사: 매체사 콜백 전송
    end
```

### Prerequisites

- 귀사의 캠페인 리스트 조회 API와 참여 요청 API를 TIP-BOX에 제공해야 합니다
- 유저 미션 완료 후 TIP-BOX로 실적 포스트백을 발송해야 합니다

---
# 1.동적 광고주 API 설정

TIP-BOX는 광고주의 기존 API 스펙에 맞추어 동적으로 연동할 수 있는 설정 기능을 제공합니다.
광고주가 이미 사용 중인 캠페인 리스트 API와 참여 요청 API가 있는 경우, TIP-BOX 어드민에서 필드 매핑을 설정하여 별도의 API 개발 없이 연동할 수 있습니다.

> 이 기능은 ADBC와의 주요 차이점으로, ADBC는 각 광고주별로 고정된 연동 코드가 필요했지만,
> TIP-BOX는 어드민 설정만으로 새로운 광고주를 추가할 수 있습니다.

### 동적 API 동기화 설정 항목

TIP-BOX 어드민에서 아래 항목을 설정하여 귀사의 API와 연동합니다.

#### 기본 설정

| 항목              | 설명                          | 비고                             |
|-------------------|-----------------------------|--------------------------------|
| apiUrl            | 캠페인 리스트 API 기본 URL          |                                |
| androidUrl        | Android 전용 API URL           | apiUrl 대신 OS별 URL 사용 시        |
| iosUrl            | iOS 전용 API URL               | apiUrl 대신 OS별 URL 사용 시        |
| advertiserCode    | 광고주 식별 코드                   | API 호출 시 인증 파라미터로 사용          |
| adFormat          | 광고 형식                       | OFFERWALL 또는 DISPLAY           |

#### 필드 매핑 설정

귀사의 API 응답에서 TIP-BOX가 인식해야 하는 필드명을 매핑합니다.

| 항목                | 설명                          | 예시                             |
|---------------------|-----------------------------|--------------------------------|
| dataArrayField      | 캠페인 배열이 포함된 JSON 필드 경로      | "camp", "data.campaigns"       |
| externalIdField     | 캠페인 식별값 필드명                  | "campid", "id"                 |
| nameField           | 캠페인명 필드명                     | "name", "title"                |
| priceField          | 단가 필드명                       | "price", "reward"              |
| totalQuantityField  | 총 수량 필드명                     | "totalquantity", "total_cap"   |
| dailyQuantityField  | 일별 수량 필드명                    | "quantity", "daily_cap"        |
| osField             | OS 필드명                       | "os", "platform"               |
| adTypeField         | 광고 유형 필드명                    | "bm", "ad_type"                |

#### 값 매핑 설정

귀사의 API에서 사용하는 값(코드)을 TIP-BOX 내부 값으로 변환합니다.

| 항목           | 설명                          | 예시                             |
|----------------|-----------------------------|--------------------------------|
| osMapping      | OS 값 변환 매핑                   | `{"AOS": "ANDROID", "IOS": "IOS"}` |
| adTypeMapping  | 광고 유형 변환 매핑                  | `{"1": "CPA", "2": "CPE"}`     |
| assetMapping   | 소재(creative) 필드 매핑           | `{"icon": "iconurl"}`          |
| fieldMapping   | 기타 사용자 정의 필드 매핑              | 귀사 응답의 커스텀 필드를 TIP-BOX 필드로 매핑 |

#### 설정 예시

귀사의 캠페인 리스트 API 응답이 아래와 같은 경우:

```json
{
    "status": "ok",
    "total": 5,
    "campaigns": [
        {
            "id": 1001,
            "title": "앱 설치 캠페인",
            "reward": 500,
            "platform": "AOS",
            "type": "CPI",
            "daily_limit": 100,
            "icon": "https://example.com/icon.png"
        }
    ]
}
```

TIP-BOX 어드민에서 아래와 같이 설정합니다:

```
dataArrayField     : campaigns
externalIdField    : id
nameField          : title
priceField         : reward
osField            : platform
adTypeField        : type
dailyQuantityField : daily_limit
osMapping          : {"AOS": "ANDROID", "IOS": "IOS"}
assetMapping       : {"icon": "iconurl"}
```

> 동적 API 설정에 대한 자세한 문의는 연동 담당자에게 문의해주세요.

---
# 2.TIP-BOX로 실적 전송 (포스트백)

- 사용자가 정상적으로 미션에 참여해 전환이 완료된 경우 TIP-BOX로 포스트백을 발송합니다
- method : GET
- TIP-BOX postback URL : `https://postback.tipbox.kr/api/postback`

> 포스트백 발송에 필요한 파라미터가 있으면 연동 담당자에게 요청해주세요.

### 파라미터

| 항목            | 형태     | 설명                               | 필수 |
|---------------|--------|----------------------------------|----|
| clickId       | string | 캠페인 참여시 전달받은 참여 식별 파라미터 | O  |
| callbackParam | string | 귀사가 관리하는 트랜잭션 ID                 | O  |
| price         | int    | 수익금 (원)                          |    |

### 예시

```
https://postback.tipbox.kr/api/postback?clickId=abc123&callbackParam=tx_001&price=100
```

### 응답

> 포스트백 API는 모든 케이스에 대해 **HTTP 200**을 반환하며, 처리 결과는 응답 body의 `status` / `code` 필드로 구분합니다.

| 항목      | 형태     | 설명                      |
|---------|--------|-------------------------|
| status  | int    | 처리 결과 코드 (200=성공, 그 외 실패) |
| code    | string | 응답 코드 (예: `OK`, `NOT_FOUND_REWARD_DOC`) |
| message | string | 응답 메시지                  |
| data    | object | 처리 상세 (성공 시에만 포함)        |

#### 처리 결과 코드 (status)

| status | code                       | 설명                                                           |
|--------|----------------------------|--------------------------------------------------------------|
| 200    | `OK`                       | 정상 처리 (매체 콜백 성공/실패 여부는 `data.statusCode` 로 확인)               |
| 400    | `VALIDATION_FAILED`        | 필수 파라미터 누락 (`clickId`, `eventType` 등)                         |
| 400    | `INVALID_PARAMETER`        | `clickId` 형식 오류 (ObjectId 형식이 아님)                             |
| 400    | `INVALID_JSON`             | 요청 body JSON 파싱 실패                                            |
| 405    | `NOT_FOUND_REWARD_DOC`     | 해당 `clickId`의 참여(RewardDoc) 정보를 찾을 수 없음                       |
| 405    | `INVALID_CLICK_ID`         | 유효하지 않은 `clickId` 형식                                          |
| 429    | `EVENTS_LIMIT_EXCEEDED`    | 단일 클릭에 누적된 이벤트가 최대 개수(100개)를 초과                              |
| 501    | `DUPLICATE_EVENT`          | 이미 성공 처리된 전환(conversion) 이벤트가 존재 (중복 포스트백)                    |
| 500    | `INTERNAL_SERVER_ERROR`    | 서버 내부 오류                                                      |
| 900    | `POSTBACK_PROCESSING_FAILED` | 포스트백 처리 중 알 수 없는 오류 발생                                      |

#### 포스트백 응답 예시

**성공**
```json
{
    "status": 200,
    "code": "OK",
    "message": "ok",
    "data": {
        "success": true,
        "postbackSent": true,
        "statusCode": 200
    }
}
```

> `data.statusCode` 는 TIP-BOX → 매체로 전송된 콜백의 HTTP 상태 코드입니다. (200~299: 매체 콜백 성공)

**실패 - 참여 이력 없음**
```json
{
    "status": 405,
    "code": "NOT_FOUND_REWARD_DOC",
    "message": "RewardDoc을 찾을 수 없습니다."
}
```

**실패 - 중복 포스트백**
```json
{
    "status": 501,
    "code": "DUPLICATE_EVENT",
    "message": "이미 성공한 conversion 이벤트가 존재합니다."
}
```

**실패 - 필수 파라미터 누락**
```json
{
    "status": 400,
    "code": "VALIDATION_FAILED",
    "message": "clickId는 필수입니다"
}
```

---
## 부록: TrackerJS 스크립트 연동

광고주의 프로모션 웹페이지에서 스크립트 기반으로 전환을 추적하는 경우, 별도의 TrackerJS 연동 가이드를 참고해주세요.

- [TrackerJS integration guide for Advertiser](./CPA%20script%20integration%20guide%20for%20Advertiser.md)

---

## Authors

* **CHOI BAWOO** - *Integration technical support* - bw@adbc.co.kr


