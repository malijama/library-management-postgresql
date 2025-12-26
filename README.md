# Système de Gestion de Bibliothèque - PostgreSQL

## Description

Un système complet de gestion de bibliothèque construit avec PostgreSQL. Ce projet démontre la conception de bases de données relationnelles, les requêtes SQL et la manipulation de données pour gérer les livres, auteurs, membres et enregistrements d'emprunts.

**Note importante** : Les données (noms de livres, auteurs, membres) sont en **anglais**, mais toute la documentation est en **français** pour faciliter la compréhension.

## Fonctionnalités

- **Schéma de base de données complet** : 4 tables interconnectées avec relations de clés étrangères
- **Jeu de données riche** : Pré-rempli avec 18 auteurs célèbres, 21 livres classiques, 18 membres et 30+ enregistrements d'emprunts
- **19 requêtes SQL** : De basiques à avancées couvrant des scénarios réels de bibliothèque
- **Intégrité des données** : Contraintes et relations appropriées pour maintenir la cohérence

## Structure de la Base de Données

### Tables

1. **Authors** (Auteurs)
   - AuthorID (Clé primaire)
   - FirstName (Prénom)
   - LastName (Nom)

2. **Books** (Livres)
   - BookID (Clé primaire)
   - Title (Titre)
   - AuthorID (Clé étrangère → Authors)
   - Genre
   - PublishedYear (Année de publication)
   - Quantity (Quantité)

