---
title: "Prisma 存储日期的最佳实践"
date: 2022-09-06T18:37:59+08:00
draft: false
---

## 问题
业务中常常会需要存储 `YYYY-MM-DD` 这样的日期，使用 Prisma 时的按常理会在 `schema.prisma` 中这样定义。

```
model Model {
  date       DateTime  @db.Date
}
```

但是这却会有很大的问题，PrismaClient 的 `DateTime` 类型只支持传入一个 `Date` 对象或者 `ISO 8601` 格式的字符串，如果传入 `Date` 对象，Prisma 在转化为 SQL 时会自动调用 `toISOString` 方法，执行 `SQL` 之后日期的时区信息就被丢弃掉了。
例如 `new Date('2022-09-01T00:00:00+08:00')` 会被转换为 `2022-08-31T16:00:00.000Z` ，写入数据库后，日期就变成了 `2022-08-31`。


## 最佳实践
将 `YYYY-MM-DD` 类型的日期定义为 `Int` 而不是 `DateTime`
```
model Model {
	date  Int
}
```

使用时 
```typescript
prisma.model.create({
	data: {
    	date: 20220901
    }
})

```
