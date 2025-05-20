# 📝 문제 풀이

[복수 국적 메달 수상한 선수 찾기](https://solvesql.com/problems/multiple-medalist/)

```SQL
SELECT a.name
FROM records r
JOIN athletes a ON r.athlete_id = a.id
JOIN teams t ON r.team_id = t.id
JOIN games g ON r.game_id = g.id
WHERE g.year >= 2000
  AND r.medal IS NOT NULL
GROUP BY a.id, a.name
HAVING COUNT(DISTINCT t.team) >= 2
ORDER BY a.name;
```