# Zythologue - Base de Données pour Amateurs de Bière

Ce projet a pour but de configurer une base de données PostgreSql de Zythologue avec Docker, Docker-compose et DBeaver.

## Contexte du Projet

Tu as une passion dévorante pour la bière artisanale et souhaites découvrir et partager ce monde au-delà de la dégustation : explorer l'histoire derrière chaque brasserie, les ingrédients spécifiques, et les techniques de brassage.

En tant que développeur, tu envisages de créer une application web/mobile pour organiser et partager cette passion. Avant de te lancer dans le développement, une base de données bien structurée est cruciale.

À toi de jouer 🙂

## Conception 

Vous trouverez tous les documents lié à la conception dans le projet dans le dossier "conception" à savoir :

- Un dictionnaire de données 
- les règles de gestion
- un MCD
- un MLD
- un MPD

## Pré-requis

Pour installer ce projet en local vous aurez besoin de :

- DOCKER
- Docker Compose
- Dbeaver

## Lancement Local

Clonez le projet :

```bash
  git clone https://github.com
```

Lancez le container Docker grâce à cette commande dans votre terminal :

```bash
  docker-compose up -d
```

Ainsi le docker-compose lancera automatiquement tout son contenu, à savoir :

- il téléchargera l'image nécessaire via docker-hub
- vous pourrez changer le nom du conteneur si vous le souhaitez 
- on retrouve les variables d'environnement qu'on peut modifier en fonction des besoins
- le port utilisé par docker localement et par le conteneur 
- le volume utilisé
- le script d'initialisation qui sera automatiquement chargé

```
services:

  db:
    image: postgres
    container_name: postgres_service
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin
      POSTGRES_DB: Zytho
    ports:
      - 5433:5432
    volumes:
    - db-data:/var/lib/postgresql/data #Volume pour que même si je stoppe mon conteneur, mes données ne soient pas perdues
    - ./init.sql:/docker-entrypoint-initdb.d/init.sql #même si je supprime tout, docker vient initiliser la base de donnée par rapport au fichier 

volumes:
  db-data:
```

## Se connecter à DBeaver

La prochaine étape est de connecter DBeaver 

- Ouvrir l'application DBeaver
- Créer une nouvelle connexion en choisissant bien PostgreSql comme type. 
- Rentrer les configurations nécessaires venant de votre docker-compose (Host, Port, Database, Username, Password)

Une fois que la configuration est prête, tout se chargera automatiquement car souvenez-vous nous avons notre script qui est dans "init.sql", et notre volume nous permet de retrouver toutes les informations. 

Ainsi vous retrouverez toutes vos tables et on a même fournis de la donnée. 

## Requêtes SQL

Voici une liste de requêtes SQL permettant d'interagir avec la base de données Zythologue, chaque requête étant décrite brièvement pour expliquer son fonctionnement.

### 1. Lister les bières par taux d'alcool, de la plus légère à la plus forte
```sql
SELECT * FROM beer
ORDER BY abv ASC;
```

### 2. Afficher le nombre de bières par catégorie.
```sql
select c.id, c.name as category_name, COUNT(bc.id) as beer_count
from category c
left join beer_category bc on c.id = bc.id_category
group by c.id
order by beer_count desc;
```

### 3. Trouver toutes les bières d'une brasserie donnée.
```sql
select be.id, be.name as beer_name
from brewery b
inner join beer_brewery bbr on b.id = bbr.id_brewery
inner join beer be on bbr.id_beer = be.id
where b.name = 'Ancient Brewers';
```

### 4. Lister les utilisateurs et le nombre de bières qu'ils ont ajoutées à leurs favoris.
```sql
select u.firstname as user_name, COUNT(bu.id_beer) as beer_favorites_count
from users u
left join beer_users bu on u.id = bu.id_users
group by u.firstname
order by beer_favorites_count desc;
```

### 5. Ajouter une nouvelle bière à la base de données.
```sql
insert into beer (name, description, abv, acidity, bitterness, sweetness, container_type, beer_volume, organic_beer)
values 
    ('Session IPA', 
     'A nice and smooth IPA beer, a little bit fruity', 
     5.5, 
     2, 
     3, 
     4, 
     'Bottle', 
     250, 
     true);
```

### 6. Afficher les bières et leurs brasseries, ordonnées par pays de la brasserie.
```sql
select 
    b.name as beer_name,
    br.name as brewery_name,
    br.country as brewery_country
from 
    beer b
left join  
    beer_brewery bbr on b.id = bbr.id_beer
left join
	brewery br on bbr.id_beer = br.id
order by  
    br.country asc;
```

### 7. Lister les bières avec leurs ingrédients.
```sql
select 
    b.name as beer_name,
    i.name as ingredient_name
from 
    beer b
left join  
    beer_ingredient bi on b.id = bi.id_beer
left join
	ingredient i on bi.id_beer = i.id
;
```

### 8. Afficher les brasseries et le nombre de bières qu'elles produisent, pour celles ayant plus de 5 bières.
```sql
select 
    br.name as brewery_name,
    count(bb.id) as beer_count
from 
    brewery br
left join  
    beer_brewery bb on br.id = bb.id_brewery 
left join
	beer b on bb.id_beer = b.id
group by 
    br.name
having 
    count(bb.id) > 5
;
```

### 9. Afficher les brasseries et le nombre de bières qu'elles produisent, pour celles ayant plus de 5 bières.
```sql
select 
    b.name as beer_name
from 
    beer b
left join  
    beer_users bu on b.id = bu.id_beer
where 
	bu.id_users is null
;
```

### 10. Trouver les bières favorites communes entre deux utilisateurs.
```sql
select 
    b.name as beer_name
from 
    beer b
left join  
    beer_users bu1 on b.id = bu1.id_beer
left join 
	beer_users bu2 on b.id = bu2.id_beer
where 
	bu1.id_users = 1 
    AND bu2.id_users = 2
;
```

### 11. Afficher les brasseries dont les bières ont une moyenne de notes supérieure à une certaine valeur.
```sql
select 
    br.name as brewery_name,
    avg(r.rate) as average_rating
from 
    brewery br
left join  
    beer_brewery bb on br.id = bb.id_brewery 
left join 
	beer b on bb.id_beer = b.id
left join 
	review r on b.id = r.id_beer 
group by 
	br.name
having
	avg(r.rate) > 3
;
```

### 12. Mettre à jour les informations d'une brasserie.
```sql
update brewery
set 
    id_photo = 15,
    name = 'Dark Stout Brewing',
    description = 'new_description'
where 
    id = 5
;
```

### 13. Supprimer les photos d'une bière en particulier.
```sql
delete from photo
where photo.id_beer = 10;
```

Cette dernière requête n'est pas faisable car la photo est référencée dans la table brewery car il y a une contrainte de FK (foreign key)