# Reward Campaign API guide for publisher

### Getting Started

- 본 가이드는 TIP-BOX와 매체사간의 S2S API 연동에 대한 내용입니다
- 귀사의 앱/서비스에 TIP-BOX의 리워드 캠페인을 연동하여 사용자에게 캠페인을 노출하고, 참여 실적에 따라 리워드를 지급할 수 있습니다

### 연동 흐름도

```
매체사                          TIP-BOX                         광고주
  |                               |                               |
  |  1. 캠페인 리스트 조회 요청      |                               |
  | ----------------------------> |                               |
  |  캠페인 리스트 응답              |                               |
  | <---------------------------- |                               |
  |                               |                               |
  |  2. 캠페인 참여 요청            |                               |
  | ----------------------------> |                               |
  |  랜딩 URL + clickId 응답       |                               |
  | <---------------------------- |                               |
  |                               |                               |
  |  3. 유저가 랜딩 URL로 이동      |                               |
  | ------------------------------------------------------------> |
  |                               |                               |
  |                               |  4. 유저 미션 완료 후 포스트백    |
  |                               | <---------------------------- |
  |                               |                               |
  |  5. 매체사 콜백 (실적 전송)     |                               |
  | <---------------------------- |                               |
  |  6. 유저에게 리워드 지급         |                               |
  |                               |                               |
```

### Prerequisites

- TIP-BOX 연동 담당자로부터 매체사 식별키(token)를 발급받아야 합니다
- 실적 수신을 위한 콜백 URL을 TIP-BOX 담당자에게 전달해야 합니다

---
# 1.캠페인 리스트 조회 API

- 5분 주기로 업데이트 받는 것을 추천드립니다
- method : GET
- adFormat에 따라 아래 두 API로 나뉩니다

### 1-1. 오퍼월 캠페인 리스트 조회 API

- production API url : https://ad.tipbox.kr/api/v1/reward/offerwall/campaigns
- test API url : http://dev.qtbit.co.kr:8037/api/v1/reward/offerwall/campaigns

### 1-2. DA(디스플레이) 캠페인 리스트 조회 API

- production API url : https://ad.tipbox.kr/api/v1/reward/display/campaigns
- test API url : http://dev.qtbit.co.kr:8037/api/v1/reward/display/campaigns

> 파라미터 및 응답 스펙은 두 API 동일합니다.

- parameter : token (필수), p (선택)


| 항목       | 설명      | 필수 | 비고                                                                         |
|----------|---------|----|----------------------------------------------------------------------------|
| token    | 매체사 식별키 | O  |                                                                            |
| p        | 플랫폼     |    | 기본 값은 전체이며, 구분하여 받을 경우 Android : 1, iOS : 2로 요청                            |
| category | 카테고리    |    | 기본 값은 전체이며, 구분하여 받을 경우 보험 : 01, 금융 : 02, 자동차보험 : 03, 카드 : 04, 기타 : 99로 요청 |
- 요청 예시 : https://ad.tipbox.kr/api/v1/reward/offerwall/campaigns?token={token}
- token은 발급 후 전달드립니다



