---
title: pandas 使用技巧总结（持续更新）
categories: ["python","pandas","DataFrame"]
tags: ["python","数据分析"]
date: 2019-06-31 20:54:41
---

### pandas 对指定列做fillna

```
df.fillna({'code':'code', 'date':'date'})
df.[["code", "date"]].fillna("")
```

### pandas 指定列重命名

```
df.rename(columns={"amount": "total_amount"}, inplace=True)
```

### DataFrame 按直接列left join合并

```
df = pd.merge(df_1, df_2, on=["code", "date"], how='left')
```

### DataFrame 两列相加相减

```
df["amount"] = df["total_amount"] - df["amount"]
df["amount"] = df["total_amount"] + df["amount"]
```