3. **Members** (Membres)
   - MemberID (Clé primaire)
   - FirstName (Prénom)
   - LastName (Nom)
   - MembershipDate (Date d'adhésion)

4. **BorrowingRecords** (Enregistrements d'emprunts)
   - RecordID (Clé primaire)
   - MemberID (Clé étrangère → Members)
   - BookID (Clé étrangère → Books)
   - BorrowDate (Date d'emprunt)
   - ReturnDate (Date de retour)

## Installation

### Prérequis

- PostgreSQL 12 ou supérieur
- Outil en ligne de commande psql ou tout client PostgreSQL (pgAdmin, DBeaver, etc.)

### Configuration

1. Cloner le dépôt :
```bash
git clone https://github.com/malijama/library-management-postgresql.git
cd library-management-postgresql
```

2. Créer une base de données :
```sql
CREATE DATABASE library_db;
```

3. Exécuter le script SQL :
```bash
psql -d library_db -f library_management_system_complete.sql
```

## Données Exemples

### Auteurs Inclus (en anglais)
- J.K. Rowling, George Orwell, J.R.R. Tolkien
- Isaac Asimov, Agatha Christie, Mark Twain
- F. Scott Fitzgerald, Jane Austen
- Et 10 autres auteurs classiques

### Livres Inclus (en anglais)
- Harry Potter series, 1984, The Hobbit
- Foundation, Murder on the Orient Express
- The Great Gatsby, Pride and Prejudice
- Et 14 autres titres classiques

## Liste des 19 Requêtes SQL

### Requêtes Basiques

**Question 1: Find all books borrowed by John**
*Trouver tous les livres empruntés par John*

```sql
SELECT firstname || ' ' || lastname AS fullname, title
FROM books b
JOIN borrowingrecords b2 ON b.bookid = b2.bookid
JOIN members m ON b2.memberid = m.memberid
WHERE m.firstname LIKE 'John%';
```

**Question 2: List all available books (not borrowed)**
*Lister tous les livres disponibles (non empruntés)*

```sql
SELECT title
FROM books b
WHERE b.bookid NOT IN (
    SELECT b2.bookid FROM borrowingrecords b2
);
```

**Question 3: Count the total number of books by each author**
*Compter le nombre total de livres par auteur*

```sql
SELECT COUNT(title) AS number_of_books,
       a.firstname || ' ' || a.lastname AS authors_name
FROM books b
JOIN authors a ON b.authorid = a.authorid
GROUP BY a.firstname, a.lastname
ORDER BY number_of_books;
```

**Question 4: List all books by a genre**
*Lister tous les livres par genre*

```sql
SELECT DISTINCT genre, title
FROM books
ORDER BY genre;
```

**Question 5: List members who haven't borrowed any books**
*Lister les membres qui n'ont emprunté aucun livre*

```sql
SELECT m.firstname || ' ' || m.lastname AS members_name
FROM members m
WHERE m.memberid NOT IN (
    SELECT b.memberid FROM borrowingrecords b
);
```

**Question 6: Find the total of books available in the library across all title**
*Trouver le nombre total de livres disponibles dans la bibliothèque*

```sql
SELECT SUM(quantity) total_books
FROM books;
```

**Question 7: List all authors who have written more than one book in the library**
*Lister tous les auteurs ayant écrit plus d'un livre dans la bibliothèque*

```sql
SELECT a.firstname || ' ' || a.lastname AS authors_name,
       COUNT(*) AS number_of_books
FROM authors a
JOIN books b ON a.authorid = b.authorid
GROUP BY authors_name
HAVING COUNT(*) > 1
ORDER BY number_of_books DESC;
```

### Requêtes Intermédiaires

**Question 8: Find the average number of books borrowed by each member**
*Trouver le nombre moyen de livres empruntés par membre*

```sql
SELECT ROUND(AVG(books_count), 2) AS average
FROM (
    SELECT m.firstname || ' ' || m.lastname AS members_name,
           COUNT(*) AS books_count
    FROM members m
    JOIN borrowingrecords b ON m.memberid = b.memberid
    GROUP BY members_name
);
```

**Question 9: List all books that have never been borrowed**
*Lister tous les livres qui n'ont jamais été empruntés*

```sql
SELECT title
FROM books b
LEFT JOIN borrowingrecords b2 ON b.bookid = b2.bookid
WHERE b2.bookid IS NULL;
```

**Question 10: Find the books borrowed within a specific date range**
*Trouver les livres empruntés dans une période spécifique (ex: entre '2023-08-01' et '2023-08-15')*

```sql
SELECT title
FROM books b
JOIN borrowingrecords b2 ON b.bookid = b2.bookid
WHERE b2.borrowdate BETWEEN '2023-08-01' AND '2023-08-15'
ORDER BY title;
```

**Question 11: Determine the most popular genre in the library based on the number of books borrowed**
*Déterminer le genre le plus populaire basé sur le nombre d'emprunts*

```sql
SELECT b.genre,
       COUNT(*) AS number_books_borrowed
FROM books b
JOIN borrowingrecords b2 ON b.bookid = b2.bookid
GROUP BY b.genre
ORDER BY number_books_borrowed DESC;
```

**Question 12: Find overdue books (books that have not been returned in 30 days)**
*Trouver les livres en retard (non retournés après 30 jours)*

```sql
SELECT m.firstname || ' ' || m.lastname AS fullname,
       b.title,
       b2.borrowdate,
       CURRENT_DATE - b2.borrowdate AS date_overdue
FROM books b
JOIN borrowingrecords b2 ON b.bookid = b2.bookid
JOIN members m ON b2.memberid = m.memberid
WHERE CURRENT_DATE - b2.borrowdate > 30;
```

**Question 13: Find all books currently borrowed by members**
*Trouver tous les livres actuellement empruntés*

```sql
SELECT b.title,
       m.firstname || ' ' || m.lastname AS fullname
FROM books b
JOIN borrowingrecords b2 ON b.bookid = b2.bookid
JOIN members m ON b2.memberid = m.memberid
WHERE b2.returndate IS NULL;
```

**Question 14: Identify members who have borrowed more than one book**
*Identifier les membres ayant emprunté plus d'un livre*

```sql
SELECT m.firstname || ' ' || m.lastname AS fullname,
       COUNT(b2.bookid) AS numbers_of_books
FROM books b
JOIN borrowingrecords b2 ON b.bookid = b2.bookid
JOIN members m ON b2.memberid = m.memberid
GROUP BY fullname
HAVING COUNT(b2.bookid) > 1;
```

### Requêtes Avancées

**Question 15a: List all overdue books along with the days overdue**
*Lister tous les livres en retard avec le nombre de jours de retard*

```sql
SELECT m.firstname || ' ' || m.lastname AS fullname,
       b.title,
       CURRENT_DATE - b2.borrowdate AS days_overdue
FROM books b
JOIN borrowingrecords b2 ON b.bookid = b2.bookid
JOIN members m ON b2.memberid = m.memberid
WHERE CURRENT_DATE - b2.borrowdate > 30
  AND b2.returndate IS NULL;
```

**Question 15b: List all books and their corresponding authors that have been borrowed at least twice**
*Lister tous les livres et leurs auteurs empruntés au moins deux fois*

```sql
SELECT b.title,
       a.firstname || ' ' || a.lastname AS authors_name,
       COUNT(b2.bookid) AS numbers_of_booking
FROM books b
JOIN borrowingrecords b2 ON b.bookid = b2.bookid
JOIN authors a ON b.authorid = a.authorid
GROUP BY authors_name, b.title
HAVING COUNT(b2.bookid) > 1;
```

**Question 16: Find the difference in the number of books borrowed between two specific members**
*Trouver la différence du nombre de livres empruntés entre deux membres spécifiques (ex: 'John Doe' et 'Alice Johnson')*

```sql
-- Méthode CROSS JOIN
SELECT ABS(john.no_books - alice.no_books) AS books_difference
FROM
    (SELECT COUNT(b2.bookid) AS no_books
     FROM borrowingrecords b2
     JOIN members m ON b2.memberid = m.memberid
     WHERE m.firstname || ' ' || m.lastname LIKE 'John%') AS john
CROSS JOIN
    (SELECT COUNT(b2.bookid) AS no_books
     FROM borrowingrecords b2
     JOIN members m ON b2.memberid = m.memberid
     WHERE m.firstname || ' ' || m.lastname LIKE 'Alice%') AS alice;
```

**Question 17: Identify all members who borrowed books by a specific author**
*Identifier tous les membres ayant emprunté des livres d'un auteur spécifique (ex: 'George Orwell')*

```sql
SELECT DISTINCT m.firstname || ' ' || m.lastname AS member_name,
                a.firstname || ' ' || a.lastname AS author_name,
                b.title
FROM members m
JOIN borrowingrecords b2 ON b2.memberid = m.memberid
JOIN books b ON b2.bookid = b.bookid
JOIN authors a ON b.authorid = a.authorid
WHERE a.firstname || ' ' || a.lastname LIKE 'Geo%';
```

**Question 18: Find the most borrowed book (using window functions)**
*Trouver le livre le plus emprunté (avec fonctions fenêtre)*

```sql
SELECT title, most_borrowed
FROM (
    SELECT b.title,
           COUNT(*) AS most_borrowed,
           DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) AS ranks
    FROM books b
    JOIN borrowingrecords b2 ON b.bookid = b2.bookid
    GROUP BY b.title
) ranked
WHERE ranks = 1;
```

**Question 19: Identify the member who has borrowed the most books**
*Identifier le membre ayant emprunté le plus de livres*

```sql
SELECT member_name, number_books
FROM (
    SELECT firstname || ' ' || lastname AS member_name,
           COUNT(b2.memberid) AS number_books,
           DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) AS ranks
    FROM members m
    JOIN borrowingrecords b2 ON m.memberid = b2.memberid
    GROUP BY firstname || ' ' || lastname
) ranked
WHERE ranks = 1;
```

## Techniques SQL Démontrées

- **Jointures** : INNER JOIN, LEFT JOIN, CROSS JOIN
- **Sous-requêtes** : IN, NOT IN, sous-requêtes imbriquées
- **Agrégation** : COUNT, SUM, AVG avec GROUP BY
- **Opérations sur chaînes** : Concaténation avec ||
- **Fonctions de date** : CURRENT_DATE, arithmétique de dates
- **Fonctions fenêtre** : DENSE_RANK() OVER()
- **Filtrage** : Clauses WHERE, HAVING
- **Distinct** : Suppression des doublons
- **Tri** : ORDER BY avec ASC/DESC

## Cas d'Usage

Ce système de base de données peut gérer :
- Gestion de l'inventaire des livres
- Inscription et suivi des membres
- Opérations d'emprunt et de retour
- Identification des livres en retard
- Analyse des genres populaires
- Suivi de l'activité des membres
- Statistiques sur les auteurs et les livres

## Auteur

**Mohamed** ([@malijama](https://github.com/malijama))

## Licence

Ce projet est open source et disponible à des fins éducatives.

---

**Parfait pour** : Apprentissage SQL, Pratique de conception de bases de données, Projets portfolio, Préparation aux entretiens SQL
