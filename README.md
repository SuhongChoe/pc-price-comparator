# PC Price Comparator

사용자가 보유한 결제수단을 반영하여
PC 부품의 판매처별 예상 실결제액을 계산하고
가장 저렴한 구매처를 비교하는 서비스.

# 상품: CPU만

판매처:
- Gmarket
- 11st
- Naver

실제 자동 연동:
- 처음에는 없음

가격 데이터:
- Fixture

결제수단:
- CASH
- SHINHAN
- SAMSUNG
- KB

할인:
- 정률
- 정액
- 최소 결제금액
- 최대 할인한도
- 시작/종료일

핵심 결과:
예상 실결제액 순위

## API 설계 초안

CPU 선택 후 판매처별 기본 가격을 조회하고, 보유 결제수단을 선택하면 할인 적용 후 예상 결제액과 순위를 확인한다.

| 기능 | 메서드 | 경로 | 반환 내용 |
| --- | --- | --- | --- |
| CPU 목록 조회 | GET | `/api/products/cpus` | CPU ID, 상품명 |
| CPU별 판매조건 조회 | GET | `/api/products/cpus/{cpuId}/offers` | 판매처, 판매가격, 배송비 |
| 카드 조건을 적용한 가격 비교 | POST | `/api/products/cpus/{cpuId}/compare` | 할인 적용 후 예상 결제액, 순위 |

- `offers`는 판매처 자체가 아니라 해당 CPU의 판매가격·배송비 등 판매조건을 의미한다.
- 비교 대상 CPU는 경로의 `cpuId`로 지정하고, 보유 결제수단은 요청 본문의 `paymentMethods`로 전달한다.
- 비교 요청 예시: `POST /api/products/cpus/1/compare`

```json
{
  "paymentMethods": ["SHINHAN", "SAMSUNG"]
}
```

현재 문서는 API 설계 초안이며 구현 완료를 의미하지 않는다. 오류 응답과 상태코드 정책의 상세 확정은 후속 설계에서 진행한다.

## 가격비교 정상 응답

정상 계산은 `200 OK`로 응답한다. 전체 응답은 객체이며 `cpuId`로 비교 대상을, `results` 배열로 판매처별 결과 목록을 전달한다. 결과는 예상 결제액이 낮은 순으로 정렬한다.

아래는 실제 판매가격이 아닌 개발용 예시이며, 요청한 카드에 적용 가능한 할인이 없는 상황이다.

```json
{
  "cpuId": 1,
  "results": [
    {
      "rank": 1,
      "storeName": "Gmarket",
      "price": 100000,
      "shippingFee": 5000,
      "discountAmount": 0,
      "finalPrice": 105000
    },
    {
      "rank": 2,
      "storeName": "Naver",
      "price": 108000,
      "shippingFee": 0,
      "discountAmount": 0,
      "finalPrice": 108000
    }
  ]
}
```

- `price`: 판매가격
- `shippingFee`: 배송비
- `discountAmount`: 적용 할인액. 적용 가능한 할인이 없으면 0이다.
- `finalPrice`: 판매가격 + 배송비 - 적용 할인액
- `rank`: 예상 결제액 기준 순위. 동액 순위 처리 규칙은 후속 설계에서 정한다.

지원하는 결제수단이지만 적용 가능한 할인이 없는 경우도 정상 계산이며, 판매가격과 배송비의 합계를 반환한다.
