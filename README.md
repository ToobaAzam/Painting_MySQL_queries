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
******Output*****
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
******Output*****
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

******Output*****
 NO SUCH PAINTINGS AVAILABLE


4.Identify the paintings whose asking price is less than 50% of its regular price

```sql
SELECT *
FROM product_size
WHERE SALE_PRICE < (regular_price*1/2); 
```

******Output*****

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
******Output*****

-- 4896,	48" x 96"(122 cm x 244 cm),	1115





 