| 항목                   | 형태            | 설명                                     | 비고                                                                 |
|----------------------|---------------|----------------------------------------|--------------------------------------------------------------------|
| result               | int           | 응답 결과 값                                | 200인 경우에만 정상, '결과 코드' 참고                                           |
| cnt                  | int           | 캠페인 수                                  |                                                                    |
| camp                 | array         | 캠페인 리스트                                |                                                                    |
| camp.adGroupId       | int           | 캠페인 식별값                                |                                                                    |
| camp.os              | string(7)     | 모바일 OS                                 | android, ios, WEB (WEB은 모든 OS 호환)                                  |
| camp.name            | string(255)   | 캠페인명                                   |                                                                    |
| camp.bm              | int           | 캠페인 유형                                 | 1: CPA, 2:CPE, 3:CPI, 4:CPS, 0: 기타                                 |
| camp.sub_type        | int           | 캠페인 하위 유형(bm 값이 1일 경우)                 | 0: 기본, 1: 플레이스 저장하기, 2:퀴즈맞추기, 3:스토어상품찜, 4:스토어 알림받기, 5: 유튜브, 6: SNS |
| camp.detail_type     | int           | 캠페인 하위 유형(bm 값이 1일 경우)                 | *아래 표 참고                                                           |
| camp.ad_category     | string(10)    | 캠페인의 카테고리                              | *아래 표 참고                                                           |
| camp.package         | string(100)   | 패키지네임 또는 url scheme                    | android: package name, iOS: custom url                             |
| camp.price           | int           | 캠페인 집행 단가 (=매체비)                       | 100                                                                |
| camp.price_dollar    | float(11,3)   | 캠페인 집행 단가 (=매체비)                       | 0.01                                                               |
| camp.rewarddesc      | string(50)    | 캠페인 적립 조건 안내                           | ex) 앱 설치 후 회원가입 완료                                                 |
| camp.joindesc        | string(65535) | 캠페인 참여 방법 안내                           |                                                                    |
| camp.totalquantity   | int           | 해당 캠페인의 참여 가능한 총 수량                    | 0: 무제한                                                             |
| camp.quantity        | int           | 해당 캠페인의 일별 참여 가능 수량(=매체사에 할당된 데일리 캡)   | 0: 무제한                                                             |
| camp.enddate         | string(8)     | 캠페인 종료일                                | 1) 값이 있는 경우: YYYYMMDD 2) 값이 없는 경우: 별도 종료일 없음                       |
| camp.day_event_limit | int           | 일일 사용자당 참여 가능 횟수                       | 0: 단 한번만 참여 가능, 1: 하루 한번 참여 가능, N: 하루 N번 참여 가능                     |
| camp.targetcarrier   | string(1)     | 통신사 타겟팅                                | 공백: 타겟 없음, 1: KT, 2: LGU+, 3: SKT                                  |
| camp.targetgender    | int           | 성별 타겟팅                                 | 0: 타겟 없음, 1: 남자, 2: 여자                                             |
| camp.targetagemin    | int           | 최소 연령 타겟팅                              | 0: 타겟 없음                                                           |
| camp.targetagemax    | int           | 최대 연령 타겟팅                              | 0: 타겟 없음                                                           |
| camp.targetpkg       | string(100)   | 앱 패키지네임 타겟팅. AOS 캠페인일 경우에만 내려옵니다       |                                                                    |
| camp.detargetpkg     | string(100)   | 앱 패키지네임 디타겟팅. AOS 캠페인일 경우에만 내려옵니다      |                                                                    |
| camp.iconurl         | string(255)   | 캠페인의 아이콘 이미지 url                       |                                                                    |
| camp.ctv             | array         | 아이콘 이미지를 제외한 가로형, 세로형 소재 정보를 포함하고 있습니다 | 세부 항목 참고                                                           |

(*) : 해당 사항이 없을 경우, 응답 항목이 전달되지 않습니다.

camp -> ad_category 세부 항목

| 카테고리 값 | 설명     |
|--------|--------|
| 01     | 보험     |
| 02     | 금융     |
| 03     | 자동차 보험 |
| 04     | 카드     |
| 99     | 기타     |

camp -> ctv 세부 항목

| 항목   | 형태          | 설명                            |
|------|-------------|-------------------------------|
| type | int         | 1:가로형 소재, 1200*600, 2:1 ratio |
|      |             | 2:세로형 소재, 720*780             |
|      |             | 3:세로형 소재, 640*960             |
|      |             | 4:가로형 소재, 640*100             |
| url  | string(255) | 소재 url                        |

camp -> detail_type 세부 항목

| detail_type | 설명                |
|-------------|-------------------|
| 101         | 기본 플레이스 저장하기      |
| 102         | 플레이스 URL 공유하기     |
| 103         | 업체 클릭 후 저장        |
| 104         | 플레이스 홈 저장         |
| 105         | keep 공유           |
| 106         | 알림받기              |
| 107         | 검색 후 정답 입력        |
| 108         | 서치 커스텀 미션         |
| 201         | 대표자명 맞추기          |
| 202         | 상품 클릭 후 태그 단어 맞추기 |
| 203         | 상품 클릭 후 대표자명 맞추기  |
| 204         | 플레이스 퀴즈           |
| 205         | 서치 플레이스 퀴즈        |
| 301         | 기본 상품찜            |
| 302         | 상품 클릭 후 상품찜       |
| 303         | 무신사 상품 찜하기        |
| 304         | 카카오톡 선물하기 찜하기     |
| 401         | 기본 알림받기           |
| 402         | 상품 클릭 후 알림받기      |
| 501         | 시청하기              |
| 502         | 구독하기              |
| 503         | 쇼츠 좋아요            |
| 504         | 영상 좋아요            |
| 505         | 영상 좋아요 & 채널 구독    |
| 601         | 인스타그램 팔로우         |
| 602         | 당근 단골 맺기          |
| 603         | 밴드 회원가입           |
| 604         | 와디즈 알림받기          |
| 605         | 카페 회원가입           |


