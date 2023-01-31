/* Set1Q1 - Query used for first insight */
WITH t1 AS (
    SELECT f.title film_title, c.name category_name, COUNT(r.rental_id) rental_count

    FROM category c
    JOIN film_category fc
        ON c.category_id = fc.category_id
    JOIN film f
        ON f.film_id = fc.film_id
    JOIN inventory i
        ON i.film_id = f.film_id
    JOIN rental r
        ON r.inventory_id = i.inventory_id
    GROUP BY 1,2 )

SELECT film_title, category_name, rental_count,
CASE WHEN category_name = 'Animation' OR category_name = 'Children' OR category_name = 'Classics' OR category_name = 'Comedy' OR category_name = 'Family' OR category_name = 'Music'
     THEN 'Family Movie' ELSE 'No Family Movie' END AS classification
FROM t1
ORDER BY rental_count DESC;



/* Set1Q3 - Query used for second insight */
WITH t1 AS (
    SELECT c.name category_name, 
    NTILE(4) OVER (ORDER BY f.rental_duration) AS standard_quartile

    FROM category c
    JOIN film_category fc
        ON c.category_id = fc.category_id
    JOIN film f
        ON f.film_id = fc.film_id
    WHERE c.name IN ('Animation','Children','Classics','Comedy','Family','Music') )

 SELECT  category_name, standard_quartile, COUNT(*)
 FROM t1
 GROUP BY category_name, standard_quartile
 ORDER BY category_name, standard_quartile;



/* Set2Q1 - Query used for third insight */
SELECT DATE_PART('month',r.rental_date) AS rental_month, DATE_PART('year',r.rental_date) AS rental_year, s.store_id AS The_store_ID, COUNT(r.rental_id) AS count_rentals
FROM rental r
JOIN staff sf
	ON r.staff_id = sf.staff_id
JOIN store s
	ON s.store_id = sf.store_id
GROUP BY rental_month, rental_year, The_store_ID
ORDER BY count_rentals DESC;



/* Set2Q2 - Query used for fourth insight */
SELECT DATE_TRUNC('month', p.payment_date) AS pay_month, CONCAT(c.first_name,' ',c.last_name) AS full_name, COUNT(p.payment_id) AS pay_countpermon, SUM(p.amount) AS pay_amount
FROM customer c
JOIN payment p
    ON p.customer_id = c.customer_id
WHERE CONCAT(c.first_name,' ',c.last_name) IN

(SELECT t1.all_name
FROM
    (SELECT CONCAT(c.first_name,' ',c.last_name) AS all_name, SUM(p.amount) AS sum_amount 
    FROM customer c
    JOIN payment p
        ON p.customer_id = c.customer_id
    GROUP BY all_name
    ORDER BY sum_amount DESC
    LIMIT 10) t1)

GROUP BY pay_month, full_name
ORDER BY full_name, pay_month;
