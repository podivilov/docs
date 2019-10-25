# Оконные функции в YQL

## Агрегатные функции {#aggregate-functions}

Все [агрегатные функции](aggregation.md) также могут использоваться в роли оконных. В этом случае на каждой строке оказывается результат агрегации, примененный к текущему положению окна **(по умолчанию от начала партиции до текущей строки, включительно)**.

**Примеры**

```sql
SELECT
    SUM(int_column) OVER w AS running_total
FROM my_table
WINDOW w AS ();
```

## ROW_NUMBER {#row_number}

Номер строки в рамках партиции. Без аргументов.

**Примеры**

```sql
SELECT
    ROW_NUMBER() OVER w AS row_num
FROM my_table
WINDOW w AS ();
```

## LAG / LEAD {#lag-lead}

Доступ к значению из строки, отстающей (`LAG`) или опережающей (`LEAD`) текущую на фиксированное число. В первом аргументе указывается выражение, к которому необходим доступ, а во втором — отступ в строках. Отступ можно не указывать, по умолчанию используется соседняя строка — предыдущая или следующая, соответственно, то есть подразумевается 1. В строках, для которых нет соседей с заданным расстоянием (например `LAG(expr, 3)` в первой и второй строках окна) значение выражения - `NULL`.

**Примеры**

```sql
SELECT
   int_value - LAG(int_value) OVER w AS int_value_diff
FROM my_table
WINDOW w AS ();
```

{% if audience == "internal" %}

[Пример операции](https://yql.yandex-team.ru/Operations/WmBLikeUGe7ks4bOu_B5TS1gctsF7EgRNztKCMY0IKs)

{% endif %}

## FIRST_VALUE / LAST_VALUE {#first-last-value}

Доступ к значениям из первой и последней строк окна. Единственный аргумент - выражение, к которому необходим доступ.

Опционально перед `OVER` может указываться дополнительный модификатор `IGNORE NULLS`, который меняет поведение функций на первое или последнее __не пустое__ (то есть не `NULL`) значение среди строк окна. Антоним этого модификатора — `RESPECT NULLS` является поведением по умолчанию и может не указываться.

**Примеры**

```sql
SELECT
   LAST_VALUE(my_column) IGNORE NULLS OVER w
FROM my_table
WINDOW w AS ();
```

## RANK / DENSE_RANK {#rank}

Пронумеровать группы соседних строк с одинаковыми значением выражения в аргументе. `DENSE_RANK` нумерует группы подряд, а `RANK` — пропускает `(N - 1)` значений, где `N` — число строк в предыдущей группе.

При отсутствии аргумента использует порядок, указанный в секции ORDER BY.

**Примеры**

```sql
SELECT
   RANK(my_column) OVER w
FROM my_table
WINDOW w AS ();
```

```sql
SELECT
   RANK() OVER w
FROM my_table
WINDOW w AS (ORDER BY my_column);
```