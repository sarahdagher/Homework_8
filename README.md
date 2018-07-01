# Homework_8
MySql Queries

USE	sakila;

#Display the first and last names of all actors from the table `actor`.
SELECT first_name, last_name
FROM actor;

#Display the first and last name of each actor in a single column in upper case letters. Name the column `Actor Name`.
SELECT UPPER(CONCAT_WS('',first_name,last_name)) as 'Actor Name'
FROM actor;

#You need to find the ID number, first name, and last name of an actor, of whom you know only the first name, "Joe." What is one query would you use to obtain this information?
SELECT actor_id, first_name, last_name
FROM actor
WHERE first_name LIKE "Joe%";

#Find all actors whose last name contain the letters `GEN`
SELECT last_name
FROM actor
WHERE last_name LIKE "%GEN%";

#Find all actors whose last names contain the letters `LI`. This time, order the rows by last name and first name, in that order
SELECT last_name, first_name
FROM actor
WHERE last_name LIKE "%LI%"
ORDER BY last_name;

#Using `IN`, display the `country_id` and `country` columns of the following countries: Afghanistan, Bangladesh, and China:
SELECT country_id, country
FROM country
WHERE country IN ('Afghanistan', 'Bangladesh', 'China');

#Add a `middle_name` column to the table `actor`. Position it between `first_name` and `last_name`. Hint: you will need to specify the data type.
ALTER TABLE actor
ADD COLUMN middle_name VARCHAR(30) 
AFTER first_name;

#You realize that some of these actors have tremendously long last names. Change the data type of the `middle_name` column to `blobs`.
ALTER TABLE actor
MODIFY middle_name BLOB;

#Now delete the `middle_name` column.
ALTER TABLE actor
DROP COLUMN middle_name;

#List the last names of actors, as well as how many actors have that last name.
SELECT last_name, count(*) FROM actor
GROUP BY last_name;

#List last names of actors and the number of actors who have that last name, but only for names that are shared by at least two actors
SELECT last_name, count(last_name) AS total FROM actor
GROUP BY last_name HAVING TOTAL >2;

#Oh, no! The actor `HARPO WILLIAMS` was accidentally entered in the `actor` table as `GROUCHO WILLIAMS`, the name of Harpo's second cousin's husband's yoga teacher. Write a query to fix the record.
UPDATE actor
SET first_name = 'HARPO' WHERE first_name = 'GROUCHO' and last_name = 'WILLIAMS';
#Perhaps we were too hasty in changing `GROUCHO` to `HARPO`. It turns out that `GROUCHO` was the correct name after all! In a single query, if the first name of the actor is currently `HARPO`, change it to `GROUCHO`. Otherwise, change the first name to `MUCHO GROUCHO`, as that is exactly what the actor will be with the grievous error. BE CAREFUL NOT TO CHANGE THE FIRST NAME OF EVERY ACTOR TO `MUCHO GROUCHO`, HOWEVER! (Hint: update the record using a unique identifier.)

UPDATE actor
SET first_name = 'GROUCHO' WHERE first_name = 'HARPO' and last_name = 'WILLIAMS';

#You cannot locate the schema of the `address` table. Which query would you use to re-create it?
DESCRIBE address;

#Use `JOIN` to display the first and last names, as well as the address, of each staff member. Use the tables `staff` and `address`:
SELECT first_name, last_name, address
FROM staff
INNER JOIN address  ON staff.address_id = address.address_id;

#Use JOIN to display the total amount rung up by each staff member in August of 2005. Use tables staff and payment. 
SET sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY',''));
SELECT staff.staff_id, staff.first_name, staff.last_name, sum(payment.amount) AS total_payments_August
FROM staff
INNER JOIN payment
ON staff.staff_id = payment.staff_id
WHERE payment_date LIKE ('2005-08-%')
GROUP BY 1; 

#List each film and the number of actors who are listed for that film. Use tables `film_actor` and `film`. Use inner join.
SELECT film.film_id, film.title, count(film_actor.actor_id) AS total_actors
FROM film
INNER JOIN film_actor
ON film.film_id = film_actor.film_id
GROUP BY 1; 


