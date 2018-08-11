---  
layout: post  
title: Mssql行转列  
description: 行转列几种方法汇总  
category: blog  
subtype: database
---  

## Mssql行转列   
### 生成示例数据  
```  
CREATE TABLE BD_Score  
(  
        ScoreID    INT IDENTITY(1,1) PRIMARY KEY NOT NULL,  
        Name VARCHAR(20) NOT NULL DEFAULT(''),  
        Subject VARCHAR(50) NOT NULL DEFAULT(''),  
        Score INT NOT NULL DEFAULT(0)  
)  
```  
### 生成测试数据  
```  
INSERT INTO BD_Score(Name, Subject, Score)  
SELECT '张三', '语文', 80 UNION ALL  
SELECT '张三', '数学', 90 UNION ALL  
SELECT '张三', '英语', 90 UNION ALL  
SELECT '李四', '语文', 78 UNION ALL  
SELECT '李四', '数学', 90 UNION ALL  
SELECT '李四', '英语', 80 UNION ALL  
SELECT '王五', '语文', 73 UNION ALL  
SELECT '王五', '数学', 90 UNION ALL  
SELECT '王五', '英语', 97 UNION ALL  
SELECT '麻六', '语文', 65 UNION ALL  
SELECT '麻六', '数学', 84 UNION ALL  
SELECT '麻六', '英语', 77  
```  
---
### 方法一 (SQL2000版本) 静态方法  
```  
SELECT	A.Name, MAX(CASE A.Subject WHEN '语文' THEN A.Score ELSE 0 END) AS 'Chinese',  
        MAX(CASE A.Subject WHEN '数学' THEN A.Score ELSE 0 END) AS 'Math',  
        MAX(CASE A.Subject WHEN '英语' THEN A.Score ELSE 0 END) AS 'English',  
        SUM(A.Score) AS TotalScore, CAST(AVG(A.Score) AS DECIMAL(3, 1)) AS AvgScore  
FROM BD_Score AS A  
GROUP BY A.Name  
```  

### 方法二 (SQL2000版本) 动态方法
```
DECLARE @strSql VARCHAR(8000)
SELECT @strSql = 'SELECT Name AS 姓名'
SELECT @strSql += ', MAX(CASE A.Subject WHEN ''' + A.Subject + ''' THEN A.Score ELSE 0 END) AS [' + A.Subject + ']'
FROM (SELECT DISTINCT Subject FROM dbo.BD_Score) AS A

SELECT @strSql += ', SUM(A.Score) AS 总分, CAST(AVG(A.Score) AS DECIMAL(3, 1)) AS 平均分 FROM BD_Score AS A GROUP BY A.Name'
EXEC (@strSql)
```

### 方法三 (SQL2005版本及以上) 静态方法
```
SELECT A.Name AS 姓名, A.语文, A.数学, A.英语, B.TotalScore AS 总分, B.AvgScore AS 平均分
FROM dbo.BD_Score AS A PIVOT(MAX(Score) FOR Subject IN(语文,数学,英语)) AS A 
INNER JOIN
(
    SELECT A.Name, SUM(A.Score) AS TotalScore, CAST(AVG(A.Score) AS DECIMAL(3, 1)) AS AvgScore
    FROM dbo.BD_Score AS A
    GROUP BY A.Name
) AS B ON A.Name = B.Name
```

### 方法四 (SQL2005版本及以上) 动态方法
```
DECLARE @strSql VARCHAR(8000)
SELECT @strSql = ISNULL(@strSql + ',', '') + Subject FROM dbo.BD_Score GROUP BY Subject
SELECT @strSql = 'SELECT A.*, B.TotalScore AS 总分, B.AvgScore AS 平均分 FROM BD_Score AS A PIVOT(MAX(Score)
FOR Subject IN('+@strSql+')) AS A
INNER JOIN
(
    SELECT A.Name, SUM(A.Score) AS TotalScore, CAST(AVG(A.Score) AS DECIMAL(3, 1)) AS AvgScore
    FROM dbo.BD_Score AS A
    GROUP BY A.Name
) AS B ON A.Name = B.Name'
EXEC (@strSql)
```
---