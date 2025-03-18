# MYSQL 공식문서 정리
## Subqueries
### 15.2.15
- 서브쿼리는 다른 문에 포함될 수 있는 SELECT문 
- outer query 안에 nested됨! <br/>
  서브쿼리 안에 서브쿼리도 가능, depth를 쌓아간다고 이해하면 됨<br/>
  <br/>

- 서브쿼리의 장점
    - 각 문들을 분리해서 볼 수 있음 (구조화된 쿼리!)
    - 복잡한 JOIN, UNION을 대체 가능
    

### 15.2.15.2 : 서브쿼리로 COMPARSIONS 쓰기
**비교 연산자로 비교**
```=   >   <  >=   <=   <>   !=   <=>```
예를 들어,
```... WHERE 'a' = (SELECT column1 FROM t1)```
또는
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



### 15.2.15.4

### 15.2.15.6

### 15.2.15.10

