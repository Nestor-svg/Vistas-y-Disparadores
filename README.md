# Vistas-y-Disparadores

## Realice la restauración de la base de datos alquilerdvd.tar. Observe que la base de datos no tiene un formato SQL como el empleado en actividades anteriores.
```
createdb alquilerdvd
pg_restore -d alquilerdvd -U usuario -h localhost -p 5432 AlquilerPractica.tar
```

## Consultas
### 1. Obtenga las ventas totales por categoría de películas ordenadas descendentemente.

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

### 2. Obtenga las ventas totales por tienda, donde se refleje la ciudad, el país (concatenar la ciudad y el país empleando como separador la “,”), y el encargado. Pudiera emplear GROUP BY, ORDER BY

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

### 3. Obtenga una lista de películas, donde se reflejen el identificador, el título, descripción, categoría, el precio, la duración de la película, clasificación, nombre y apellidos de los actores (puede realizar una concatenación de ambos). Pudiera emplear GROUP BY
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

### 4. Obtenga la información de los actores, donde se incluya sus nombres y apellidos, las categorías y sus películas. Los actores deben de estar agrupados y, las categorías y las películas deben estar concatenados por “:” 
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

## Vistas
### 1. Vista primera consulta

```
alquilerdvd=> CREATE VIEW view_total_peliculas_por_categoria AS
alquilerdvd-> SELECT category_id, COUNT(*) AS total_peliculas
alquilerdvd-> from film_category
alquilerdvd-> GROUP BY category_id
alquilerdvd-> ORDER BY total_peliculas DESC;
CREATE VIEW

alquilerdvd=> SELECT * FROM view_total_peliculas_por_categoria;
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

### 2. Vista segunda consulta

```
alquilerdvd=> CREATE VIEW view_ventas_totales_por_tienda AS
alquilerdvd-> SELECT CONCAT(c.city, ', ', co.country) AS ciudad_pais, s.store_id, CONCAT(st.first_name, ' ', st.last_name) AS encargado, SUM(p.amount) AS ventas_totales
alquilerdvd-> FROM store s
alquilerdvd-> JOIN staff st ON s.manager_staff_id = st.staff_id
alquilerdvd-> JOIN address a ON s.address_id = a.address_id
alquilerdvd-> JOIN city c ON a.city_id = c.city_id
alquilerdvd-> JOIN country co ON c.country_id = co.country_id
alquilerdvd-> JOIN payment p ON p.staff_id = st.staff_id
alquilerdvd-> GROUP BY s.store_id, c.city, co.country, st.staff_id
alquilerdvd-> ORDER BY ventas_totales DESC;
CREATE VIEW

alquilerdvd=> SELECT * FROM view_ventas_totales_por_tienda;
     ciudad_pais      | store_id |  encargado   | ventas_totales 
----------------------+----------+--------------+----------------
 Woodridge, Australia |        2 | Jon Stephens |       31059.92
 Lethbridge, Canada   |        1 | Mike Hillyer |       30252.12
(2 rows)
```

### 3. Vista tercera consulta
#### Seleccionamos las 3 primeras películas para simplificar la salida.

```
alquilerdvd-> SELECT f.film_id, f.title, f.description, fc.category_id, f.rental_rate, f.length, f.rating, STRING_AGG(CONCAT(a.first_name, ' ', a.last_name), ', ') AS actors
alquilerdvd-> FROM film f
alquilerdvd-> JOIN film_category fc ON f.film_id = fc.film_id
alquilerdvd-> JOIN film_actor fa ON f.film_id = fa.film_id
alquilerdvd-> JOIN actor a ON fa.actor_id = a.actor_id
alquilerdvd-> GROUP BY f.film_id, f.title, f.description, fc.category_id, f.rental_rate, f.length, f.rating;
CREATE VIEW

alquilerdvd=> SELECT * FROM view_lista_peliculas;
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

### 4. Vista cuarta consulta
#### Seleccionamos los 3 primeros actores para simplificar la salida.

