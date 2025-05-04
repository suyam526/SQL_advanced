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