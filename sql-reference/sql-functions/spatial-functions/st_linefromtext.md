# ST_LineFromText, ST_LineStringFromText

## description

### Syntax

```Haskell
GEOMETRY ST_LineFromText(VARCHAR wkt)
```

将一个 WKT（Well Known Text）转化为一个 Line 形式的内存表现形式

## example

```Plain Text
MySQL > SELECT ST_AsText(ST_LineFromText("LINESTRING (1 1, 2 2)"));
+---------------------------------------------------------+
| st_astext(st_geometryfromtext('LINESTRING (1 1, 2 2)')) |
+---------------------------------------------------------+
| LINESTRING (1 1, 2 2)                                   |
+---------------------------------------------------------+
```

## keyword

ST_LINEFROMTEXT, ST_LINESTRINGFROMTEXT, ST, LINEFROMTEXT, LINESTRINGFROMTEXT
