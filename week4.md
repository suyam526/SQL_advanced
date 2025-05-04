# 📖 MYSQL 공식문서 정리
## [GROUP CONCAT](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_group-concat)
### 기본 개념
- 그룹 내에서 NULL이 아닌 값을 연결해 문자열로 반환
- 모든 값이 NULL인 경우 NULL 반환 
```sql
GROUP_CONCAT([DISTINCT] expr [, expr ...]
             [ORDER BY col_name [ASC | DESC] [, ...]]
             [SEPARATOR str_val])
```
<BR/>

```SQL
-- 기본 틀
SELECT <그룹화할 컬럼>, GROUP_CONCAT(<합쳐서 보여줄 컬럼>)
FROM <테이블명>
GROUP BY <그룹화할 컬럼>

-- 기본 사용
SELECT student_name, GROUP_CONCAT(test_score)
FROM student
GROUP BY student_name;
```

```SQL
-- DISTINCT, 정렬, 구분자 지정
SELECT student_name,
       GROUP_CONCAT(DISTINCT test_score ORDER BY test_score DESC SEPARATOR ' ')
FROM student
GROUP BY student_name;
```
<BR/>


- SEPARATOR : 값 사이의 구분자 지정 (기본: ,)
<BR/>

### 반환 형식
- 반환값은 TEXT 또는 BLOB
- group_concat_max_len ≤ 512인 경우: VARCHAR 또는 VARBINARY


<BR/>
<BR/>

## [Recursive CTE](https://dev.mysql.com/doc/refman/8.0/en/with.html#common-table-expressions-recursive)

### 기본 개념
- CTE : WITH절을 이용해 임시로 사용하는 테이블
- 재귀 CTE : **자기 자신**을 참조하는 CTE
    - 일반적으로 초기값과 재귀적으로 반복할 규칙을 지정해서 반복되는 값을 생성

```sql
--- 문법 구조
WITH RECURSIVE cte_name (column1, column2, ...) AS (
  -- 1단계: 초기값(비재귀 SELECT)
  SELECT ...

  UNION ALL

  -- 2단계: 재귀 SELECT (자기 자신을 참조)
  SELECT ... FROM cte_name WHERE 조건
)
SELECT * FROM cte_name;
```

```SQL
--- 예시
WITH RECURSIVE cte(n) AS (
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte WHERE n < 5 -- 조건이 False가 되면 재귀 종료
)
SELECT * FROM cte;
```

```SQL
--- 예시에 대한 결과
1
2
3
4
5
```

<br/>

### 문자열 확장
```sql
--- 예시
WITH RECURSIVE cte AS (
  SELECT 1 AS n, CAST('abc' AS CHAR(20)) AS str
  UNION ALL
  SELECT n + 1, CONCAT(str, str) FROM cte WHERE n < 3
)
SELECT * FROM cte;
```

```sql
--- 예시에 대한 결과
1 | abc
2 | abcabc
3 | abcabcabcabc
```
<BR/>

- CAST() : 데이터 타입 변환할 때 사용
    - 문자열 등이 점점 길어질 경우, 미리 넓은 타입으로 지정해야 오류 또는 잘림 방지 가능
- 재귀 CTE에서는 "UNION ALL" 권장 ! 성능상 좋다
- 재귀 SELECT에서 사용 불가능한 것
    - 집계 함수
    - 윈도우 함수
    - GROUP BY, ORDER BY, DISTINCT
    - LIMIT

```sql
-- 예시
WITH RECURSIVE date_series AS (
  SELECT DATE('2022-01-01') AS date
  UNION ALL
  SELECT date + INTERVAL 1 DAY FROM date_series WHERE date < '2022-01-10'
)
SELECT * FROM date_series;
```

<br/>

### 순서 및 규칙
1. 초기 SELECT는 한번만 실행됨
2. 재귀 SELECT는 **이전 단계 결과에 따라 반복**됨
3. CTE는 FROM 절에서만 딱 한번 참조 가능
4. 조인 가능, BUT LEFT JOIN의 오른쪽에서는 사용 불가

### 3줄 요약 ㅋ 
- 반복적으로 데이터 생성 OR 트리 구조 탐색 등에 유용
- 필수 구조 : WITH RECURSIVE + UNION ALL + 종료 조건 
- CAST() 사용해 문자열 길이 주의 !

------

### 재귀 CTE에서의 제한 / 종료 조건
- 자기 자신을 참조해서 반복적으로 데이터 만듬 -> 재귀가 끝나지 않으면 무한 루프에 빠질 수도..! 
- 그래서 명시적으로 종료 조건/제한 설정 필수

| 설정 방법                     | 설명                     | 예시                                                  |
| ------------------------- | ---------------------- | --------------------------------------------------- |
| `cte_max_recursion_depth` | 재귀 깊이 제한 (기본값: 1000)   | `SET SESSION cte_max_recursion_depth = 10;`         |
| `max_execution_time`      | 쿼리 전체 실행 시간 제한 (ms 단위) | `SET max_execution_time = 1000;`                    |
| `MAX_EXECUTION_TIME` 힌트   | 해당 쿼리만 시간 제한 설정        | `SELECT /*+ MAX_EXECUTION_TIME(1000) */ * FROM ...` |
| `LIMIT`                   | 반환할 행 수 제한             | `SELECT n + 1 FROM cte LIMIT 10000`                 |



<BR/>
<BR/>
<BR/>



# 문제 풀이
### 1. [우유와 요거트가 담긴 장바구니](https://school.programmers.co.kr/learn/courses/30/lessons/62284)
```sql
WITH CTE AS (
    SELECT 
        CART_ID,
        GROUP_CONCAT(NAME) AS NAME2
    FROM CART_PRODUCTS
    GROUP BY CART_ID
)

SELECT CART_ID
FROM CTE
WHERE NAME2 LIKE '%Milk%' AND NAME2 LIKE '%Yogurt%'
ORDER BY CART_ID;
```

- 문제 풀이
반드시 WITH문을 써야하는 문제는 아니지만 GROUP_CONCAT() 사용해보기 위해서 이렇게 풀어봄!
특이한 사항은 없으나 SELECT에 오는 컬럼은 GROUP BY에 있는 컬럼 OR 집계함수여야 하는 것 잊지 말기~~

### 2. [입양 시각 구하기(2)](https://school.programmers.co.kr/learn/courses/30/lessons/59413)

```SQL
WITH RECURSIVE hour_table AS (
    SELECT 0 AS hour
    UNION ALL -- WITH RECURSIVE에서는 중복 허용되는 UNION ALL 사용이 권장된다! 
    SELECT hour + 1 FROM hour_table WHERE hour < 23
)
SELECT h.hour AS HOUR, 
       COUNT(a.DATETIME) AS COUNT
FROM hour_table h
LEFT JOIN ANIMAL_OUTS a
ON h.hour = HOUR(a.DATETIME)
GROUP BY h.hour
ORDER BY h.hour;
```