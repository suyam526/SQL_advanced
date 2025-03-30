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
    - 두 테이블에 **공통으로 존재하는 열으로 조인**할 때 사용
    ```a LEFT JOIN b USING (공통 컬럼)```
    =
    ```a LEFT JOIN b ON a.c1 = b.c1 AND a.c2 = b.c2 AND a.c3 = b.c3```

- NATURAL JOIN
    - 두 테이블에 공통된 컬럼을 **자동으로 찾아** 조인
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
    

