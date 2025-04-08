## ✅ MySQL 흐름 제어 함수 (Flow Control Functions)
MySQL에서 지원되는 조건에 따라 값을 반환하는 흐름 제어 함수: CASE, IF(), IFNULL(), NULLIF() 

### 📌 CASE
- 문법

```SQL
-- 값 기반
CASE value
    WHEN compare_value THEN result
    [WHEN compare_value THEN result ...]
    [ELSE result]
END

-- 조건 기반
CASE
    WHEN condition THEN result
    [WHEN condition THEN result ...]
    [ELSE result]
END
```

- 값 기반 CASE는 value = compare_value인 첫 번째 조건의 결과를 반환
- 조건 기반 CASE는 condition이 참인 첫 번째 결과를 반환
- ELSE가 없으면 기본적으로 NULL 반환
<br/>
<br/>


### 📌 IF()
```SQL
IF(expr1, expr2, expr3)
```
- expr1이 참이면 expr2, 거짓이면 expr3 반환
- expr1이 NULL이거나 0이면 false로 간주
<br/>

- 예시
```SQL
SELECT IF(1>2, 2, 3);              -- 3
SELECT IF(1<2, 'yes', 'no');       -- 'yes'
SELECT IF(STRCMP('test','test1'),'no','yes'); -- 'no'
```
- 문자열이 하나라도 포함되면 문자열
- 실수 포함 시 실수
- 정수 포함 시 정수

<br/>
<br/>

### 📌 IFNULL()
```SQL
IFNULL(expr1, expr2)
```
- expr1이 NULL이 아니면 그 값을 반환
- NULL이면 expr2 반환
<br/>

- 예시
```SQL
SELECT IFNULL(1, 0);        -- 1
SELECT IFNULL(NULL, 10);    -- 10
SELECT IFNULL(1/0, 10);     -- 10 (1/0은 오류지만 IFNULL은 우회 가능)
```

- 반환 타입
    - STRING > REAL > INTEGER 우선 순위로 더 일반적인 타입이 결과 타입으로 결정
<br/>
<br/>

### 📌 NULLIF()
```SQL
NULLIF(expr1, expr2)
```
- expr1 = expr2이면 NULL 반환
- 다르면 expr1 반환
<br/>

- 예시

```SQL
SELECT NULLIF(1, 1);  -- NULL
SELECT NULLIF(1, 2);  -- 1
```
- expr1이 두 번 평가될 수 있음
- CASE WHEN expr1 = expr2 THEN NULL ELSE expr1 END와 동일


------

## ✅ MySQL 비교 연산자 및 함수 요약

### MySQL 비교 연산자 및 함수 요약표
| 연산자 / 함수              | 설명                                                 | 예시                                  |
|----------------------------|------------------------------------------------------|---------------------------------------|
| `=`                        | 같음                                                 | `WHERE age = 30`                      |
| `!=` 또는 `<>`             | 같지 않음                                            | `WHERE name != 'John'`               |
| `<`                        | 작음                                                 | `WHERE price < 100`                  |
| `<=`                       | 작거나 같음                                          | `WHERE score <= 80`                  |
| `>`                        | 큼                                                   | `WHERE date > '2024-01-01'`          |
| `>=`                       | 크거나 같음                                          | `WHERE salary >= 50000`              |
| `IS NULL`                  | NULL 인지 확인                                       | `WHERE phone IS NULL`                |
| `IS NOT NULL`              | NULL 이 아닌지 확인                                  | `WHERE phone IS NOT NULL`            |
| `BETWEEN a AND b`          | a 이상 b 이하 범위에 포함되는지 확인                 | `WHERE age BETWEEN 20 AND 30`        |
| `IN (list)`                | 주어진 목록 중 하나인지 확인                         | `WHERE country IN ('KR', 'JP')`      |
| `(col1, col2) IN ((...))`  | 여러 열을 동시에 비교 (튜플 비교)                    | `WHERE (x, y) IN ((1, 2), (3, 4))`    |
| `LIKE`                     | 문자열 패턴 비교 (`%`, `_` 와일드카드 사용)           | `WHERE name LIKE 'A%'`               |
| `NOT LIKE`                 | 문자열 패턴이 아닌 경우                              | `WHERE email NOT LIKE '%@gmail.com'` |
| `STRCMP(a, b)`             | 문자열 비교: a < b → -1, a = b → 0, a > b → 1        | `WHERE STRCMP(name, 'John') = 0`     |
| `IS TRUE / IS FALSE`       | Boolean 타입 값 비교                                 | `WHERE is_active IS TRUE`            |
| `NULLIF(a, b)`             | a와 b가 같으면 NULL, 다르면 a 반환                   | `SELECT NULLIF(5, 5)` → `NULL`       |
| `IF(expr, t, f)`           | 조건식 삼항 연산자                                    | `SELECT IF(score > 60, 'Pass', 'Fail')` |

<br/>
<br/>


### 🔎 비교 연산자 결과값
- 1: TRUE
- 0: FALSE
- NULL: 비교 불가 (보통 NULL이 포함될 때)


<br/>
<br/>


### 💡 유의사항
- 문자 vs 숫자 비교: 자동 형변환이 일어남
    - ex. '0.0' = 0 → 1 (문자가 숫자로 변환됨)

- **NULL과 비교 시** 일반 ```=``` 연산자는 NULL 반환 → 확실한 비교 원할 경우 ```<=>``` 사용

- 문자열 비교는 기본적으로 **대소문자 구분 없음**, utf8mb4 문자셋 기준

- IN, BETWEEN 등은 행(row) 비교도 지원됨:
    - (a, b) IN ((1, 2), (3, 4))