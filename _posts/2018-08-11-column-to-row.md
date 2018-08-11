---  
layout: post  
title: Mssql列转行  
description: 列转行几种方法汇总  
category: database  
---  

## Mssql列转行   
### 生成示例数据  
```  
CREATE TABLE BD_ScoreReset
(
	Name VARCHAR(20) NOT NULL DEFAULT(''),
	Chinese INT DEFAULT(0),
	Math INT DEFAULT(0),
	English INT DEFAULT(0)
) 
```  
### 生成测试数据  
```  
INSERT INTO BD_ScoreReset
SELECT '张三', 80, 93, 88 UNION ALL
SELECT '李四', 87, 68,96 UNION ALL
SELECT '王五', 87, 89, 95  
```  
---
### 方法一 (SQL2000版本) 静态方法  
```
SELECT *
FROM
(
    SELECT A.Name, 'Chinese' AS Subject, A.Chinese AS Score FROM dbo.BD_ScoreReset AS A
    UNION ALL
    SELECT A.Name, 'Math' AS Subject, A.Math AS Score FROM dbo.BD_ScoreReset AS A
    UNION ALL
    SELECT A.Name, 'English' AS Subject, A.English AS Score FROM dbo.BD_ScoreReset AS A
) AS A
ORDER BY A.Name
```

### 方法二 (SQL2000版本) 动态方法
```
DECLARE @strSql VARCHAR(8000)
SELECT @strSql = ISNULL(@strSql + ' UNION ALL ', '') + ' SELECT Name, Subject = ' + QUOTENAME(name, '''') + ', Score = ' + QUOTENAME(name) + ' FROM BD_ScoreReset AS A '
FROM SYSCOLUMNS AS A
WHERE A.name != 'Name' AND A.id = OBJECT_ID('BD_ScoreReset')
ORDER BY A.colid

SELECT @strSql += ' ORDER BY A.Name'

EXEC (@strSql)
```

### 方法三 (SQL2005版本及以上) 静态方法
```
SELECT A.Name, Subject, Score 
FROM dbo.BD_ScoreReset UNPIVOT(Score FOR Subject IN(Chinese, Math, English)) AS A
ORDER BY A.Name
```

### 方法四 (SQL2005版本及以上) 动态方法
```
DECLARE @strSql VARCHAR(8000)
SELECT @strSql = ISNULL(@strSql + ',', '') + QUOTENAME(name)
FROM SYSCOLUMNS
WHERE id = OBJECT_ID('BD_ScoreReset') AND name != 'Name'
ORDER BY colid

SELECT @strSql = 'SELECT Name, Subject, Score FROM BD_ScoreReset AS A UNPIVOT(Score FOR Subject IN(' + @strSql + ')) AS A ORDER BY A.Name'

EXEC (@strSql)
```
---