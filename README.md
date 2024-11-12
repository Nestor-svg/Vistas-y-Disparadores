# Vistas-y-Disparadores
# Vistas-y-Disparadores

## Consultas
### Obtenga las ventas totales por categoría de películas ordenadas descendentemente.

```
alquilerdvd=> SELECT category_id, COUNT(*) AS Total_peliculas
FROM film_category
GROUP BY category_id
ORDER BY Total_peliculas DESC;
 category_id | total_peliculas 
-------------+-----------------
          15 |              74
           9 |              73
           8 |              69
           6 |              68
           2 |              66
           1 |              64
          13 |              63
           7 |              62
          14 |              61
          10 |              61
           3 |              60
           5 |              58
           4 |              57
          16 |              57
          11 |              56
          12 |              51
(16 rows)
```

### Obtenga las ventas totales por tienda, donde se refleje la ciudad, el país (concatenar la ciudad y el país empleando como separador la “,”), y el encargado. Pudiera emplear GROUP BY, ORDER BY
```
alquilerdvd=> SELECT CONCAT(c.city, ', ', co.country) AS ciudad_pais, s.store_id, CONCAT(st.first_name, ' ', st.last_name) AS encargado, SUM(p.amount) AS ventas_totales
FROM store s
JOIN staff st ON s.manager_staff_id = st.staff_id
JOIN address a ON s.address_id = a.address_id
JOIN city c ON a.city_id = c.city_id
JOIN country co ON c.country_id = co.country_id
JOIN payment p ON p.staff_id = st.staff_id
GROUP BY s.store_id, c.city, co.country, st.staff_id
ORDER BY ventas_totales DESC;
     ciudad_pais      | store_id |  encargado   | ventas_totales 
----------------------+----------+--------------+----------------
 Woodridge, Australia |        2 | Jon Stephens |       31059.92
 Lethbridge, Canada   |        1 | Mike Hillyer |       30252.12
(2 rows)

```

### Obtenga una lista de películas, donde se reflejen el identificador, el título, descripción, categoría, el precio, la duración de la película, clasificación, nombre y apellidos de los actores (puede realizar una concatenación de ambos). Pudiera emplear GROUP BY
#### Seleccionamos las 3 primeras películas para simplificar la salida.
```
alquilerdvd=> SELECT f.film_id, f.title, f.description, fc.category_id, f.rental_rate, f.length, f.rating, STRING_AGG(CONCAT(a.first_name, ' ', a.last_name), ', ') AS actors
FROM film f
JOIN film_category fc ON f.film_id = fc.film_id
JOIN film_actor fa ON f.film_id = fa.film_id
JOIN actor a ON fa.actor_id = a.actor_id
GROUP BY f.film_id, f.title, f.description, fc.category_id, f.rental_rate, f.length, f.rating
-[ RECORD 1 ]-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
film_id     | 1
title       | Academy Dinosaur
description | A Epic Drama of a Feminist And a Mad Scientist who must Battle a Teacher in The Canadian Rockies
category_id | 6
rental_rate | 0.99
length      | 86
rating      | PG
actors      | Penelope Guiness, Christian Gable, Lucille Tracy, Sandra Peck, Johnny Cage, Mena Temple, Warren Nolte, Oprah Kilmer, Rock Dukakis, Mary Keitel
-[ RECORD 2 ]-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
film_id     | 2
title       | Ace Goldfinger
description | A Astounding Epistle of a Database Administrator And a Explorer who must Find a Car in Ancient China
category_id | 11
rental_rate | 4.99
length      | 48
rating      | G
actors      | Bob Fawcett, Minnie Zellweger, Sean Guiness, Chris Depp
-[ RECORD 3 ]-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
film_id     | 3
title       | Adaptation Holes
description | A Astounding Reflection of a Lumberjack And a Car who must Sink a Lumberjack in A Baloon Factory
category_id | 6
rental_rate | 2.99
length      | 50
rating      | NC-17
actors      | Nick Wahlberg, Bob Fawcett, Cameron Streep, Ray Johansson, Julianne Dench

```

### Obtenga la información de los actores, donde se incluya sus nombres y apellidos, las categorías y sus películas. Los actores deben de estar agrupados y, las categorías y las películas deben estar concatenados por “:” 
#### Seleccionamos los 3 primeros actores para simplificar la salida.
```
alquilerdvd=> SELECT 
    a.actor_id,
    CONCAT(a.first_name, ' ', a.last_name) AS actor_name,
    STRING_AGG(DISTINCT c.name, ' : ') AS categories,
    STRING_AGG(DISTINCT f.title, ' : ') AS films
FROM actor a
JOIN film_actor fa ON a.actor_id = fa.actor_id
JOIN film f ON fa.film_id = f.film_id
JOIN film_category fc ON f.film_id = fc.film_id
JOIN category c ON fc.category_id = c.category_id
GROUP BY a.actor_id, a.first_name, a.last_name
ORDER BY a.actor_id;
-[ RECORD 1 ]------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
actor_id   | 1
actor_name | Penelope Guiness
categories | Animation : Children : Classics : Comedy : Documentary : Family : Foreign : Games : Horror : Music : New : Sci-Fi : Sports
films      | Academy Dinosaur : Anaconda Confessions : Angels Life : Bulworth Commandments : Cheaper Clyde : Color Philadelphia : Elephant Trojan : Gleaming Jawbreaker : Human Graffiti : King Evolution : Lady Stage : Language Cowboy : Mulholland Beast : Oklahoma Jumanji : Rules Human : Splash Gump : Vertigo Northwest : Westward Seabiscuit : Wizard Coldblooded
-[ RECORD 2 ]------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
actor_id   | 2
actor_name | Nick Wahlberg
categories | Action : Animation : Children : Classics : Comedy : Documentary : Drama : Family : Foreign : Games : Music : New : Sci-Fi : Travel
films      | Adaptation Holes : Apache Divine : Baby Hall : Bull Shawshank : Chainsaw Uptown : Chisum Behavior : Destiny Saturday : Dracula Crystal : Fight Jawbreaker : Flash Wars : Gilbert Pelican : Goodfellas Salute : Happiness United : Indian Love : Jekyll Frogmen : Jersey Sassy : Liaisons Sweet : Lucky Flying : Maguire Apache : Mallrats United : Mask Peach : Roof Champion : Rushmore Mermaid : Smile Earring : Wardrobe Phantom
-[ RECORD 3 ]------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
actor_id   | 3
actor_name | Ed Chase
categories | Action : Classics : Documentary : Drama : Foreign : Music : New : Sci-Fi : Sports : Travel
films      | Alone Trip : Army Flintstones : Artist Coldblooded : Boondock Ballroom : Caddyshack Jedi : Cowboy Doom : Eve Resurrection : Forrest Sons : French Holiday : Frost Head : Halloween Nuts : Hunter Alter : Image Princess : Jeepers Wedding : Luck Opus : Necklace Outbreak : Platoon Instinct : Spice Sorority : Wedding Apollo : Weekend Personal : Whale Bikini : Young Language

```
