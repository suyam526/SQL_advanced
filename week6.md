# 📝 문제 풀이

## [복수 국적 메달 수상한 선수 찾기](https://solvesql.com/problems/multiple-medalist/)

### 정답 코드
```SQL
SELECT a.name
FROM records r
JOIN athletes a ON r.athlete_id = a.id
JOIN teams t ON r.team_id = t.id
JOIN games g ON r.game_id = g.id
WHERE g.year >= 2000
  AND r.medal IS NOT NULL
GROUP BY a.id, a.name
HAVING COUNT(DISTINCT t.team) >= 2
ORDER BY a.name;
```

### 문제 풀이

- 과정 
1. 테이블별 컬럼 이름 찾기.. ^^ <br/>
    솔브SQL 불친절해.....짜증나........
    - ```LIMIT 5``` 넣어서 실행 결과 개수 제한 두고, 각 테이블별로 컬럼 이름 보면서 join key 이름 알아내기
2. 조건은<br/>
    (1) 2000년 이후에 (2) 2개 이상의 국적으로 (3) 메달을 수상한 선수이므로 events 테이블 제외하고 다 JOIN해야됨<br/>

3. 다른 테이블과 매핑할 수 있는 ID 정보를 가진 records 테이블을 중심으로 JOIN ㄱㄱ <br/>

4. WHERE로 데이터 자체를 필터링해줌 <br/>
(3번의 JOIN으로 만들어진 테이블에서 (1) 2000년 이후에 (3) 메달을 수상한 선수 데이터만 뽑아옴) <br/>

5. 2개 이상의 국적으로 메달을 수상한 선수만 뽑아와야 하는데, 1명의 선수가 2번 수상했다고 하면 2개의 수상 기록이 각각 따로따로 데이터 저장되어 있을 것이기 때문에 GROUP BY 사용해서 일단 선수 id별로 수상 기록들을 묶어줘야됨! 
<br/> ```GROUP BY a.id, a.name``` 사용
    - 이때 a.name으로 GROUP BY하면 동명이인까지 불러올 수도 있으므로 이를 방지하기 위해 a.id로 먼저 불러오고 a.name으로 한번 더 GROUP BY 처리 <br/>

6. 그 중 2개 이상을 가진 id를 찾기 위해 HAVING 사용 <br/>

7. 오름차순 정리 <br/>

8. SELECT a.name으로 이름 불러오기 끗~ <br/>

---------

## [온라인 쇼핑몰의 월 별 매출액 집계](https://solvesql.com/problems/shoppingmall-monthly-summary/)

### 정답 코드
```SQL
SELECT
  TO_CHAR(o.order_date, 'YYYY-MM') AS order_month,
  SUM(CASE WHEN o.order_id NOT LIKE 'C%' THEN oi.unit_price * oi.quantity ELSE 0 END) AS ordered_amount,
  SUM(CASE WHEN o.order_id LIKE 'C%' THEN -1 * oi.unit_price * oi.quantity ELSE 0 END) AS canceled_amount,
  SUM(CASE 
        WHEN o.order_id NOT LIKE 'C%' THEN oi.unit_price * oi.quantity
        WHEN o.order_id LIKE 'C%' THEN -1 * oi.unit_price * oi.quantity
      END) AS total_amount
FROM
  orders o
JOIN
  order_items oi ON o.order_id = oi.order_id
GROUP BY
  TO_CHAR(o.order_date, 'YYYY-MM')
ORDER BY
  order_month;
```
### 문제 풀이
1. 테이블 JOIN해서 금액 계산 (JOIN KEY: order_id)
2. 월 단위로 그룹핑<BR/>
    이때 MYSQL이랑은 다르게 strftime()을 사용해서 YYYY-MM 형식 추출해야됨
3. 취소가 아닌 주문 금액 계산하기 (C로 시작하지 않는 주문만 필터링해서 계산, 간단하니까 LIKE 사용)<br/>
    ```CASE WHEN ~ NOT LIKE 'C%' THEN ~ ELSE 0 END```
4. 취소 주문 금액 계산하기 (음수로 표시해야됨!)
5. 취소 금액 + 정상 주문 = 총 합계 계산 끗~

----

## [세 명이 서로 친구인 관계 찾기](https://solvesql.com/problems/friend-group-of-3/)

### 정답 코드 
```SQL
SELECT
  e1.user_a_id AS user_a_id,
  e1.user_b_id AS user_b_id,
  e2.user_b_id AS user_c_id
FROM edges e1
JOIN edges e2
  ON e1.user_b_id = e2.user_a_id
JOIN edges e3
  ON e3.user_a_id = e1.user_a_id AND e3.user_b_id = e2.user_b_id
WHERE
  e1.user_a_id < e1.user_b_id AND
  e1.user_b_id < e2.user_b_id AND
  3820 IN (e1.user_a_id, e1.user_b_id, e2.user_b_id)
ORDER BY
  user_a_id, user_b_id, user_c_id;
```

### 문제 풀이
- 조건
    1. edges 테이블은 양방향 친구 관계(A-B == B-A)
    2. 세 명이 서로 친구여야 하니까 총 3개의 관계가 존재해야됨 (A-B, B-C, C-A)
    3. 세 명 중 한명은 반드시 3820이어야 됨
    4. 출력 컬럼은 오름차순 (A -> B-> C)으로 정렬된 세 명 
    5. 중복 결과는 제거 ㄱ- 뭐이딴문제가다있어

- 풀이 과정
1. 3개 테이블 조인해서 A-B, B-C, C-A 관계 만들기<BR/>
    왜냐면 A-B, B-C, C-A 이 세 개가 충족되어야 셋이 친구임
2. 순서만 다르고 조합은 같은 중복을 방지하기 위해서 WHERE 조건으로 제한 두기<BR/>
    이거 어떻게 해야될지 고민을 좀 했는데 제일 쉬운건 세 명의 친구 ID가 오름차순인 정렬만 가능하다고 제한 두는거인듯
3. ```3820 IN ()```으로 세 명 중 한 명이 3820인 조건
4. 최종 출력~
