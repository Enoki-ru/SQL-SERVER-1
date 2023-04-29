Всем снова привет!
Сегодня мы создадим свою базу данных, на которой потом будем обучаться работать, изменять таблицы, строить возможно схемы, и нормализовывать данные.
Поехали!

---
1. Запустим Microsoft SSMS
![[Pasted image 20230428170711.png]]

2. Создадим командой новую БД, назовем ее video_games:
```sql
CREATE DATABASE video_games;
```
![[Pasted image 20230428172028.png]]


3. Скачиваем уже готовые таблицы из интернета (для обучения в самый раз)
Сделаем это по готовым примерам таблиц, скачать можно, к примеру, по ссылке
https://github.com/bbrumm/databasestar/tree/main/sample_databases/sample_db_videogames
![[Pasted image 20230428171317.png]]

5. Поочередно открываем их на сервере, и выполняем программы по заполнению новых таблиц

6. Рассмотрим схему всех таблиц:

![[Pasted image 20230428173440.png]]

Посмотрим на парочку из них и убедимся, что все они составлены именно так (вдруг обманывают)

![[Pasted image 20230428173507.png]]

![[Pasted image 20230428173518.png]]

Всё сходится! Можем начинать строить себе задачи, которые могут давать нам работодатели 

---
Задание №1.
Создайте новое представление, которое будет показывать только игры для ПК платформ, выпущенных до 2002 года

Комментарий:
```
Мда уж, сам себе поставил непростую задачу.
Давайте обратимся к заранее изученным познаниям того, что такое представление, и в особенности, как его использовать в T-SQL (если кто не понял пока мы работаем в проге от Microsoft тут своя СУБД, но разницы особой нет).
Во время подготовки к одному из собедеований, которое так и не состоялось, я выписал следующие вещи:
Представление представляет собой хранимый запрос к базе данных, также ее можно назвать виртуальная таблица, но в этой таблице данные не хранятся, а хранится только сам запрос. Но, тем не менее, к вьюшке можно обращаться как к обычной таблице и извлекать данные из нее.
```

Например, в [PostgreSQL](https://info-comp.ru/sisadminst/598-install-postgresql-10-on-ubuntu-server.html "Установка и настройка PostgreSQL 10 на Linux Ubuntu Server") запрос создания представления будет выглядеть так:
   
```mysql
CREATE VIEW MyView 
AS
	SELECT id, name, org
	FROM work.TableName
```

Полный синтаксис выгладит следующим образом:
```mysql
CREATE [OR REPLACE]  
[ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]  
VIEW view_name [(column_list)]  
AS select_statement  
[WITH [CASCADED | LOCAL] CHECK OPTION]
```

Попробуем для начала реализовать запрос с выводом топа, а потом уже будем запихивать его в представление, хорошо? Надеюсь, что да.

**Решение**

Если немного покопаться, то можно прийти к такой команде:
```sql
SELECT g.game_name Игра,gp.release_year Год_выпуска, p.platform_name Платформа
FROM dbo.game g
JOIN dbo.game_platform gp
ON g.id=gp.id
JOIN dbo.platform p
ON gp.platform_id=p.id
WHERE p.platform_name='PC'
AND gp.release_year<2002
```

![[Pasted image 20230428182200.png]]

Осталось только создать представление.

```sql
CREATE OR ALTER VIEW games_2002_pc
AS
	SELECT g.game_name Игра,gp.release_year Год_выпуска, p.platform_name Платформа
	FROM dbo.game g
	JOIN dbo.game_platform gp
	ON g.id=gp.id
	JOIN dbo.platform p
	ON gp.platform_id=p.id
	WHERE p.platform_name='PC'
	AND gp.release_year<2002

```

![[Pasted image 20230428183001.png]]
Готово! Теперь у нас есть представление, которое можно использовать в своих нуждах.

Можно теперь подставить это представление в команду, и поделать с этим что-нибудь.
![[Pasted image 20230428183543.png]]




Задача 2.
Выведите самые продаваемые игры для каждой компании в БД.
Таблица должна выглядеть:
publisher_name, best_game, global_sales.

Решение:

Давайте посмотрим на схеме, как нам решать эту задачу:
![[Pasted image 20230429143159.png]]

Решение:
Делал я это почему-то долго, но написал в итоге только через WITH.

```mysql
WITH task2
AS
	(SELECT game_name,publisher_name,num_sales
	FROM dbo.publisher pub
	JOIN dbo.game_publisher  gpub ON gpub.publisher_id=pub.id
	JOIN dbo.game_platform gpl ON gpl.game_publisher_id=gpub.id
	JOIN dbo.region_sales rsales ON rsales.game_platform_id=gpl.id
	JOIN dbo.game game ON game.id=gpub.game_id
	),
task2_2
AS 
	(SELECT game_name best_game, SUM(num_sales) global_sales
	FROM task2
	GROUP BY game_name),
t3
AS (SELECT publisher_name,MAX(global_sales) best_sale
FROM task2_2 t2
JOIN task2 t ON t.game_name=t2.best_game
GROUP BY publisher_name)

SELECT DISTINCT t3.publisher_name, t2.best_game, t3.best_sale
FROM task2
JOIN task2_2 t2 ON t2.best_game=task2.game_name
JOIN t3 ON task2.publisher_name=t3.publisher_name
WHERE t2.global_sales=t3.best_sale
ORDER BY t3.best_sale DESC
```


![[Pasted image 20230429155922.png]]