# Painting_MySQL_Queries

This project involves querying an art database using MySQL to answer 22 specific questions.
These questions range from identifying the most expensive canvas size to finding the most 
popular museums and painting styles, removing duplicates, and handling invalid entries.

# Database Overview :
This is a Famous Painting database which have been collected from kaggle: https://www.kaggle.com/datasets/mexwell/famous-paintings 
containg eight tables. They are:

ARTIST
Columns: artist_id, full_name, first_name, middle_names, last_name, nationality, style, birth, death
Description: Contains information about artists including their names, nationality, artistic style, and birth/death dates.

CANVAS_SIZE
Columns: size_id, width, height, label
Description: Details various canvas sizes, including dimensions and descriptive labels.

IMAGE_LINK
Columns: work_id, url, thumbnail_small_url, thumbnail_large_url
Description: Provides URLs for images and thumbnails of the artworks.

MUSEUM_HOURS
Columns: museum_id, day, open, close
Description: Specifies the opening and closing hours of museums for each day of the week.

MUSEUM
Columns: museum_id, name, address, city, state, postal, country, phone, url
Description: Contains detailed information about museums, including their location, contact information, and website.

PRODUCT_SIZE
Columns: work_id, size_id, sale_price, regular_price
Description: Lists available product sizes for artworks along with their sale and regular prices.

SUBJECT
Columns: work_id, subject
Description: Describes the subjects of the artworks.

WORK
Columns: work_id, name, artist_id, style, museum_id
Description: Provides details about individual artworks, including their names, associated artists, artistic styles, and the museums where they are displayed.

# Inspecting the Database:
```sql
SELECT * FROM ARTIST ; -- 421
SELECT * FROM CANVAS_SIZE  ; -- 200
SELECT * FROM IMAGE_LINK ; -- 14,775
SELECT * FROM MUSEUM_HOURS; -- 351
SELECT * FROM MUSEUM; -- 57
SELECT * FROM PRODUCT_SIZE ; -- 110,347
SELECT * FROM SUBJECT; -- 6771
SELECT * FROM WORK; -- 14,776

```
# 22 Queries & It's Solve:

1. Fetch all the paintings which are not displayed on any museums?
   
```sql
SELECT 
    *
FROM
    WORK
WHERE
    MUSEUM_ID IS NULL;

```
*Output*

TOTAL 10223 PAINTINGS ARE NOT DISPLAYED AT ANY MUSEUM

2. Are there museums without any paintings?

```sql
SELECT 
    *
FROM
    MUSEUM M
        LEFT JOIN
    WORK W ON M.MUSEUM_ID = W.MUSEUM_ID
WHERE
    NOT EXISTS( SELECT 
            *
        FROM
            WORK);
```
*Output*

NO SUCH MUSEUMS AVAILABLE

3. How many paintings have an asking price of more than their regular price?
   
```sql
SELECT 
    COUNT(*)
FROM
    product_size
WHERE
    SALE_PRICE > regular_price; 
```

*Output*

 NO SUCH PAINTINGS AVAILABLE


4.Identify the paintings whose asking price is less than 50% of its regular price

```sql
SELECT *
FROM product_size
WHERE SALE_PRICE < (regular_price*1/2); 
```

*Output*

 TOTAL 58 PAINTINGS ARE THERE WHOES ASKING PRICE IS LESS THEN 50%

5. Which canva size costs the most?
   
```sql
SELECT SIZE_ID, LABEL, MAX_PRICE
FROM(
	SELECT 
		C.SIZE_ID, C.LABEL, P.SALE_PRICE AS MAX_PRICE,
		RANK() OVER (ORDER BY P.SALE_PRICE DESC) AS PRICE_RANK 
	FROM CANVAS_SIZE C 
	JOIN PRODUCT_SIZE P 
		ON C.SIZE_ID = P.SIZE_ID) AS RANKED_PRICES
WHERE PRICE_RANK = 1;
```
*Output*

