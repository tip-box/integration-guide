# Reward Campaign API guide for advertiser

### Getting Started

- 본 가이드는 귀사와 TIP-BOX와의 S2S API 연동을 위한 가이드입니다
- 귀사의 캠페인을 TIP-BOX에게 제공함으로 TIP-BOX의 매체사들에게 캠페인을 노출시킬 수 있습니다

### 연동 흐름도

```
광고주                          TIP-BOX                         매체사
  |                               |                               |
  |                               |  1. 캠페인 리스트 조회 (동기화)   |
  | <---------------------------- |                               |
  |  캠페인 리스트 응답              |                               |
  | ----------------------------> |                               |
  |                               |                               |
  |                               |  2. 유저 캠페인 참여 요청         |
  |                               | <---------------------------- |
  |  3. 참여 요청 전달 (클릭)       |                               |
  | <---------------------------- |                               |
  |  랜딩 URL 응답                 |                               |
  | ----------------------------> |                               |
  |                               |  랜딩 URL 전달                  |
  |                               | ----------------------------> |
  |                               |                               |
  |  4. 유저 미션 완료              |                               |
  |  실적 포스트백 전송              |                               |
  | ----------------------------> |                               |
  |                               |  5. 매체사 콜백 전송              |
  |                               | ----------------------------> |
  |                               |                               |
```

### Prerequisites

- 귀사의 캠페인 리스트 조회 API와 참여 요청 API를 TIP-BOX에 제공해야 합니다
- 유저 미션 완료 후 TIP-BOX로 실적 포스트백을 발송해야 합니다

---
# 1.캠페인 리스트 조회 API

- 캠페인 리스트 조회 API는 귀사의 캠페인을 API를 통해 TIP-BOX에게 제공합니다
- TIP-BOX에서 주기적으로 귀사의 API를 호출하여 캠페인 리스트를 동기화합니다
- method : GET
- content-type : application/json

### 파라미터 정의

- 캠페인 요청시 필요한 파라미터를 정의하여 제공

| 항목      | 설명      | 필수 | 비고                                              |
|---------|---------|----|-------------------------------------------------|
| api_key | 매체사 식별키 | O  |                                                 |

### 응답 정의

- 캠페인 리스트 조회 API의 응답은 아래 예시와 같이 캠페인의 정보를 제공합니다
- 아래는 예시일 뿐이며 항목, 변수명 등 다르게 정의해주셔도 됩니다

| 항목                   | 형태     | 설명                                     | 비고                                     |
|----------------------|--------|----------------------------------------|----------------------------------------|
| result               | int    | 응답 결과 값                                |                                        |
| cnt                  | int    | 캠페인 수                                  |                                        |
| camp                 | array  | 캠페인 리스트                                |                                        |
| camp.campid          | int    | 캠페인 식별값                                |                                        |
| camp.os              | string | 모바일 OS                                 | android, ios                           |
| camp.name            | string | 캠페인명                                   |                                        |
| camp.bm              | int    | 캠페인 유형                                 | CPA, CPE, CPI, CPS 등                   |
| camp.sub_type        | int    | 캠페인 하위 유형                              | 퀴즈맞추기, 플레이스 저장하기, 영상 시청하기, 기타 등등 타입 분류 |
| camp.price           | int    | 캠페인 단가 (=매체비)                          | 100                                    |
| camp.rewarddesc      | string | 캠페인 적립 조건 안내                           | ex) 앱 설치 후 회원가입 완료                     |
| camp.joindesc        | string | 캠페인 참여 방법 안내                           |                                        |
| camp.totalquantity   | int    | 해당 캠페인의 참여 가능한 총 수량                    |                                        |
| camp.quantity        | int    | 해당 캠페인의 일별 참여 가능 수량(=매체사에 할당된 데일리 캡)   |                                        |
| camp.enddate         | string | 캠페인 종료일                                |                                        |
| camp.day_event_limit | int    | 일일 사용자당 참여 가능 횟수                       |                                        |
| camp.iconurl         | string | 캠페인의 아이콘 이미지 url                       |                                        |
| camp.ctv             | array  | 아이콘 이미지를 제외한 가로형, 세로형 소재 정보를 포함하고 있습니다 | 세부 항목 참고                               |

camp -> ctv 세부 항목

| 항목   | 형태     | 설명                            |
|------|--------|-------------------------------|
| type | int    | 1:가로형 소재, 1200*600, 2:1 ratio |
|      |        | 2:세로형 소재, 720*780             |
| url  | string | 소재 url                        |

