### GROUP BY

GROUP BY 必须包含SELECT后的所有查询列，且SELECT还要有聚合函数

``` sql
SELECT student.sid, student.sname, AVG(result.score) AS avg_score
FROM student
JOIN result on student.sid = result.sid
GROUP BY student.sid, student.sname;
HAVING avg_score < 80;
```

且对分组后的结果进行过滤的话，只能使用HAVING， 因为WHERE只能用在GROUP BY之前

### 给你两张表，一张分班学生表，一张分科成绩表，求每个班总成绩前十名的学生 

可以用WITH建立一个临时表

``` sql
WITH ClassTotalScores AS (
    SELECT
        s.class_id,
        s.student_id,
        SUM(sc.score) AS total_score
    FROM
        students s
    JOIN
        scores sc ON s.student_id = sc.student_id
    GROUP BY
        s.class_id, s.student_id
)
SELECT
    cts.class_id,
    cts.student_id,
    cts.total_score,
    ROW_NUMBER() OVER (PARTITION BY cts.class_id ORDER BY cts.total_score DESC) AS rank
FROM
    ClassTotalScores cts
WHERE
    rank <= 10;

```