-- 4896,	48" x 96"(122 cm x 244 cm),	1115

6. Delete duplicate records from work, product_size, subject and image_link tables

```sql
delete from work 
	where ctid not in (select min(ctid)
						from work
						group by work_id );

	delete from product_size 
	where ctid not in (select min(ctid)
						from product_size
						group by work_id, size_id );

	delete from subject 
	where ctid not in (select min(ctid)
						from subject
						group by work_id, subject );

	delete from image_link 
	where ctid not in (select min(ctid)
						from image_link
						group by work_id );
 
```

*Output*

Duplicates deleted from work, product_size, subject and image_link tables


7. Identify the museums with invalid city information in the given dataset
```sql
	select * from museum 
	where city ~ '^[0-9]'
```

8. Museum_Hours table has 1 invalid entry. Identify it and remove it.
```sql
	delete from museum_hours 
	where ctid not in (select min(ctid)
						from museum_hours
						group by museum_id, day );
```

9. Fetch the top 10 most famous painting subject
```sql
	select * 
	from (
		select s.subject,count(1) as no_of_paintings
		,rank() over(order by count(1) desc) as ranking
		from work w
		join subject s on s.work_id=w.work_id
		group by s.subject ) x
	where ranking <= 10;
```

10. Identify the museums which are open on both Sunday and Monday. Display museum name, city.
	
 ```sql
 	select distinct m.name as museum_name, m.city, m.state,m.country
	from museum_hours mh 
	join museum m on m.museum_id=mh.museum_id
	where day='Sunday'
	and exists (select 1 from museum_hours mh2 
				where mh2.museum_id=mh.museum_id 
			    and mh2.day='Monday');

``` 
11. How many museums are open every single day?

```sql
	select count(1)
	from (select museum_id, count(1)
		  from museum_hours
		  group by museum_id
		  having count(1) = 7) x;

```

12. Which are the top 5 most popular museum? (Popularity is defined based on most no of paintings in a museum)

```sql
	select m.name as museum, m.city,m.country,x.no_of_painintgs
	from (	select m.museum_id, count(1) as no_of_painintgs
			, rank() over(order by count(1) desc) as rnk
			from work w
			join museum m on m.museum_id=w.museum_id
			group by m.museum_id) x
	join museum m on m.museum_id=x.museum_id
	where x.rnk<=5;
```

13. Who are the top 5 most popular artist? (Popularity is defined based on most no of paintings done by an artist)
```sql
 	select a.full_name as artist, a.nationality,x.no_of_painintgs
	from (	select a.artist_id, count(1) as no_of_painintgs
			, rank() over(order by count(1) desc) as rnk
			from work w
			join artist a on a.artist_id=w.artist_id
			group by a.artist_id) x
	join artist a on a.artist_id=x.artist_id
	where x.rnk<=5;
```

14. Display the 3 least popular canva sizes
```sql
	select label,ranking,no_of_paintings
	from (
		select cs.size_id,cs.label,count(1) as no_of_paintings
		, dense_rank() over(order by count(1) ) as ranking
		from work w
		join product_size ps on ps.work_id=w.work_id
		join canvas_size cs on cs.size_id::text = ps.size_id
		group by cs.size_id,cs.label) x
	where x.ranking<=3;

```
15. Which museum is open for the longest during a day. Dispay museum name, state and hours open and which day?

```sql
	select museum_name,state as city,day, open, close, duration
	from (	select m.name as museum_name, m.state, day, open, close
			, to_timestamp(open,'HH:MI AM') 
			, to_timestamp(close,'HH:MI PM') 
			, to_timestamp(close,'HH:MI PM') - to_timestamp(open,'HH:MI AM') as duration
			, rank() over (order by (to_timestamp(close,'HH:MI PM') - to_timestamp(open,'HH:MI AM')) desc) as rnk
			from museum_hours mh
		 	join museum m on m.museum_id=mh.museum_id) x
	where x.rnk=1;
```


