# 📖 MYSQL 공식문서 정리
## [정규 표현식](https://dev.mysql.com/doc/refman/8.0/en/regexp.html)

- 개념
    - 복잡한 문자열 패턴 지정하고 검색하는 강력한 도구

- 주요 함수 및 연산자<br/>

| 이름              | 설명                                               |
|-------------------|----------------------------------------------------|
| `REGEXP` / `RLIKE` | 패턴과 문자열이 일치하는지 확인                    |
| `NOT REGEXP`       | 위 결과의 부정                                     |
| `REGEXP_LIKE()`    | `REGEXP`와 동일 (함수 형태)                         |
| `REGEXP_INSTR()`   | 패턴이 처음 일치하는 위치(인덱스) 반환             |
| `REGEXP_REPLACE()` | 패턴과 일치하는 부분을 다른 문자열로 치환         |
| `REGEXP_SUBSTR()`  | 패턴과 일치하는 하위 문자열 반환                   |


- ✅ REGEXP / RLIKE
    - 일치하면 1, 아니면 0 반환
    - NULL 값 포함 시 NULL 반환
    ```sql
    SELECT 'Michael!' REGEXP '.*'; -- 1 (모든 문자열에 일치)
    SELECT 'a' REGEXP '^[a-d]'; -- 1 (a는 a-d 범위에 있음)
    ```

- ✅ REGEXP_INSTR(expr, pat[, pos[, occurrence[, return_option[, match_type]]]])
    - pat와 일치하는 시작 위치(인덱스) 반환 (1부터 시작)
    - 일치하는게 없으면 0
    - 옵션
        - pos: 검색 시작 위치 (기본값 1)
        - occurrence: 몇 번째 일치를 찾을지 (기본값 1)
        - return_option: 0이면 시작 위치, 1이면 끝난 후 위치
        - match_type: REGEXP_LIKE()의 매칭 방식 사용
    ```SQL
    SELECT REGEXP_INSTR('dog cat dog', 'dog'); -- 1
    SELECT REGEXP_INSTR('dog cat dog', 'dog', 2); -- 9
    SELECT REGEXP_INSTR('aa aaa aaaa', 'a{4}'); -- 8
    ```

- ✅ REGEXP_LIKE(expr, pat[, match_type])   
    - expr이 pat 정규표현식과 일치하면 1, 아니면 0
    - 옵션
        - match_type: 매칭 방식 제어
    ```SQL
    SELECT REGEXP_LIKE('abc', 'ABC'); -- 1 (기본적으로 대소문자 무시)
    SELECT REGEXP_LIKE('abc', 'ABC', 'c'); -- 0 (대소문자 구분)
    ```
    > ⚠️ 정규표현식 안의 \는 문자열 이스케이프 때문에 \\로 작성해야 함


- MYSQL 정규 표현식 문법
    - 위치 지정자
        - ^: 문자열 시작과 일치
        - ^fo → "fo"로 시작하는 문자열
        - $: 문자열 끝과 일치
        - fo$ → "fo"로 끝나는 문자열

    - 임의 문자 및 줄 처리
        - .: 모든 문자 1개 (줄바꿈 제외)
        - (?m): 다중 줄 모드 활성화 (줄바꿈 포함해서 ^, $ 사용 가능)

    - 수량자
        - a*: a가 0개 이상
        - a+: a가 1개 이상
        - a?: a가 0개 또는 1개
        - a{n}: 정확히 n개
        - a{n,}: n개 이상
        - a{m,n}: m개 이상 n개 이하

    - 선택(OR)
        - a|b: a 또는 b 중 하나

    - 그룹화
        - (abc)*: abc라는 패턴이 0번 이상 반복

    - 문자 클래스
        - [abc]: a, b, c 중 하나
        - [a-z]: 소문자 알파벳 중 하나
        - [^abc]: a, b, c 제외한 문자

    - 문자 클래스 (POSIX 스타일)
        - [[:digit:]]: 숫자 (0-9)
        - [[:alpha:]]: 알파벳 문자
        - [[:alnum:]]: 알파벳 또는 숫자
        - [[:space:]]: 공백 문자

    ⚠️ 성능 팁: [:digit:]보다 [0-9]가 빠름 (MySQL 8.0에서 성능 최적화됨)

    - 특수문자 이스케이프
        - +, ., *, ? 등 특수문자를 문자로 인식하게 하려면 \\를 두 번 사용
        - 예: '1\\+2' 는 문자열 "1+2"를 정확히 매칭


# 📝 문제 풀이
### [서울에 위치한 식당 목록 출력하기](https://school.programmers.co.kr/learn/courses/30/lessons/131118)

- 정규표현식 사용하지 않은 버전
```SQL
SELECT
    I.REST_ID,
    REST_NAME,
    FOOD_TYPE,
    FAVORITES,
    ADDRESS,
    ROUND(AVG(REVIEW_SCORE), 2) AS SCORE
FROM REST_INFO I
JOIN REST_REVIEW R
ON I.REST_ID = R.REST_ID
WHERE I.ADDRESS LIKE '서울%'
GROUP BY I.REST_ID, REST_NAME, FOOD_TYPE, FAVORITES, ADDRESS
ORDER BY SCORE DESC, FAVORITES DESC
```

