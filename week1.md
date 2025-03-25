# 📖 MYSQL 공식문서 정리
## 윈도우 함수
### 14.20.2
- 쿼리 행 집합에 대해 집계 연산, 근데 집계 연산과는 다름
    - 집계 연산: 쿼리 행을 단일 결과 행으로 그룹화
    - 윈도우 함수: 각 쿼리 행에 대한 결과
![SQL1](./image/SQL1.png)
![SQL2](./image/SQL2.png)
```
AVG()
BIT_AND()
BIT_OR()
BIT_XOR()
COUNT()
JSON_ARRAYAGG()
JSON_OBJECTAGG()
MAX()
MIN()
STDDEV_POP(), STDDEV(), STD()
STDDEV_SAMP()
SUM()
VAR_POP(), VARIANCE()
VAR_SAMP()
```

![SQL3](./image/SQL3.png)
![SQL4](./image/SQL4.png)


# 문제 풀이
### 1. Rank Scores 
[문제 링크](https://leetcode.com/problems/rank-scores/description/)
```
SELECT 점수, 
       DENSE_RANK() OVER (ORDER BY 점수 DESC) AS 순위
FROM 점수표;
```

### 2. 다음날도 서울숲의 미세먼지 농도는 나쁨 😢
[문제 링크](https://solvesql.com/problems/bad-finedust-measure/)
```
SELECT
  *
FROM(
    SELECT
        measured_at AS today,
        LEAD(measured_at) OVER() AS next_day,
        pm10,
        LEAD(pm10) OVER() AS next_pm10
    FROM measurements) AS INFO
WHERE
  pm10 < next_pm10
```

### 3. 그룹별 조건에 맞는 식당 목록 출력하기
[문제 링크]()
```
WITH REVIEW_COUNT AS (
    SELECT MEMBER_ID, COUNT(*) AS REVIEW_TOTAL
    FROM REST_REVIEW
    GROUP BY MEMBER_ID
), MAX_REVIEWER AS (
    SELECT MEMBER_ID
    FROM REVIEW_COUNT
    WHERE REVIEW_TOTAL = (SELECT MAX(REVIEW_TOTAL) FROM REVIEW_COUNT)
)


SELECT M.MEMBER_NAME,
    R.REVIEW_TEXT,
    DATE_FORMAT(R.REVIEW_DATE, '%Y-%m-%d') AS "REVIEW_DATE"
FROM REST_REVIEW R
JOIN MEMBER_PROFILE M ON R.MEMBER_ID = M.MEMBER_ID
WHERE R.MEMBER_ID IN (SELECT MEMBER_ID FROM MAX_REVIEWER)
ORDER BY R.REVIEW_DATE ASC, R.REVIEW_TEXT ASC;
```