#How many copies of the film `Hunchback Impossible` exist in the inventory system?
SELECT title, count(film_id) AS total_copies
FROM film 
	JOIN inventory USING (film_id) WHERE title = "Hunchback Impossible"
GROUP BY title;


#Using the tables `payment` and `customer` and the `JOIN` command, list the total paid by each customer. List the customers alphabetically by last name:
SELECT customer.last_name, customer.first_name, SUM(payment.amount)
FROM payment 
JOIN customer
ON customer.customer_id = payment.customer_id
GROUP BY payment.customer_id
ORDER BY customer.last_name;

 #The music of Queen and Kris Kristofferson have seen an unlikely resurgence. As an unintended consequence, films starting with the letters `K` and `Q` have also soared in popularity. Use subqueries to display the titles of movies starting with the letters `K` and `Q` whose language is English.
SELECT title
FROM film
WHERE language_id IN (
	SELECT language_id
    FROM LANGUAGE
    WHERE NAME = 'English')
AND (title LIKE "K%") OR (title LIKE "Q%");

#Use subqueries to display all actors who appear in the film `Alone Trip`.
SELECT
first_name,
last_name
FROM actor
WHERE actor_id IN 
(
SELECT actor_id
FROM film_actor
WHERE film_id IN 
(
SELECT film_id
FROM film
WHERE title LIKE 'Alone Trip'
)
);
#You want to run an email marketing campaign in Canada, for which you will need the names and email addresses of all Canadian customers. Use joins to retrieve this information.
SELECT 
first_name, last_name, address
FROM customer
INNER JOIN address ON customer.address_id = address.address_id
INNER JOIN city ON address.city_id = city.city_id
INNER JOIN country ON country.country_id = city.country_id
WHERE country = 'Canada';

#Sales have been lagging among young families, and you wish to target all family movies for a promotion. Identify all movies categorized as famiy films.
Select *
FROM film
WHERE rating = 'G';

#Display the most frequently rented movies in descending order.
SELECT inventory.inventory_id, film.title, count(rental.inventory_id) AS 'Rental Count' FROM rental 
JOIN inventory on inventory.inventory_id = rental.inventory_id
JOIN film on inventory.film_id = film.film_id
GROUP BY inventory.inventory_id
order by `Rental Count` DESC;

#Write a query to display how much business, in dollars, each store brought in.
SELECT inventory.store_id, SUM(payment.amount) FROM payment
JOIN inventory ON payment.rental_id = inventory.inventory_id
GROUP BY store_id;

#Write a query to display for each store its store ID, city, and country.
SELECT store.store_id, address.address, city.city, country.country FROM store
JOIN address ON address.address_id = store.address_id
JOIN city ON city.city_id =  address.city_id
JOIN country ON country.country_id = city.country_id;

#List the top five genres in gross revenue in descending order. (**Hint**: you may need to use the following tables: category, film_category, inventory, payment, and rental.)
SELECT category.NAME, SUM(payment.amount) AS 'Total Revenue' FROM payment
JOIN rental ON rental.rental_id = payment.rental_id
JOIN inventory ON inventory.inventory_id = rental.inventory_id
JOIN film_category ON film_category.film_id = inventory.film_id
JOIN category ON category.category_id = film_category.category_id
GROUP BY category.NAME
ORDER BY SUM(payment.amount) DESC LIMIT 5;

#In your new role as an executive, you would like to have an easy way of viewing the Top five genres by gross revenue. Use the solution from the problem above to create a view. If you haven't solved 7h, you can substitute another query to create a view.
CREATE VIEW top_five_grossing_genres AS

SELECT name, SUM(payment.amount)
FROM category
INNER JOIN film_category on category.category_id = film_category.category_id
INNER JOIN inventory ON inventory.film_id = film_category.film_id
INNER JOIN rental ON rental.inventory_id = inventory.inventory_id
INNER JOIN payment on payment.rental_id = rental.rental_id
GROUP BY name
LIMIT 5;
#How would you display the view that you created in 8a?
SELECT * FROM top_five_grossing_genres;

#You find that you no longer need the view `top_five_genres`. Write a query to delete it.
DROP VIEW top_five_grossing_genres;
