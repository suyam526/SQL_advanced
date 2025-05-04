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
-- 기본 사용
SELECT student_name, GROUP_CONCAT(test_score)
FROM student
GROUP BY student_name;

-- DISTINCT, 정렬, 구분자 지정
SELECT student_name,
       GROUP_CONCAT(DISTINCT test_score ORDER BY test_score DESC SEPARATOR ' ')
FROM student
GROUP BY student_name;
```
<BR/>

- SEPARATOR : 값 사이의 구분자 지정 (기본: ,)
