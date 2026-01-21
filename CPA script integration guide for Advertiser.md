# trackerJS integration guide for Advertiser

### Getting Started

> **사전예약/가입/상담신청/구입** 등 광고주의 프로모션 웹페이지와 스크립트 연동을 위한 가이드입니다 


### Prerequisites

> 광고주는 스크립트 연동을 위해 웹페이지를 소유하고 수정 권한이 있어야합니다

# 1.Configuration

head tag 내에 script를 로드합니다


```
<script src="https://static.tip-box.kr/dist/tipbox-tracker.min.js"></script>
<script>
    // Initialization
    const tracker = new Tracker({
        debug : true // true로 설정시 console에 로그를 출력합니다 (default:true)
    });
</script>
```


# 2.Method

- ### event()
- ### event(event_name)
- ### event(event_name[, option])

사용자가 가입/신청/구매 등의 액션을 한 경우 아래의 코드를 실행합니다 

```

// 간단하게 실적 전송 (비추천)
tracker.event();

// 이벤트명을 지정하여 전송
tracker.event("join");

// 유저를 식별할 수 있는 키와 함께 실적 전송
const user_id = "abc123"; // user ID, transaction ID, click ID, GAID 등 
tracker.event("join", { userId : user_id });

// 캠페인의 실적과 상관없이 사용자의 행동을 추적하고 싶은 경우
tracker.event("detail_view", { conversion : false });
tracker.event("detail_view", { conversion : false, userId : user_id });

// 구입 이벤트에 상품 정보의 정보를 추가하는 경우
tracker.event("purchase", { productId : "product123", price : 12000, quantity : 2, currency : "KRW" });

```

# 3. Option

- ### uid

`userId`(식별자) 값은 사용자의 ID나 사용자를 식별할 수 있는 어떤 값도 대체 가능합니다. 식별값은 귀사와의 데이터 비교시 사용되므로 필수로 넣는 것을 추천합니다
```
tracker.event("join", { userId : user_id });
```


- ### conversion (default : true)

캠페인의 실적으로 인정되는 이벤트일 경우 `true`로 설정하고, 실적과 상관없이 사용자의 행동을 추적을 하는 경우 `false`로 설정합니다
```
tracker.event("join", { conversion : true });

tracker.event("login", { conversion : false });
```

- ### productId, price, quantity, currency

구매 이벤트에 대해 상품 정보를 추가할 수 있습니다
```
tracker.event("purchase", { productId : "product123", price : 12000, quantity : 2, currency : "KRW", conversion : true });
```


## Authors

* **CHOI BAWOO** - *Integration technical support* - bw@adbc.co.kr