16. Which museum has the most no of most popular painting style?
```sql
	with pop_style as 
			(select style
			,rank() over(order by count(1) desc) as rnk
			from work
			group by style),
		cte as
			(select w.museum_id,m.name as museum_name,ps.style, count(1) as no_of_paintings
			,rank() over(order by count(1) desc) as rnk
			from work w
			join museum m on m.museum_id=w.museum_id
			join pop_style ps on ps.style = w.style
			where w.museum_id is not null
			and ps.rnk=1
			group by w.museum_id, m.name,ps.style)
	select museum_name,style,no_of_paintings
	from cte 
	where rnk=1;
```

17. Identify the artists whose paintings are displayed in multiple countries
```sql
	with cte as
		(select distinct a.full_name as artist
		--, w.name as painting, m.name as museum
		, m.country
		from work w
		join artist a on a.artist_id=w.artist_id
		join museum m on m.museum_id=w.museum_id)
	select artist,count(1) as no_of_countries
	from cte
	group by artist
	having count(1)>1
	order by 2 desc;
```

18. Display the country and the city with most no of museums. Output 2 seperate columns to mention the city and country. If there are multiple value, seperate them with comma.
```sql
	with cte_country as 
			(select country, count(1)
			, rank() over(order by count(1) desc) as rnk
			from museum
			group by country),
		cte_city as
			(select city, count(1)
			, rank() over(order by count(1) desc) as rnk
			from museum
			group by city)
	select string_agg(distinct country.country,', '), string_agg(city.city,', ')
	from cte_country country
	cross join cte_city city
	where country.rnk = 1
	and city.rnk = 1;
```sql

19. Identify the artist and the museum where the most expensive and least expensive painting is placed. 
Display the artist name, sale_price, painting name, museum name, museum city and canvas label
```sql
	with cte as 
		(select *
		, rank() over(order by sale_price desc) as rnk
		, rank() over(order by sale_price ) as rnk_asc
		from product_size )
	select w.name as painting
	, cte.sale_price
	, a.full_name as artist
	, m.name as museum, m.city
	, cz.label as canvas
	from cte
	join work w on w.work_id=cte.work_id
	join museum m on m.museum_id=w.museum_id
	join artist a on a.artist_id=w.artist_id
	join canvas_size cz on cz.size_id = cte.size_id::NUMERIC
	where rnk=1 or rnk_asc=1;
```

20. Which country has the 5th highest no of paintings?
```sql
 	with cte as 
		(select m.country, count(1) as no_of_Paintings
		, rank() over(order by count(1) desc) as rnk
		from work w
		join museum m on m.museum_id=w.museum_id
		group by m.country)
	select country, no_of_Paintings
	from cte 
	where rnk=5;
```sql

21. Which are the 3 most popular and 3 least popular painting styles?
```sql
	with cte as 
		(select style, count(1) as cnt
		, rank() over(order by count(1) desc) rnk
		, count(1) over() as no_of_records
		from work
		where style is not null
		group by style)
	select style
	, case when rnk <=3 then 'Most Popular' else 'Least Popular' end as remarks 
	from cte
	where rnk <=3
	or rnk > no_of_records - 3;
```

22. Which artist has the most no of Portraits paintings outside USA?. Display artist name, no of paintings and the artist nationality.
```sql
	select full_name as artist_name, nationality, no_of_paintings
	from (
		select a.full_name, a.nationality
		,count(1) as no_of_paintings
		,rank() over(order by count(1) desc) as rnk
		from work w
		join artist a on a.artist_id=w.artist_id
		join subject s on s.work_id=w.work_id
		join museum m on m.museum_id=w.museum_id
		where s.subject='Portraits'
		and m.country != 'USA'
		group by a.full_name, a.nationality) x
	where rnk=1;	

```





