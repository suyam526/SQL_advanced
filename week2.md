# 📖 MYSQL 공식문서 정리
## [JOIN문](https://dev.mysql.com/doc/refman/8.0/en/join.html)
### 15.2.13.2
- 기본
```SELECT * FROM (SELECT 1, 2, 3) AS t1;```
- 단일 조인에서 참조할 수 있는 테이블 수는 61개

- LEFT JOIN과 NULL 처리
    - LEFT JOIN에서 오른쪽 테이블에 일치하는 행이 없는 경우, 오른쪽 테이블의 모든 컬럼이 NULL값이 됨. 이걸 잘 활용하면 한 테이블에서 다른 테이블에 존재하지 않는 데이터를 찾을 수 있다!
    ```
    SELECT left_tbl.*
        FROM left_tbl LEFT JOIN right_tbl ON left_tbl.id = right_tbl.id
        WHERE right_tbl.id IS NULL;
    ```

- USING 절
    - 두 테이블에 **공통으로 존재하는 열으로 조인**할 때 사용 <br/>
    ```a LEFT JOIN b USING (공통 컬럼)```
    =
    ```a LEFT JOIN b ON a.c1 = b.c1 AND a.c2 = b.c2 AND a.c3 = b.c3```

- NATURAL JOIN
    - 두 테이블에 공통된 컬럼을 **자동으로 찾아** 조인 <br/>
    ```SELECT * FROM t1 NATURAL JOIN t2;```
    =
    ```SELECT * FROM t1 JOIN t2 USING (공통 컬럼);```

    - 표준 SQL에 따라 NATURAL JOIN은 **중복 컬럼을 제거**함!
    ```
    CREATE TABLE t1 (i INT, j INT);
    CREATE TABLE t2 (k INT, j INT);
    INSERT INTO t1 VALUES(1, 1);
    INSERT INTO t2 VALUES(1, 1);
    SELECT * FROM t1 NATURAL JOIN t2;
    ```
    🔽
    ```
    +------+------+------+
    | j    | i    | k    |
    +------+------+------+
    |    1 |    1 |    1 |
    +------+------+------+
    ```
    💡 j 컬럼이 두 테이블 모두에 존재하지만 결과적으로는 하나의 컬럼만 유지됐다!

    - STRAIGHT JOIN
        - JOIN과 유사하지만, **항상 왼쪽 테이블을 먼저 읽도록 강제**함
        - MYSQL의 자동 최적화 기능 무시하고 테이블 조인 순서를 강제하는 방식 (일반적으로 MYSQL은 최적의 테이블 조인 순서를 자동으로 결정함)
            - EX. 
            ```SELECT * FROM employees e JOIN departments d ON e.dept_id = d.id;```<br/>
            💡 MYSQL은 더 효율적인 테이블 조인 순서를 자동으로 결정하므로 departments 테이블이 더 작은 테이블이라면 이걸 먼저 읽고 employees를 나중에 읽을 것 <br/>
            💡 하지만 STRAIGHT_JOIN을 쓰면 무조건 왼쪽에 있는 employees 테이블을 먼저 읽음
<br/>
<br/>
<br/>

# 📝 문제 풀이
### [1. 저자 별 카테고리 별 매출액 집계하기](https://school.programmers.co.kr/learn/courses/30/lessons/144856)

- 문제 <br/>
2022년 1월의 도서 판매 데이터를 기준으로 저자 별, 카테고리 별 매출액(TOTAL_SALES = 판매량 * 판매가) 을 구하여, 저자 ID(AUTHOR_ID), 저자명(AUTHOR_NAME), 카테고리(CATEGORY), 매출액(SALES) 리스트를 출력하는 SQL문을 작성해주세요.<br/>
결과는 저자 ID를 오름차순으로, 저자 ID가 같다면 카테고리를 내림차순 정렬해주세요.
<br/>
<br/>

- 정답
```
SELECT 
    B.AUTHOR_ID,
    A.AUTHOR_NAME,
    B.CATEGORY,
    SUM(S.SALES*B.PRICE) AS "TOTAL_SALES"
FROM BOOK B
JOIN AUTHOR A ON B.AUTHOR_ID = A.AUTHOR_ID
JOIN BOOK_SALES S ON S.BOOK_ID = B.BOOK_ID
WHERE EXTRACT(YEAR_MONTH FROM S.SALES_DATE) = '202201'
GROUP BY A.AUTHOR_ID, B.CATEGORY
ORDER BY A.AUTHOR_ID, B.CATEGORY DESC;
```
<br/>
<br/>
<br/>    

---

<br/>
<br/>
<br/>

# 📖 MYSQL 공식문서 정리
## [GROUP BY](https://dev.mysql.com/doc/refman/8.0/en/group-by-handling.html)
### 기본 개념
- 기본적으로 ONLY_FULL_GROUP_BY 모드 활성화
- 🚨 GROUP BY에 포함되지 않은 비집계 컬럼이 있을 경우 오류 발생!!
    - 단, 해당 컬럼이 GROUP BY의 키 값에 의해 유일하게 결정되면 허용 (함수적 종속성)
