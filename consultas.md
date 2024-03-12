# Punto1 Inserte un registro en la tabla 'película' utilizando valores ficticios, asegurando la integridad referencial con otras tablas

INSERT INTO 
film (title, description, release_year, language_id, original_language_id, rental_duration, rental_rate, length, replacement_cost, rating, special_features)
VALUES 
('Beekeepers', 'la pelicula es d ela venganza d euna colmena', 2022, 1, 1, 5, 2.99, 120, 19.99, 'PG-13', 'Trailers');


![Punto1](https://github.com/AlexGiraldo2/BD2/assets/161048738/031513bf-f196-48ba-8fe6-f88fb5fd3c93)

# Punto 2  ¿Qué películas son más largas que la duración promedio de las películas?

SELECT *
FROM sakila.film
WHERE rental_duration > (SELECT AVG(rental_duration) FROM sakila.film)
limit 10;

# Punto 3 ¿Qué películas se alquilan actualmente en la tienda con store_id = 1?

SELECT f.title
FROM sakila.film f
JOIN sakila.inventory i ON f.film_id = i.film_id
JOIN sakila.rental r ON i.inventory_id = r.inventory_id
WHERE r.return_date IS NULL
AND i.store_id = 1
LIMIT 10;


# Punto 4 De las películas en la tienda con store_id = 1, ¿cuáles se alquilaron por un período más largo que el período de alquiler promedio? '''  '''

# Punto 5 ¿Qué actores forman parte del elenco de 5 o menos películas?

SELECT a.actor_id, a.first_name, a.last_name, 
COUNT(film_actor.film_id) AS num_films
FROM sakila.actor a
JOIN film_actor ON a.actor_id = film_actor.actor_id
GROUP BY a.actor_id 
HAVING num_films <= 5
ORDER BY num_films;


# Punto 6 ¿Qué apellidos no se repiten entre los diferentes actores?

SELECT DISTINCT actor.last_name
FROM sakila.actor;


# Punto 7 Cree una vista con los 3 géneros principales que generan mayores ingresos. Listarlos en orden descendente, considerando el campo 'monto' de la tabla de pagos para el cálculo.

CREATE VIEW top_genero AS
SELECT 
c.name, 
SUM(p.amount) AS total_revenue
FROM sakila.category c
JOIN sakila.film_category fc ON sakila.c.category_id = sakila.fc.category_id
JOIN sakila.film f ON sakila.fc.film_id = sakila.f.film_id
JOIN sakila.inventory i ON sakila.f.film_id = sakila.i.film_id
JOIN sakila.rental r ON i.inventory_id =sakila.r.inventory_id
JOIN sakila.payment p ON r.rental_id = sakila.p.rental_id
GROUP BY sakila.c.name
ORDER BY total_revenue DESC
LIMIT 3;
select * from top_genero;


# Punto 8 Seleccione las dos películas más vistas en cada ciudad.

SELECT 
c.city_id, c.city, film.title AS 'Título de la Película', 
COUNT(rental.rental_id) AS 'Cantidad de Alquileres'
FROM city c
JOIN address ON c.city_id = address.city_id
JOIN store ON address.address_id = store.address_id
JOIN inventory ON store.store_id = inventory.store_id
JOIN rental ON inventory.inventory_id = rental.inventory_id
JOIN film ON inventory.film_id = film.film_id
GROUP BY c.city_id, film.title
ORDER BY c.city_id, COUNT(rental.rental_id) DESC
LIMIT 2;

# Punto 9 Seleccione el nombre, apellido y correo electrónico de todos los clientes de Estados Unidos que no hayan alquilado ninguna película en los últimos tres meses.

SELECT c.customer_id, c.first_name, c.last_name
FROM customer c
JOIN address a ON c.address_id = a.address_id
JOIN city ci ON a.city_id = ci.city_id
JOIN country co ON ci.country_id = co.country_id
LEFT JOIN rental r ON c.customer_id = r.customer_id
WHERE co.country = 'United States'
AND (r.rental_date IS NULL OR r.rental_date < DATE_SUB(NOW(), INTERVAL 3 MONTH))
GROUP BY c.customer_id, c.first_name, c.last_name, c.email
HAVING COUNT(r.rental_id) = 0;

