# 📖 MYSQL 공식문서 정리
## Subqueries
### 15.2.15
- 서브쿼리는 다른 문에 포함될 수 있는 SELECT문 
- outer query 안에 nested됨! <br/>
  서브쿼리 안에 서브쿼리도 가능, depth를 쌓아간다고 이해하면 됨<br/>
  <br/>

- 서브쿼리의 장점
    - 각 문들을 분리해서 볼 수 있음 (구조화된 쿼리!)
    - 복잡한 JOIN, UNION을 대체 가능
<br/>

### 15.2.15.2 : 서브쿼리로 COMPARSIONS 쓰기
**비교 연산자로 비교**<br/>
```=   >   <  >=   <=   <>   !=   <=>```<br/>
예를 들어,<br/>
```... WHERE 'a' = (SELECT column1 FROM t1)```<br/>
또는<br/>
```... LIKE (subquery)```

<br/>

**최대값 찾기** -> 조인으로는 불가능한 구문들!
```
SELECT * FROM t1
  WHERE column1 = (SELECT MAX(column2) FROM t2);
```
또는
```
SELECT * FROM t1 AS t
  WHERE 2 = (SELECT COUNT(*) FROM t1 WHERE t1.id = t.id);
```
테이블 중 하나에 대한 집계가 필요하므로 조인으로는 불가능
<br/>

### 15.2.15.3 : ANY, IN or SOME
```
operand comparison_operator ANY (subquery)
operand IN (subquery)
operand comparison_operator SOME (subquery)
```
<br/>

**ANY**
```
SELECT s1 FROM t1 WHERE s1 > ANY (SELECT s1 FROM t2);
```
- 반드시 비교연산자와 함께
    - ANY of the values in the column that the subquery returns와의 비교가 TRUE라면 TRUE를 반환
    - t2가 NULL만 포함하고 있다면 NULL을 반환

- IN과 =ANY는 동의어가 X
- <>ANY와 <>SOME은 동의어 O
    - <>SOME
        - SQL 구문에서 "a is not equal to any b"는 "a와 같은 b는 하나도 없다"는 뜻이 아니라, **"하나라도 a와 다른 b가 있다"는 뜻**이다!
        - 이런 의미를 잘 전달하려면 <>SOME(not equal SOME=일부와는 동일하지 않다)을 쓰면 됨. 보통 잘 쓰는 표현은 아니긴 하다~

<br/>

### 15.2.15.4 : ALL
```operand comparison_operator ALL (subquery)```
- ALL of the values in the column that the subquery returns이 TRUE면 TRUE를 반환
```SELECT s1 FROM t1 WHERE s1 > ALL (SELECT s1 FROM t2);```
- t2가 NULL을 포함하고 있거나 비어있다면 NULL 반환
<BR/>

- NOT IN과 <>ALL은 동의어 O

<br/>

### 15.2.15.6 : EXISTS, NOT EXISTS
- EXISTS 서브쿼리에서는 보통 SELECT문에 *을 쓰곤 했는데, 뭘 써도 상관없음(어차피 달라지는거 없음)
- **중요!** EXISTS 서브쿼리에서 NULL이더라도 하나 이상의 열을 가지고 있으면 TRUE 반환 

<br/>

### 15.2.15.10 : 서브쿼리 ERRORS
- 1242 에러: NOT서브쿼리 비교연산자 서브쿼리 썼는데 서브쿼리에서 2개 이상의 결과를 반환했을 때 **자주 발생! 주의**
    ```
    ERROR 1242 (ER_SUBSELECT_NO_1_ROW)
SQLSTATE = 21000
Message = "Subquery returns more than 1 row"
    ```


- not yet support 에러 : 서브쿼리에서 지원하지 않는 구문 사용했을 때
    ```
    ERROR 1235 (ER_NOT_SUPPORTED_YET)
SQLSTATE = 42000
Message = "This version of MySQL doesn't yet support
'LIMIT & IN/ALL/ANY/SOME subquery'"
    ```

<br/>

----------
----------
# 📝 문제 풀이
1. 많이 주문한 테이블[https://solvesql.com/problems/find-tables-with-high-bill/]
```
SELECT *
FROM tips
WHERE total_bill > (SELECT AVG(total_bill) FROM tips)
```

2. 레스토랑의 대목[https://solvesql.com/problems/high-season-of-restaurant/]
```
SELECT *
FROM tips
WHERE day IN (
    SELECT day
    FROM tips
    GROUP BY day
    HAVING SUM(total_bill) >= 1500
);
```
<br/>

----------
# 📖 MYSQL 공식문서 정리
## WITH 표현식
```
WITH
  cte1 AS (SELECT a, b FROM table1),
  cte2 AS (SELECT c, d FROM table2)
SELECT b, d FROM cte1 JOIN cte2
WHERE cte1.a = cte2.c;
```
<br/>
----------

# 📝 문제 풀이
1. 식품분류별 가장 비싼 식품의 정보 조회하기[https://school.programmers.co.kr/learn/courses/30/lessons/131116]
```
SELECT A.CATEGORY, A.PRICE, A.PRODUCT_NAME
FROM FOOD_PRODUCT A
JOIN (
    SELECT CATEGORY, MAX(PRICE) AS MAX_PRICE
    FROM FOOD_PRODUCT 
    GROUP BY CATEGORY) B ON A.CATEGORY = B.CATEGORY AND A.PRICE = B.MAX_PRICE
WHERE A.CATEGORY IN ('과자', '국', '김치', '식용유')
ORDER BY PRICE DESC;
```

