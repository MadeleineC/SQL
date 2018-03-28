# SQLize

use sakila;

##-- 1a Display the first and last names of all actors from the table `actor`.
select first_name, last_name from actor;

##-- 1b Display the first and last name of each actor in a single column in upper case letters. Name the column `Actor Name`. 
ALTER TABLE actor
ADD COLUMN actor_name VARCHAR(45) NULL AFTER last_update;

SELECT CONCAT(a.first_name, a.last_name) AS actor_name,
       a.*
FROM   actor a;

##-- 2a You need to find the ID number, first name, and last name of an actor, of whom you know only the first name, "Joe." What is one query would you use to obtain this information?
Select first_name, last_name, actor_id
from actor
where first_name = "Joe";

##-- 2b Find all actors whose last name contain the letters `GEN`:
Select *
from actor
where last_name like "%gen%";

##-- 2c Find all actors whose last names contain the letters `LI`. This time, order the rows by last name and first name, in that order:

Select *
from actor
where last_name like "%li%"
ORDER BY last_name, first_name;

##-- 2d Using `IN`, display the `country_id` and `country` columns of the following countries: Afghanistan, Bangladesh, and China:

Select country_id, country
from country where country in("Afghanistan", "Bangladesh", "China");

##-- 3a Add a `middle_name` column to the table `actor`. Position it between `first_name` and `last_name`. Hint: you will need to specify the data type.

ALTER TABLE actor
ADD COLUMN middle_name VARCHAR(45) NULL AFTER first_name;

##-- 3b You realize that some of these actors have tremendously long last names. Change the data type of the `middle_name` column to `blobs`.
ALTER TABLE actor 
CHANGE COLUMN middle_name middle_name BLOB NULL DEFAULT NULL;

##-- 3c Now delete the `middle_name` column.
ALTER TABLE actor 
DROP COLUMN middle_name;

##-- 4a List the last names of actors, as well as how many actors have that last name.
select last_name,
count(*) as num
from actor
group by last_name;

##-- 4b List last names of actors and the number of actors who have that last name, but only for names that are shared by at least two actors
SELECT last_name, COUNT(*)
    FROM actor
    GROUP BY last_name
    HAVING COUNT(*) >= 2;
    
##-- 4c Oh, no! The actor `HARPO WILLIAMS` was accidentally entered in the `actor` table as `GROUCHO WILLIAMS`, the name of Harpo's second cousin's husband's yoga teacher. Write a query to fix the record.
Select *
from actor
where last_name = "Williams";

UPDATE actor
SET first_name = "HARPO"
WHERE actor_id = 172;

##-- 4d Perhaps we were too hasty in changing `GROUCHO` to `HARPO`. It turns out that `GROUCHO` was the correct name after all! In a single query, if the first name of the actor is currently `HARPO`, change it to `GROUCHO`. Otherwise, change the first name to `MUCHO GROUCHO`, as that is exactly what the actor will be with the grievous error. BE CAREFUL NOT TO CHANGE THE FIRST NAME OF EVERY ACTOR TO `MUCHO GROUCHO`, HOWEVER! (Hint: update the record using a unique identifier.)
Select *
from actor
where last_name = "Williams";


UPDATE actor 
SET first_name =  CASE  
	WHEN first_name = "HARPO" THEN 'GROUCHO' 
	WHEN first_name = "GROUCHO" THEN 'MUCHO GROUCHO'
	ELSE first_name
	END 
WHERE actor_id = 172;

Select *
from actor
where last_name = "Williams";

##-- 5a You cannot locate the schema of the `address` table. Which query would you use to re-create it? 
Show create table address;

##-- 6a Use `JOIN` to display the first and last names, as well as the address, of each staff member. Use the tables `staff` and `address`:


SELECT s.first_name, s.last_name, a.address
FROM staff as s
LEFT JOIN address as a ON
s.address_id = a.address_id;

##-- 6b    Use `JOIN` to display the total amount rung up by each staff member in August of 2005. Use tables `staff` and `payment`.
SELECT s.last_name, COUNT(p.payment_id) 
FROM staff as s
LEFT JOIN payment as p ON
s.staff_id = p.staff_id
Group By s.last_name;

