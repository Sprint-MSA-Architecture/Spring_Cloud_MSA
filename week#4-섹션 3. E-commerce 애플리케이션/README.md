E-commerce 애플리케이션

1. 전체 구조

   <img src="https://user-images.githubusercontent.com/64065318/138685784-badee362-3efa-4132-b13c-63e237d271a6.png">

   2. 제약사항

   - 주문시 재고수량에서 -1, 재고수량이 0일 경우 주문 불가해야
   - Messaging 서비스 사용(메세지를 발행하는 쪽 / 메세지를 얻어가는쪽)
   - 사용자 주문 -> DB에 해당 데이터 저장 -> ORDER-SERVICE는 Kafka에 해당 내용 전달 ->  CATALOG-SERVICE는 구독을 했으므로 변경사항이 들어오면 받음