```
alquilerdvd-> FROM actor a
alquilerdvd-> JOIN film_actor fa ON a.actor_id = fa.actor_id
alquilerdvd-> JOIN film f ON fa.film_id = f.film_id
alquilerdvd-> JOIN film_category fc ON f.film_id = fc.film_id
alquilerdvd-> JOIN category c ON fc.category_id = c.category_id
alquilerdvd-> GROUP BY a.actor_id, a.first_name, a.last_name
alquilerdvd-> ORDER BY a.actor_id;
CREATE VIEW

alquilerdvd=> SELECT * FROM view_informacion_actores;
[ RECORD 1 ]------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
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

## Restricciones CHECK
### 1. Restricción CHECK para evitar nombres vacíos o solo espacios en la tabla actor.

```
alquilerdvd=> ALTER TABLE actor 
alquilerdvd-> ADD CONSTRAINT check_first_name_not_empty CHECK (TRIM(first_name) <> '');
ALTER TABLE
alquilerdvd=> ALTER TABLE actor 
ADD CONSTRAINT check_last_name_not_empty CHECK (TRIM(last_name) <> '');
ALTER TABLE
```

### 2. Restricción CHECK para garantizar que city_id sea válido en la tabla address.

```
alquilerdvd=> ALTER TABLE address
alquilerdvd-> ADD CONSTRAINT fk_city_id FOREIGN KEY (city_id) REFERENCES city(city_id);
ALTER TABLE
```

### 3. Restricción CHECK para garantizar unicidad en la tabla country.

```
alquilerdvd=> ALTER TABLE country
alquilerdvd-> ADD CONSTRAINT unique_country_name UNIQUE (country);
ALTER TABLE
```

### 4. Restricción CHECK para validar el formato de release_year en la tabla film.

```
alquilerdvd=> ALTER TABLE film
alquilerdvd-> ADD CONSTRAINT check_release_year_valid CHECK (release_year BETWEEN 1900 AND EXTRACT(YEAR FROM CURRENT_DATE));
ALTER TABLE
```

### 5. Restricción CHECK para garantizar que amount no sea negativo en la tabla payment.

```
alquilerdvd=> ALTER TABLE payment
ADD CONSTRAINT check_amount_positive CHECK (amount >= 0);
ALTER TABLE
```

### 6. Restricción CHECK para garantizar que active solo contenga valores booleanos en la tabla staff.

```
alquilerdvd=> ALTER TABLE staff
alquilerdvd-> ADD CONSTRAINT check_active_boolean CHECK (active IN (true, false));
ALTER TABLE
```
### 7. Restricción CHECK para garantizar que last_update no sea futura en la tabla store.

```
alquilerdvd=> ALTER TABLE store
alquilerdvd-> ADD CONSTRAINT check_last_update_past CHECK (last_update <= CURRENT_TIMESTAMP);
ALTER TABLE
```

## Explicación de la sentencia

```
last_updated BEFORE UPDATE ON customer 
FOR EACH ROW EXECUTE PROCEDURE last_updated();
```

Explicación:
- Disparador (Trigger): Es un mecanismo en PostgreSQL que ejecuta una función automáticamente al ocurrir un evento en una tabla (como INSERT, UPDATE o DELETE).
- En este caso:
    - El disparador se activa antes de que se actualice un registro en la tabla customer.
    - Llama a la función last_updated(), que probablemente actualiza el campo last_updated con la fecha y hora actuales, para registrar cuándo se modificó la fila.

## Construya un disparador que guarde en una nueva tabla creada por usted la fecha de cuando se insertó un nuevo registro en la tabla film. 

```
alquilerdvd=> SELECT * FROM film;
alquilerdvd=> CREATE TABLE film_insert_log (
alquilerdvd(> film_id INT,
alquilerdvd(> insert_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP);
CREATE TABLE

alquilerdvd=> CREATE OR REPLACE FUNCTION log_film_insert() 
alquilerdvd-> RETURNS TRIGGER AS $$
alquilerdvd$> BEGIN
alquilerdvd$>     INSERT INTO film_insert_log(film_id, insert_date)
alquilerdvd$>     VALUES (NEW.film_id, CURRENT_TIMESTAMP);
alquilerdvd$>     RETURN NEW;
alquilerdvd$> END;
alquilerdvd$> $$ LANGUAGE plpgsql;
CREATE FUNCTION

alquilerdvd=> CREATE TRIGGER trigger_film_insert
alquilerdvd-> AFTER INSERT ON film
alquilerdvd-> FOR EACH ROW
alquilerdvd-> EXECUTE FUNCTION log_film_insert();
CREATE TRIGGER
```

## Construya un disparador que guarde en una nueva tabla creada por usted la fecha de cuando se eliminó un registro en la tabla film y el identificador del film. 

```
alquilerdvd=> CREATE TABLE film_delee_log (
alquilerdvd(> film_id INT,
alquilerdvd(> delete_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP);
CREATE TABLE

alquilerdvd=> CREATE OR REPLACE FUNCTION log_film_delete() 
alquilerdvd-> RETURNS TRIGGER AS $$
alquilerdvd$> BEGIN
alquilerdvd$>     INSERT INTO film_delete_log(film_id, delete_date)
alquilerdvd$>     VALUES (OLD.film_id, CURRENT_TIMESTAMP);
alquilerdvd$>     RETURN OLD;
alquilerdvd$> END;
alquilerdvd$> $$ LANGUAGE plpgsql;
CREATE FUNCTION

alquilerdvd=> CREATE TRIGGER trigger_film_delete
alquilerdvd-> AFTER DELETE ON film
alquilerdvd-> FOR EACH ROW
alquilerdvd-> EXECUTE FUNCTION log_film_delete();
CREATE TRIGGER
```

## Significado y relevancia de las secuencias

- Las secuencias en PostgreSQL son objetos que generan números de manera secuencial. Son esenciales para gestionar valores únicos y auto-incrementales, como identificadores en las tablas, garantizando que no se repitan.

- Relevancia de las secuencias:
    - Generación automática de identificadores: Son muy útiles para crear valores únicos para columnas de tipo serial o bigserial sin necesidad de intervención manual.
    - Integridad de datos: Aseguran que los identificadores sean únicos y consecutivos, evitando colisiones o duplicados.
    - Optimización de rendimiento: Mejoran la eficiencia al gestionar automáticamente la asignación de valores en operaciones de inserción masiva, eliminando la sobrecarga de manejar los identificadores por separado.