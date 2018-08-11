---  
layout: post  
title: Mssql生成序号的方法  
description: Mssql数据库生成序号的几种方法汇总  
category: database  
---  

## Mssql生成序号的方法  
1. 生成示例数据  
```
CREATE TABLE BD_Score  
(  
        ScoreID    INT IDENTITY(1,1) PRIMARY KEY NOT NULL,  
        Name VARCHAR(20) NOT NULL DEFAULT(''),  
        Subject VARCHAR(50) NOT NULL DEFAULT(''),  
        Score INT NOT NULL DEFAULT(0)  
)  
```
2. 生成测试数据  
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
SELECT '麻六', '语文', 84 UNION ALL  
SELECT '麻六', '语文', 77  
```
3. 数据初步处理  
```
SELECT A.Name, SUM(A.Score) AS TotalScore INTO #Temp FROM dbo.BD_Score AS A GROUP BY A.Name  
```
---
#### 方法一 IDENTITY  
* 此方法缺点是必须要生成一个临时表  
```
SELECT IDENTITY(INT, 1, 1) AS RowNumber, A.Name, A.TotalScore INTO #Temp2 FROM #Temp AS A ORDER BY A.TotalScore DESC  
SELECT * FROM #Temp2  
```
#### 方法二 ROW_NUMBER  
* 顺序生成序号  
```
SELECT ROW_NUMBER() OVER(ORDER BY A.TotalScore DESC) AS RowNum, A.Name, A.TotalScore FROM #Temp AS A  
```
#### 方法三 RANK  
* 相同的值序号会一样，但序号会跳号  
```
SELECT RANK() OVER(ORDER BY A.TotalScore DESC) AS RowNum, A.Name, A.TotalScore FROM #Temp AS A  
```
#### 方法四 DENSE_RANK  
* 相同的值序号会一样，序号顺序递增  
```
SELECT DENSE_RANK() OVER(ORDER BY A.TotalScore DESC) AS RowNum, A.Name, A.TotalScore FROM #Temp AS A  
```
#### 方法五 NTILE  
* NTILE分组依据  
  
1.每组的记录数不能大于它上一组的记录数，即编号小的桶放的记录数不通用小于编号大的桶  
2.所有组中的记录数要么都相同，要么从某一个记录较少的组开始后面的所有组的记录数都与该组的记录数相同。  
```
SELECT NTILE(2) OVER(ORDER BY A.TotalScore DESC) AS RowNum, A.Name, A.TotalScore FROM #Temp AS A  
```
#### 方法六  
* 兼容所有SQL版本的写法  
```
SELECT (SELECT COUNT(*)+1 FROM #Temp AS B WHERE A.TotalScore < B.TotalScore) AS RowNum, A.Name, A.TotalScore FROM #Temp AS A ORDER BY A.TotalScore DESC  
```
---
### 总结  
###### 除方法一、方法六外，其它几种方式都需要SQL版本至少为2005  