- 정규표현식 사용한 버전
```SQL
SELECT
    RI.REST_ID,
    RI.REST_NAME,
    RI.FOOD_TYPE,
    RI.FAVORITES,
    RI.ADDRESS,
    ROUND(AVG(RR.REVIEW_SCORE), 2) AS AVG_SCORE
FROM REST_INFO RI
JOIN REST_REVIEW RR ON RI.REST_ID = RR.REST_ID
WHERE RI.ADDRESS REGEXP '^서울'
GROUP BY
    RI.REST_ID,
    RI.REST_NAME,
    RI.FOOD_TYPE,
    RI.FAVORITES,
    RI.ADDRESS
ORDER BY AVG_SCORE DESC, RI.FAVORITES DESC;
```

------

# 📖 MYSQL 공식문서 정리
## [비트 연산자](https://dev.mysql.com/doc/refman/8.0/en/bit-functions.html)

- 개념
    - 비트 연산자: &, ㅣ, ^, ~, <<, >>
    - 연산 대상의 타입에 따라 처리 방식이 달라짐
        - 숫자: 64비트 부호 없는 정수로 변환하여 계산
        - 바이너리 문자열: 입력 길이를 그대로 유지하며 바이너리 연산 수행

- 비트 연산자
> Bitwise OR
- 숫자: 각 비트를 OR 연산

```SQL
SELECT 29 | 15; -- 결과: 31
```
<br/>

- 바이너리 문자열: 길이가 다르면 오류 발생

```SQL
SELECT _binary X'40404040' | X'01020304'; -- 결과: 'ABCD'
```

> & Bitwise AND
- 숫자: 각 비트를 AND 연산

```SQL
SELECT 29 & 15; -- 결과: 13
```
<br/>

> 바이너리 문자열

```SQL
SELECT HEX(_binary X'FF' & b'11110000'); -- 결과: 'F0'
```
<BR/>

> ^ Bitwise XOR
- 숫자: XOR 연산

```SQL
SELECT 11 ^ 3; -- 결과: 8
```
<br/>

> 바이너리 문자열

```SQL
SELECT HEX(_binary X'FEDC' ^ X'1111'); -- 결과: 'EFCD'
```
<BR/>

> << Left Shift
- 숫자: 왼쪽으로 비트 이동 (overflow 비트는 삭제됨)

```SQL
SELECT 1 << 2; -- 결과: 4
```
<BR/>

> 바이너리 문자열

```SQL
SELECT HEX(_binary X'00FF00FF00FF' << 8); -- 결과: 'FF00FF00FF00'
```
<BR/>

> '>> Right Shift'
숫자: 오른쪽으로 비트 이동

```SQL
SELECT 4 >> 2; -- 결과: 1
```
<br/>

> 바이너리 문자열

```SQL
SELECT HEX(_binary X'00FF00FF00FF' >> 8); -- 결과: '0000FF00FF00'
```
<BR/>

> ~ Bitwise NOT (비트 반전)
    - 숫자: 모든 비트를 반전

```SQL
SELECT 5 & ~1; -- 결과: 4
```
<br/>

> 바이너리 문자열

```SQL
SELECT HEX(~X'0000FFFF1111EEEE'); -- 결과: 'FFFF0000EEEE1111'
```
<BR/>

- 🔢 BIT_COUNT(N) 함수
    - 기능: 숫자 또는 문자열에서 1로 설정된 비트 수를 반환
    - 예시
    ```SQL
    SELECT BIT_COUNT(64);           -- 결과: 1
    SELECT BIT_COUNT(BINARY 64);    -- 결과: 7
    SELECT BIT_COUNT(X'40');        -- 결과: 1
    SELECT BIT_COUNT(_binary X'40');-- 결과: 1
    ```

- ⚠️ 주의사항
    - MySQL 8.0부터는 BINARY, VARBINARY, BLOB과 같은 바이너리 타입도 지원
    - MySQL 5.7과의 호환성 문제가 발생할 수 있으므로 마이그레이션 시 주의
    - 바이너리 연산 시 길이 불일치 → ER_INVALID_BITWISE_OPERANDS_SIZE 오류 발생


# 📝 문제 풀이
### [부모의 형질을 모두 가지는 대장균 찾기](https://school.programmers.co.kr/learn/courses/30/lessons/301647)

```SQL
SELECT 
    E1.ID, 
    E1.GENOTYPE, 
    E2.GENOTYPE AS PARENT_GENOTYPE
FROM ECOLI_DATA E1
JOIN ECOLI_DATA E2 ON E1.PARENT_ID = E2.ID
WHERE (E1.GENOTYPE & E2.GENOTYPE) = E2.GENOTYPE
ORDER BY E1.ID;
```