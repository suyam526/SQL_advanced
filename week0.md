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


### 15.2.15.3

### 15.2.15.4

### 15.2.15.6

### 15.2.15.10

