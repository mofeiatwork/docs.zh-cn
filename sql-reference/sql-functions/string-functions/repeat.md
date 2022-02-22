# repeat

## description

### Syntax

```Haskell
VARCHAR repeat(VARCHAR str, INT count)
```

将字符串 str 重复 count 次输出，count 小于 1 时返回空串，str，count 任一为 NULL 时，返回 NULL

## example

```Plain Text
MySQL > SELECT repeat("a", 3);
+----------------+
| repeat('a', 3) |
+----------------+
| aaa            |
+----------------+

MySQL > SELECT repeat("a", -1);
+-----------------+
| repeat('a', -1) |
+-----------------+
|                 |
+-----------------+
```

## keyword

REPEAT,
