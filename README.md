# Test-Stat4Market

## PHP/PostgreSQL
### Задача 1

- Выборки пользователей, у которых количество постов больше, чем у пользователя их пригласившего:
```sql
SELECT u1.id, u1.name
FROM users u1
INNER JOIN users u2 ON u1.invited_by_user_id = u2.id
WHERE u1.posts_qty > u2.posts_qty;
```
- Выборки пользователей, имеющих максимальное количество постов в своей группе:
```sql
SELECT u.id, u.name, u.posts_qty
FROM users u
WHERE (u.group_id, u.posts_qty) IN (
    SELECT group_id, MAX(posts_qty) AS max_posts
    FROM users
    GROUP BY group_id
);
```
- Выборки групп, количество пользователей в которых превышает 10000:
```sql
SELECT g.id, g.name
FROM groups g
WHERE (SELECT COUNT(*) FROM users u WHERE u.group_id = g.id) > 10000;
```
- Выборки пользователей, у которых пригласивший их пользователь из другой группы:
```sql
SELECT u.id, u.name
FROM users u
INNER JOIN users u2 ON u.invited_by_user_id = u2.id
WHERE u.group_id <> u2.group_id;
```
- Выборки групп с максимальным количеством постов у пользователей:
```sql
WITH GroupPostCounts AS (
    SELECT g.id AS group_id, g.name AS group_name, SUM(u.posts_qty) AS total_posts
    FROM groups g
    JOIN users u ON g.id = u.group_id
    GROUP BY g.id, g.name
)
SELECT group_id, group_name, total_posts
FROM (
    SELECT *, RANK() OVER (ORDER BY total_posts DESC) AS rank
    FROM GroupPostCounts
) ranked_groups
WHERE rank = 1;
```
### Задача 2
- Добавление трех полей:
```sql
ALTER TABLE your_table
ADD COLUMN new_column1 data_type,
ADD COLUMN new_column2 data_type,
ADD COLUMN new_column3 data_type;
```
- Изменение одного поля:
```sql
ALTER TABLE your_table
ALTER COLUMN existing_column SET DATA TYPE new_data_type;
```
- Добавление индексов:
```sql
CREATE INDEX CONCURRENTLY index_name1 ON your_table (column1);
CREATE INDEX CONCURRENTLY index_name2 ON your_table (column2);
```
Перед выполнением данных операций на большой БД (размером свыше 100 ГБ и более 8 миллионов строк), рекомендуется выполнить следущие действия:
- Создание резервной копии БД - обеспечит безопасность в случае неожиданных проблем.
- Оценка воздействия. Проанализируйте ожидаемое воздействие операций добавления полей и индексов на производительность. Оцените затраты на время и ресурсы.
- Тестирование на тестовой копии. Прежде чем выполнять операции в продакшн, проведите тестирование на копии базы данных с аналогичной структурой и объемом данных, чтобы убедиться, что операции выполняются без ошибок и в приемлемые сроки.
- Если это возможно, выполните операции поэтапно. Например, сначала добавляем новые поля, а затем создаем индексы. Это может снизить нагрузку на БД.
### Задача 3
```PHP
function countTuesdaysBetweenDates($startDate, $endDate) {
    $start = new DateTime($startDate);
    $end = new DateTime($endDate);

    // Ближайший вторник не позже начальной даты
    while ($start->format('N') != 2) {
        $start->modify('+1 day');
    }

    // Разница между датами с шагом в неделю
    $interval = $start->diff($end)->days;
    $tuesdayCount = floor($interval / 7);

    return $tuesdayCount;
}

$startDate = '2023-10-01';
$endDate = '2023-11-01';

$count = countTuesdaysBetweenDates($startDate, $endDate);
echo "Количество вторников между $startDate и $endDate: $count";
```