#### 응답 예시
```json
{
    "result": 200,
    "cnt": 2,
    "camp": [
        {
          "adGroupId": 415405,
          "name": "A 강남역 맛집 우대포 강남역점 육회 명소 퀴즈맞추기",
          "bm": 1,
          "sub_type": 2,
          "detail_type": 202,
          "price": 20,
          "price_dollar": 0.014,
          "rewarddesc": "퀴즈 정답 입력하기",
          "joindesc": "[참여방법]\r\n1. 미션 페이지 클릭하기\r\n2. 강남역 맛집 우대포 강남역점 육회 <- 복사하기, \r\n3. 붙여넣기 후 해당 플레이스 클릭 -> 주변 -> 명소 클릭\r\n4. {명소} 1번째 장소는? 단어를 입력 하면 완료!\r\n\r\n\r\n[주의사항] \r\n이미 참여한 이력이 있다면 리워드가 지급되지 않을 수 있습니다. \r\nWIFI가 아닌 환경에서는 데이터 이용료가 발생할 수 있습니다.",
          "totalquantity": 14000,
          "quantity": 2000,
          "enddate": "20240724",
          "targetcarrier": "",
          "targetgender": 0,
          "targetagemin": 0,
          "targetagemax": 0,
          "targetpkg": "",
          "detargetpkg": "",
          "ad_event_limit": 9999,
          "iconurl": "https://webapp.superap.io/res/quiz_icon.png",
          "ctv": [
            {
              "type": 1,
              "url": "https://webapp.superap.io/res/quiz_1200-600_1.png"
            },
            {
              "type": 2,
              "url": "https://webapp.superap.io/res/quiz_720-780_1.png"
            }
          ]
        },
        {
            "adGroupId": 405080,
            "name": "목포 맛집 목포관광오리탕 오리탕 저장하기",
            "bm": 1,
            "sub_type": 1,
            "detail_type": 104,
            "price": 19,
            "price_dollar": 0.013,
            "rewarddesc": "플레이스 저장하기",
            "joindesc": "[참여방법]\r\n1. 미션 페이지 클릭 -> 플레이스 더보기 클릭 후\r\n2. 홈 메뉴에서 플레이스 저장 하고 저장된 화면 캡쳐하기 \r\n3. 돌아와서 캡쳐화면 업로드 하면 완료! \r\n\r\n\r\n\r\n\r\n[주의사항]\r\n이미 참여한 이력이 있다면 리워드가 지급되지 않을 수 있습니다.\r\nWIFI가 아닌 환경에서는 데이터 이용료가 발생할 수 있습니다.",
            "totalquantity": 6000,
            "quantity": 200,
            "enddate": "20240728",
            "targetcarrier": "",
            "targetgender": 0,
            "targetagemin": 0,
            "targetagemax": 0,
            "targetpkg": "",
            "detargetpkg": "",
            "ad_event_limit": 1,
            "iconurl": "https://webapp.superap.io/res/place_icon.png",
            "ctv": [
              {
                "type": 1,
                "url": "https://webapp.superap.io/res/place_1200-600_2.png"
              },
              {
                "type": 2,
                "url": "https://webapp.superap.io/res/place_720-780_2.png"
              }
            ]
        }
    ]
}
```

---
# 2.캠페인 참여 요청 API


- 광고에 참여하기 위해서는 캠페인 참여 요청 API를 호출해 참여 가능 여부를 확인해야 합니다.
- 참여 유저의 데이터 정보를 많이 전달 할 수록 더 많은 캠페인에 참여할 수 있습니다.

### 요청
- method : GET
- production API url : https://reward.tipbox.kr/api/participate
- test API url : https://dev.qtbit.co.kr:8436/api/participate

#### 필수 파라미터

| 항목         | 형태     | 필수 | 설명                                   |
|------------|--------|------|--------------------------------------|
| token      | string | O    | 매체사 식별키                              |
| userId     | string | O    | 참여 유저 식별값                            |
| adGroupId  | long   | O    | 광고그룹 식별값                             |
| ip         | string | O    | 참여 단말기 IP                            |
| deviceAdid | string | O    | 광고 식별자 (Android: ADID, iOS: IDFA)    |

#### 선택 파라미터

