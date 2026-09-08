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

현재 문서는 API 설계 초안이며 구현 완료를 의미하지 않는다. 상세 응답 형식과 상태코드는 후속 설계에서 정한다.