<br/>
<br/>

### 함수적 종속성(Functional Dependency)
```
SELECT o.custid, c.name, MAX(o.payment)
FROM orders AS o
JOIN customers AS c ON o.custid = c.custid
GROUP BY o.custid;  -- `custid`가 PK라면 `name`도 포함 가능
```

- IF customers 테이블의 custid = 기본 키(PK), **같은 custid에 대해 name 값이 항상 동일하므로** 비집계함수인 name이 SELECT문에 있어도 ㄱㅊ <br/>
🧩 *기본 키* = 테이블에서 각 행을 유일하게 식별할 수 있는 컬럼(또는 컬럼들의 조합)이므로 반드시 함수적 종속성이 있어야 함!
<br/>

```
-- 오류 발생 가능
SELECT name, address, MAX(age) FROM t GROUP BY name;
```
- BUT IF custid is not PK, name이 여러 개의 다른 값을 가질 수 있으므로 **뭘 기준으로 어떤 name을 선택해야 할지 알 수 없음**! GROUP BY에 포함되어야 SELECT문에 쓰일 수 있다<br/>
➡️ 해결방안
    1. **GROUP BY address**
        - 같은 address를 가진 행들끼리 그룹화되므로 하나의 name만 선택 가능
    2. **ANY_VALUE(name)**
        - 같은 custid를 가진 값 중 임의의 한 개 값만 가져오게 됨
<br/>
<br/>

### DISTINCT&ORDER BY 문제
- 🚨 DISTINCT와 ORDER BY를 함께 사용 시, ORDER BY에 포함된 컬럼이 DISTINCT에 없는 경우 MYSQL에서 오류 발생<br/>
    ```SELECT DISTINCT name FROM customers ORDER BY age;```
    - DISTINCT name이므로 결과에는 name 컬럼만 존재
    - BUT ```ORDER BY age```를 하면 MYSQL은 age 값을 이용해서 name 결과값들을 정렬해야되는데, age는 DISTINCT에서 선택되지 않았으므로 어떤 값을 기준으로 정렬해야 할지 알 수 없음!!! 
    🧩 ORDER BY는 SELECT된 컬럼들만 사용 가능

➡️ 해결방안 <br/>
1. ORDER BY 컬럼을 SELECT문에 포함
    - 다만 이 경우 DISTINCT가 name, age 조합에 대해 적용되므로 name이 중복되더라도 age 값이 다르면 중복 제거되지 않을 수도..! (=name,age 조합이 유일해야 DISTINCT로 제거됨)
2. GROUP BY에 해당 컬럼을 포함
    - ```GROUP BY name``` -> 동일한 name 값을 가진 행 중에서 하나만 남김 (하지만 한 개의 값을 남긴다는 것만 확실하고, 어떤 값이 선택될지는 모름🤷‍♀️)
        - 특정 값을 남겨야된다면 ```GROUP BY MIN(age)``` OR ```GROUP BY MAX(age)``` 같은 집계 함수를 써야됨
<br/>
<br/>

### 표현식 사용 (MYSQL ONLY)
- 표준 SQL과 달리 MYSQL에서는 GROUP BY절에서 표현식 ㄱㄴ <br/>
```SELECT id, FLOOR(value/100) FROM tbl_name GROUP BY id, FLOOR(value/100);```<br/>
    - 💡 FLOOR(value/100)이 SELECT문에 포함되어 있으므로 GROUP BY에서도 허용 
```SELECT id, FLOOR(value/100) AS val FROM tbl_name GROUP BY id, val;```
    - 💡 별칭도 사용 가능
<br/>
<br/>
<br/>


## [HAVING절](https://dev.mysql.com/doc/refman/8.0/en/select.html)
- GROUP BY 뒤에, ORDER BY 앞에 위치<br/>
```SELECT col_name FROM tbl_name WHERE col_name > 0;```
- WHERE 대신 쓰지 말기!
    ```
    SELECT user, MAX(salary) 
    FROM users
    GROUP BY user 
    HAVING MAX(salary) > 10;
    ```
    - 단, WHERE절은 집계 함수 참조 못하지만 HAVING은 할 수 있다


# 📝 문제 풀이
### [2. 언어별 개발자 분류하기](https://school.programmers.co.kr/learn/courses/30/lessons/276036)

- 문제 <br/>
DEVELOPERS 테이블에서 GRADE별 개발자의 정보를 조회하려 합니다. GRADE는 다음과 같이 정해집니다.
<br/>
<br/>

A : Front End 스킬과 Python 스킬을 함께 가지고 있는 개발자<br/>
B : C# 스킬을 가진 개발자<br/>
C : 그 외의 Front End 개발자<br/>
GRADE가 존재하는 개발자의 GRADE, ID, EMAIL을 조회하는 SQL 문을 작성해 주세요.<br/>
<br/>

결과는 GRADE와 ID를 기준으로 오름차순 정렬해 주세요.
<br/>
<br/>

- 정답
```
```