| 항목            | 형태      | 설명                              | 비고                                              |
|---------------|---------|----------------------------------|---------------------------------------------------|
| osType        | string  | 모바일 OS                          | ANDROID, IOS                                      |
| osVersion     | string  | OS 버전                           |                                                   |
| deviceModel   | string  | 단말기 모델                          |                                                   |
| deviceBrand   | string  | 단말기 제조사                         |                                                   |
| deviceNetwork | string  | 네트워크 타입                         | 3G, 4G, 5G, WIFI                                  |
| deviceCarrier | string  | 통신사                             | SKT, KT, LGU+                                    |
| userAgent     | string  | User-Agent                       |                                                   |
| callbackParam | string  | 매체사 정의 파라미터                     | 해당 파라미터에 값을 넣어 보내주시면 포스트백을 통해 다시 전달받을 수 있는 값입니다 |
| affiliateId   | string  | 하위 매체 아이디                       |                                                   |
| age           | int     | 유저 연령 정보                        | 연령 타겟팅 캠페인일 경우 필수                                |
| gender        | int     | 유저 성별 정보                        | 0: 모름, 1: 남자, 2: 여자. 성별 타겟팅 캠페인일 경우 필수           |

### 응답
| 항목              | 형태     | 설명              |
|-----------------|--------|-----------------|
| status          | int    | HTTP 상태 코드      |
| code            | string | 응답 코드           |
| message         | string | 응답 메시지          |
| data.landingUrl | string | 캠페인 랜딩 URL      |
| data.clickId    | string | 클릭 식별값          |

#### 응답 예시
```json
{
    "status": 200,
    "code": "SUCCESS",
    "message": "성공",
    "data": {
        "landingUrl": "https://example.com/campaign/landing?click_id=abc123",
        "clickId": "abc123"
    }
}
```

---
# 3.CPI 설치 확인 API
- 본 항목은 CPI(앱 설치) 캠페인만 해당하는 안내입니다
- CPI 캠페인은 별도의 설치 확인 API 없이, 섹션 2의 캠페인 참여 요청 API를 동일하게 사용합니다

### 요청
- 섹션 2의 캠페인 참여 요청 API와 동일합니다
- method : GET
- production API url : https://reward.tipbox.kr/api/participate
- test API url : https://dev.qtbit.co.kr:8436/api/participate
- 요청 파라미터는 섹션 2를 참고해주세요

### CPI 전환 인정 흐름
1. 매체사가 캠페인 참여 요청 API를 호출하면, 응답으로 앱스토어 랜딩 URL이 전달됩니다
2. 유저가 앱을 설치하면, 광고주 네트워크에서 설치를 자동으로 감지합니다
3. 설치가 확인되면 매체사 서버로 실적 포스트백이 전송됩니다 (섹션 4 참고)

---
# 4.실적 전송 (콜백)

- 캠페인을 정상적으로 참여 완료했을 때, 매체사 서버로 리워드 적립 요청을 보냅니다
  (폐사에서 직접 적립 처리하지 않습니다)
- 리워드 적립 요청을 받을 콜백 URL을 연동 담당자에게 전달합니다
- 콜백은 HTTP GET 또는 POST 방식으로 전송됩니다 (매체사 설정에 따라 결정)

#### 참고 : 방화벽 등 보안 설정이 되어 있는 경우 아래 정보를 허용해주세요
- <code>도메인 : proxy.tipbox.kr</code>
- <code>허용할 IP : 15.165.252.33</code>

### 4-1. 기본 매크로

TIP-BOX에서 실적 전송 시 아래 매크로 값을 치환하여 전달합니다.

| 항목           | 형태          | 설명                           | 비고       |
|--------------|-------------|------------------------------|----------|
| cbparam      | string(255) | 캠페인 참여시 보냈던 callbackParam값   |          |
| tid          | string(38)  | 실적에 대한 유니크값 (transaction ID) |          |
| subpid       | string(100) | 하위 매체 구분값                    | optional |
| userid       | string(100) | 참여 유저 식별값                    |          |
| campid       | int         | 캠페인 식별값                      |          |
| price        | float(11,1) | 수익금                          |          |
| price_dollar | float(11,3) | 달러 수익금                       |          |

#### 기본 매크로 예시

```
https://postback.com?clk_id={cbparam}&tid={tid}&aff_id={subpid}&uid={userid}&cid={campid}&price={price}
```

### 4-2. 동적 콜백 매크로

TIP-BOX는 매체사별로 콜백 URL의 파라미터 구성을 동적으로 설정할 수 있습니다.
매체사의 콜백 URL과 매크로 매핑은 TIP-BOX 어드민에서 설정되며, 아래의 매크로를 사용할 수 있습니다.

> 동적 콜백 매크로를 사용하면 매체사의 기존 콜백 스펙에 맞추어 파라미터명과 값을 자유롭게 매핑할 수 있습니다.
> 기본 매크로와 동적 콜백 매크로 중 하나를 선택하여 사용합니다. 동적 콜백 매크로 설정 시 연동 담당자에게 문의해주세요.

