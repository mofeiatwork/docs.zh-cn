# append_trailing_char_if_absent

## description

### Syntax

```Haskell
VARCHAR append_trailing_char_if_absent(VARCHAR str, VARCHAR trailing_char)
```

如果 str 字符串非空并且末尾不包含 trailing_char 字符，则将 trailing_char 字符附加到末尾。 trailing_char 只能包含一个字符，如果包含多个字符，将返回 NULL

## example

```Plain Text
MySQL [test]> select append_trailing_char_if_absent('a','c');
+------------------------------------------+
|append_trailing_char_if_absent('a', 'c')  |
+------------------------------------------+
| ac                                       |
+------------------------------------------+
1 row in set (0.02 sec)

MySQL [test]> select append_trailing_char_if_absent('ac','c');
+-------------------------------------------+
|append_trailing_char_if_absent('ac', 'c')  |
+-------------------------------------------+
| ac                                        |
+-------------------------------------------+
1 row in set (0.00 sec)
```

## keyword

APPEND_TRAILING_CHAR_IF_ABSENT