##-- --6c List each film and the number of actors who are listed for that film. Use tables `film_actor` and `film`. Use inner join.
SELECT f.title, COUNT(fa.actor_id) 
FROM film as f
INNER JOIN film_actor as fa ON
f.film_id = fa.film_id
Group By f.title;

##-- 6d How many copies of the film `Hunchback Impossible` exist in the inventory system?

SELECT f.title, COUNT(i.film_id	) 
FROM film as f
INNER JOIN inventory as i ON
f.film_id = i.film_id
WHERE f.title = "Hunchback Impossible"
Group By f.title;

##-- 6e. Using the tables `payment` and `customer` and the `JOIN` command, list the total paid by each customer. List the customers alphabetically by last name:

SELECT c.last_name, sum(p.amount) 
FROM customer as c
INNER JOIN payment as p ON
c.customer_id = p.customer_id
Group By c.customer_id;

##-- 7a  The music of Queen and Kris Kristofferson have seen an unlikely resurgence. As an unintended consequence, films starting with the letters `K` and `Q` have also soared in popularity. Use subqueries to display the titles of movies starting with the letters `K` and `Q` whose language is English.
select title
from film
where film_id in(
	select film_id
	from film
	where language_id = 1)
    AND title LIKE 'K%' or title like 'Q%';
    
##-- 7b  Use subqueries to display all actors who appear in the film `Alone Trip`.
   
select first_name, last_name
from actor
where actor_id in (
	select actor_id
	from film_actor
	where film_id in (
		select film_id
		from film
		where title = "Alone Trip")
        );
        
##-- 7c  You want to run an email marketing campaign in Canada, for which you will need the names and email addresses of all Canadian customers. Use joins to retrieve this information.
SELECT c.email
  FROM customer as c
  INNER JOIN address as a ON c.address_id = a.address_id
  INNER JOIN city ON city.city_id = a.city_id
  WHERE city.country_id = 20;
  
##-- 7d Sales have been lagging among young families, and you wish to target all family movies for a promotion. Identify all movies categorized as famiy films.
select title
from film 
where film_id in (
	select film_id
    from film_category
    where category_id in (
		select category_id
        from category
        where name = 'Family'));


##-- --7e Display the most frequently rented movies in descending order.
SELECT f.title, COUNT(r.inventory_id) as num 
FROM film as f
INNER JOIN inventory as i ON f.film_id = i.film_id
INNER JOIN rental as r ON i.film_id = r.inventory_id
Group By f.title
Order by Count(r.inventory_id) DESC;

##-- 7f Write a query to display how much business, in dollars, each store brought in.
select s.store_id, sum(p.amount)
from store as s
INNER JOIN staff ON s.store_id = staff.store_id
INNER JOIN payment as p ON staff.staff_id = p.staff_id
Group by s.store_id;

##-- 7g Write a query to display for each store its store ID, city, and country.
select s.store_id, city.city, country.country
from store as s
join address as a on s.address_id = a.address_id
join city on a.city_id = city.city_id
join country on city.country_id = country.country_id;

##-- 7h List the top five genres in gross revenue in descending order. (**Hint**: you may need to use the following tables: category, film_category, inventory, payment, and rental.)
select category.name, sum(p.amount) as num
from category
join film_category as f on category.category_id = f.category_id
join inventory as i on f.film_id = i.film_id
join rental as r on r.inventory_id = i.inventory_id
join payment as p on p.rental_id = r.rental_id
group by category.name
order by num DESC
limit 5;

##-- 8a  In your new role as an executive, you would like to have an easy way of viewing the Top five genres by gross revenue. Use the solution from the problem above to create a view. If you haven't solved 7h, you can substitute another query to create a view.
create view top_5_genres as 
select category.name, sum(p.amount) as num
from category
join film_category as f on category.category_id = f.category_id
join inventory as i on f.film_id = i.film_id
join rental as r on r.inventory_id = i.inventory_id
join payment as p on p.rental_id = r.rental_id
group by category.name
order by num DESC
limit 5;

##-- 8b How would you display the view that you created in 8a?
select * from top_5_genres;

##-- 8c You find that you no longer need the view `top_five_genres`. Write a query to delete it.
drop view top_5_genres;