#### 사용 가능한 매크로 목록

**식별 정보**

| 매크로명        | 설명                    |
|---------------|-----------------------|
| clickId       | 클릭 트래킹 ID            |
| userId        | 참여 유저 식별값             |
| affiliateId   | 하위 매체 아이디             |
| mediaId       | 매체 계정 ID              |
| deviceAdid    | 광고 식별자 (ADID/IDFA)    |
| callbackParam | 매체사 정의 파라미터           |

**캠페인 정보**

| 매크로명          | 설명                   |
|----------------|----------------------|
| campaignId     | 캠페인 ID               |
| adGroupId      | 광고그룹 ID              |
| adFormat       | 광고 형식 (OFFERWALL, DISPLAY) |
| adType         | 광고 유형 (quest, survey, download 등) |
| mediaName      | 매체사명                 |

**금액 정보**

| 매크로명          | 설명                   |
|----------------|----------------------|
| rewardPrice    | 리워드 단가               |
| contractPrice  | 계약 단가 (원)            |
| userPrice      | 유저 지급 금액 (원)         |
| userPoint      | 유저 지급 포인트            |
| profit         | 플랫폼 수익               |

**디바이스 정보**

| 매크로명         | 설명                   |
|---------------|----------------------|
| ip            | 참여 단말기 IP             |
| osType        | OS 유형 (ANDROID, IOS)  |
| platform      | 플랫폼 (osType과 동일)     |
| osVersion     | OS 버전                |
| deviceModel   | 단말기 모델               |
| deviceBrand   | 단말기 제조사              |
| deviceCarrier | 통신사                  |
| deviceNetwork | 네트워크 타입              |
| userAgent     | User-Agent            |

**유저 정보**

| 매크로명   | 설명     |
|---------|--------|
| age     | 유저 연령  |
| gender  | 유저 성별  |

**기타**

| 매크로명           | 설명                   |
|-----------------|----------------------|
| clickTimestamp   | 클릭 시각 (KST epoch)   |

#### 동적 매크로 매핑 방식

매크로 매핑은 JSON 형태로 설정되며, 두 가지 방식을 지원합니다.

**1) 단순 매핑** : 매크로 값을 그대로 파라미터에 전달

```json
{
  "userId": "uid",
  "clickId": "click_id",
  "profit": "pft"
}
```
- 형식 : `"매크로명": "파라미터명"`
- 위 설정시 콜백 URL에 `uid={userId값}&click_id={clickId값}&pft={profit값}` 형태로 전달됩니다

**2) 열거형 매핑** : 매크로 값을 매체사에서 사용하는 코드값으로 변환하여 전달

```json
{
  "osType": {
    "paramName": "os",
    "values": {
      "IOS": "1",
      "ANDROID": "2"
    }
  }
}
```
- 형식 : `"매크로명": { "paramName": "파라미터명", "values": { "원래값": "변환값" } }`
- 위 설정시 OS가 IOS인 경우 `os=1`, ANDROID인 경우 `os=2`로 전달됩니다

#### 동적 매크로 설정 예시

아래는 매체사 콜백 URL과 매크로 매핑의 전체 예시입니다.

**콜백 URL** : `https://publisher.com/callback`

**매크로 매핑 설정** :
```json
{
  "userId": "uid",
  "clickId": "click_id",
  "campaignId": "cid",
  "rewardPrice": "price",
  "callbackParam": "cb",
  "osType": {
    "paramName": "os",
    "values": {
      "IOS": "1",
      "ANDROID": "2"
    }
  }
}
```

**실제 전송되는 콜백 URL** :
```
https://publisher.com/callback?uid=user123&click_id=663f1a2b3c4d5e6f&cid=12345&price=100&cb=my_param&os=2
```

---
## 공통
- result로 반환되는 코드 목록입니다

### result code

| 항목  | 내용           |
|-----|--------------|
| 200 | 성공           |
| 300 | 검토중으로 적립 대기  |
| 400 | 잘못된 요청       |
| 401 | 등록되지 않은 매체   |
| 402 | 유저 정보 오류     |
| 403 | 캠페인 정보 오류    |
| 405 | 참여 이력 오류     |
| 406 | 매체 오류        |
| 407 | 타겟팅 매칭 실패    |
| 500 | 종료된 캠페인      |
| 501 | 중복 참여        |
| 502 | 참여할 수 없는 캠페인 |
| 900 | 시스템 오류       |

## Authors

* **CHOI BAWOO** - *Integration technical support* - bw@adbc.co.kr