#### 응답 예시
```json
{
    "result": 200,
    "cnt": 2,
    "camp": [
        {
          "campid": 415405,
          "name": "A 강남역 맛집 우대포 강남역점 육회 명소 퀴즈맞추기",
          "bm": 1,
          "sub_type": 2,
          "detail_type": "cpc_detail_click_tag",
          "price": 20,
          "price_dollar": 0.014,
          "rewarddesc": "퀴즈 정답 입력하기",
          "joindesc": "[참여방법]\r\n1. 미션 페이지 클릭하기\r\n...",
          "totalquantity": 14000,
          "quantity": 2000,
          "enddate": "20240724",
          "iconurl": "https://example.com/res/quiz_icon.png",
          "ctv": [
            {
              "type": 1,
              "url": "https://example.com/res/quiz_1200-600_1.png"
            }
          ]
        }
    ]
}
```

---
# 2.캠페인 참여 요청 API


- 광고에 참여하기 위해서는 캠페인 참여 요청 API를 호출해 참여 가능 여부를 확인해야 합니다.
- 캠페인 참여 요청 API를 통해 해당 사용자가 중복으로 참여하는지 확인하고
- 참여 가능한 경우 캠페인에 참여할 수 있는 URL을 응답합니다

### 요청
- method : GET
- content-type : application/json

### 파라미터 정의

| 항목       | 형태     | 내용                              | 필수 | 비고                                       |
|----------|--------|---------------------------------|----|------------------------------------------|
| api_key  | string | 매체사 식별키                         | O  |                                          |
| cbparam  | string | TIP-BOX에서 전달하는 참여 식별 파라미터       | O  | 해당 파라미터에 넣은 값을 포스트백을 통해 다시 전달받을 수 있도록 제공 |
| userid   | string | 참여 유저 식별값 (최대 길이: 100)          | O  |                                          |
| campid   | int    | 캠페인 식별값                         | O  |                                          |
| adid     | string | AOS = adid, iOS = idfa          | O  |                                          |

### 응답
| 항목     | 형태     | 내용                  |
|--------|--------|---------------------|
| result | int    | 처리 결과 코드            |
| lurl   | string | 사용자가 참여할 캠페인 랜딩 URL |

---
# 3.동적 광고주 API 설정

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
# 4.TIP-BOX로 실적 전송 (포스트백)

- 사용자가 정상적으로 미션에 참여해 전환이 완료된 경우 TIP-BOX로 포스트백을 발송합니다
- method : GET
- TIP-BOX postback URL : `https://postback.tipbox.kr/api/{adFormat}/ads/reward/{network}`
  - adFormat : `offerwall` 또는 `display`
  - network : 연동 담당자가 안내하는 귀사의 네트워크명

### 파라미터

| 항목     | 형태     | 설명                               | 필수 |
|--------|--------|----------------------------------|----|
| cbparam | string | 캠페인 참여시 전달받은 참여 식별 파라미터 (clickId) | O  |
| tid    | string | 귀사가 관리하는 트랜잭션 ID                 | O  |
| userid | string | 참여 유저 식별값                        |    |
| campid | int    | 캠페인 식별값                          |    |
| price  | int    | 수익금 (원)                          |    |

### 예시

```
https://postback.tipbox.kr/api/offerwall/ads/reward/my_company?cbparam=abc123&tid=tx_001&userid=user123&campid=12345&price=100
```

### 응답

| 항목     | 형태     | 설명       |
|--------|--------|----------|
| status | int    | HTTP 상태 코드 |
| code   | string | 응답 코드    |
| message | string | 응답 메시지   |

#### 포스트백 응답 예시

**성공**
```json
{
    "status": 200,
    "code": "SUCCESS",
    "message": "성공"
}
```

**실패 - 참여 이력 없음**
```json
{
    "status": 400,
    "code": "PARTICIPATE_NOT_FOUND",
    "message": "참여 이력을 찾을 수 없습니다"
}
```

---
## 부록: TrackerJS 스크립트 연동

광고주의 프로모션 웹페이지에서 스크립트 기반으로 전환을 추적하는 경우, 별도의 TrackerJS 연동 가이드를 참고해주세요.

- [TrackerJS integration guide for Advertiser](./CPA%20script%20integration%20guide%20for%20Advertiser.md)

---

## Authors

* **CHOI BAWOO** - *Integration technical support* - bw@adbc.co.kr


