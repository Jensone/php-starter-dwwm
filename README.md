# Guide d’apprentissage PHP 

*Sprint de 2 semaines – [Méthode Agiliteach](https://agiliteach.org)*

L’objectif est de vous faire acquérir **pas à pas** des compétences concrètes en PHP, à travers un projet web progressif et centré sur la pratique. Vous apprendrez à : Dockeriser votre environnement de développement (plus besoin de WAMP/XAMPP), manipuler les bases de PHP (avec des fichiers JSON plutôt qu’une base de données), construire un routeur web « maison » en PHP, et intégrer des API externes (GPT d’[OpenAI](https://openai.com) et [ElevenLabs](https://elevenlabs.io)). Le tout sera découpé en modules progressifs comprenant des explications accessibles et des exercices pratiques guidés, dans un esprit d’**apprentissage agile, itératif et orienté compétences**.

*(Remarque : Un livre complet est mis à votre disposition sur le drive. Consultez-le pour approfondir les concepts du langage PHP.) *

## Module 1 : Mise en place de l’environnement avec Docker

Pour développer en PHP sans installer manuellement Apache/PHP (WAMP/XAMPP/MAMP), nous allons utiliser **Docker**. Docker permet de créer des *conteneurs* isolés contenant tout le nécessaire pour faire tourner une application (serveur web, interpréteur PHP, etc.). Ainsi, plus besoin de configurer votre machine pour chaque projet : il suffit de lancer le conteneur approprié. Les avantages de Docker pour le développement sont notamment :

- **Isolation et cohérence :** chaque projet tourne dans son conteneur avec sa configuration, sans impacter les autres. Fini les conflits de versions (par ex. plusieurs projets nécessitant différentes versions de PHP).
- **Environnement reproductible :** tout est décrit dans la configuration du conteneur. Vous pouvez partager votre configuration Docker avec d’autres développeurs, qui pourront lancer le même environnement en une commande, sans longue procédure d’installation.
- **Nettoyage facile :** un conteneur est éphémère. Si quelque chose ne va pas, on peut le supprimer et en recréer un proprement. Pas de risque de « casser » votre système hôte en installant/désinstallant des services.

**Concepts de base :** Docker fonctionne avec des *images* (modèles tout prêts d’environnements, par ex. une image contenant PHP et Apache) et des *conteneurs* (instances de ces images qui tournent réellement). On va utiliser l’image officielle PHP fournie par Docker Hub.

### 1.1 Installation de Docker

Si ce n’est pas déjà fait, installez Docker Desktop (Windows/Mac) ou Docker Engine (Linux) en suivant la documentation officielle. Vérifiez ensuite que Docker fonctionne : par exemple, exécutez la commande `docker --version` dans un terminal, qui devrait afficher la version de Docker installée.

### 1.2 Création d’un conteneur PHP

Nous allons à présent créer un conteneur qui servira de serveur web PHP pour notre projet :

1. **Récupérer une image PHP** : Dans un terminal (Warp, PowerShell ou autre), tapez la commande  
   ```bash
   docker pull php:8.4-apache
   ```  
   Ceci télécharge l’image Docker contenant PHP 8.1 avec Apache (serveur HTTP) préconfiguré.

2. **Lancer le conteneur** : Toujours dans le terminal, placez-vous dans le dossier de votre projet (où se trouveront vos fichiers PHP) et exécutez :  
   ```bash
    docker run -d --name php-container \
    -p 8080:80 \
    -v $(pwd):/var/www/html \
    php:8.4-apache
   ```  
   Explications :
   - `-d` lance le conteneur en arrière-plan (daemon).
   - `--name phpapp` donne un nom au conteneur (phpapp) pour le reconnaître.
   - `-p 8080:80` fait correspondre le port 80 du conteneur au port 8080 de votre machine. Ainsi, en visitant **http://localhost:8080** vous contacterez le serveur Apache du conteneur.
   - `-v "$PWD":/var/www/html` monte le dossier courant (`$PWD`) dans le conteneur à l’emplacement `/var/www/html` (le répertoire par défaut où Apache cherche les fichiers à servir). Ainsi, vos fichiers PHP locaux seront visibles par Apache à l’intérieur du conteneur. (Sur Windows PowerShell, utilisez `${PWD}` ou spécifiez le chemin complet.)
   - `php:8.4-apache` indique quel image Docker utiliser (ici PHP 8.4, avec Apache).

   Une fois la commande lancée, Docker démarre le conteneur. Vérifiez son statut avec `docker ps` (il devrait apparaître dans la liste des conteneurs actifs).

3. **Tester le serveur web** : Créez un fichier `index.php` dans votre dossier de projet avec le contenu suivant :  
   ```php
   <?php
   phpinfo();
   ?>
   ```  
   Ensuite, dans votre navigateur web, visitez **http://localhost:8080/index.php**. Vous devriez voir s’afficher la page de configuration PHP (générée par la fonction `phpinfo()`), ce qui confirme que PHP fonctionne dans le conteneur. Si c’est le cas, supprimez cette ligne `phpinfo()` qui n’est qu’un test.

4. **Ouvrir un terminal dans le conteneur (optionnel)** : Il peut être utile d’entrer dans le conteneur pour exécuter des commandes Linux ou PHP directement. Par exemple, pour obtenir un shell bash dans notre conteneur, exécutez :  
   ```bash
   docker exec -it php-container bash
   ```  
   Votre invite de commande sera alors à l’intérieur du conteneur (généralement, le prompt change pour afficher quelque chose comme `root@...:/var/www/html#`). Vous pouvez y taper des commandes Unix classiques. Essayez par exemple `php -v` pour afficher la version de PHP dans le conteneur, ou `ls` pour lister les fichiers (vous devriez voir vos fichiers de projet dans `/var/www/html`). Tapez `exit` pour sortir du conteneur et revenir à votre système hôte.

À ce stade, votre environnement est prêt : vous disposez d’un serveur Apache+PHP fonctionnel via Docker. Vous n’avez plus besoin d’un serveur local type WAMP/XAMPP, tout se passe dans le conteneur. Vous pouvez passer au développement PHP en lui-même.

## Module 2 : Premiers pas en PHP – syntaxe et concepts de base

PHP est un langage de script exécuté côté serveur, principalement utilisé pour générer du HTML dynamique. Si vous connaissez JavaScript côté client, notez bien que PHP s’exécute **avant** que la page n’arrive dans le navigateur, et c’est le serveur qui renvoie du HTML (ou du JSON, etc.) généré par PHP. Voici comment démarrer avec les bases du langage PHP.

### 2.1 Syntaxe élémentaire et affichage

Un script PHP est constitué de code enclavé dans des balises PHP `<?php ... ?>`. Le plus simple est d’écrire un script qui affiche du texte. Par exemple :

```php
<?php
   echo "PHP avec Docker";
?>
```

Enregistrez ce code dans un fichier `hello.php` (dans votre dossier de projet monté dans Docker). Via votre navigateur, accédez à **http://localhost:8080/hello.php**. Vous devriez voir apparaître “PHP avec Docker” sur la page. La commande `echo` permet d’écrire du texte en sortie (envoyé au navigateur).  

**Variables** : En PHP, les variables commencent par le signe `$`. Vous pouvez par exemple définir une variable pour personnaliser le message :

```php
<?php
   $nom = "Alice";
   echo "Bonjour, " . $nom . " !";
?>
```

Ici, on définit `$nom` puis on le concatène avec d’autres chaînes de caractères pour l’afficher. Mettez à jour votre fichier et rafraîchissez la page pour voir le résultat. Vous pouvez changer la valeur de `$nom` et tester à nouveau.

**Note** : La concaténation de chaînes se fait avec le point (`.`) en PHP (contrairement au `+` de JavaScript). Alternativement, vous pouvez inclure la variable directement dans la chaîne avec la syntaxe `"Bonjour $nom !"`, mais faites attention aux guillemets doubles (qui interprètent les variables) vs simples (qui n’interprètent pas).

**Tableaux de type de données supportés par PHP :**

| Type de données | Description | Exemple |
| --- | --- | --- |
| String | Chaîne de caractères | "Hello, world!" |
| Integer | Entier | 42 |
| Float | Nombre décimal | 3.14 |
| decimal | Nombre décimal avec virgule | 3.14 |
| Boolean | Valeur logique | true |
| NULL | Valeur nulle | null |
| Array | Tableau | array("Hello", "world!") ou [1, 2, 3] |
| Object | Objet | object("name" => "John", "age" => 20) |

### 2.2 Récupérer des paramètres (GET/POST)

PHP excelle à générer des pages dynamiques en fonction de requêtes. Les données envoyées par le navigateur (formulaires, paramètres d’URL) sont accessibles via des *variables superglobales* comme `$_GET` et `$_POST`. 

- `$_GET` contient les paramètres passés dans l’URL (requête HTTP GET, généralement via l’URL après un `?`).
- `$_POST` contient les données envoyées via un formulaire en méthode POST.

Essayons un exemple avec `$_GET` : modifiez `hello.php` pour qu’il salue un nom passé en paramètre d’URL :

```php
<?php
   $nom = $_GET['nom'] ?? "visiteur";  // Récupère le paramètre 'nom' s'il existe, sinon 'visiteur'
   echo "Bonjour, " . htmlspecialchars($nom) . " !";
?>
```

Ici, `$_GET['nom']` récupère la valeur du paramètre `nom` dans l’URL. Le `?? "visiteur"` signifie “si aucun nom n’est fourni, utiliser ‘visiteur’ par défaut”. On utilise aussi `htmlspecialchars()` pour des raisons de sécurité (il échappe les caractères spéciaux pour éviter des problèmes si quelqu’un met du HTML ou du script dans le nom).

Enregistrez et accédez à **http://localhost:8080/hello.php?nom=Alice**. Vous devriez voir “Bonjour, Alice !”. Essayez avec d’autres noms ou sans paramètre pour voir le comportement par défaut.

De même, pour un formulaire HTML envoyé en POST, vous retrouveriez les valeurs dans `$_POST['champ']`. Nous utiliserons `$_POST` plus loin pour traiter un formulaire d’envoi de question à l’API GPT.

### 2.3 Contrôle et boucles

PHP propose les structures de contrôle habituelles : `if/else`, boucles `for`, `while` et surtout `foreach` (très utile pour parcourir des tableaux/objets).

Par exemple, créons un petit script qui affiche les éléments d’une liste. Nous allons simuler une liste de technologies à partir d’un tableau PHP et les afficher en HTML :

```php
<?php
   $technos = ["HTML", "CSS", "JavaScript", "PHP"];
?>
<ul>
<?php foreach ($technos as $techno): ?>
   <li><?= $techno ?></li>
<?php endforeach; ?>
</ul>
```

En PHP, on peut mélanger du HTML et du PHP. Ci-dessus, on sort du mode PHP pour écrire une balise `<ul>`, puis on utilise une syntaxe spéciale `<?php foreach (...) : ?> ... <?php endforeach; ?>` qui permet d’alterner PHP et HTML facilement (équivalent à ouvrir une accolade `{` et la refermer après le bloc). À l’intérieur, `<?= $techno ?>` est un raccourci pour `<?php echo $techno ?>`. Si vous enregistrez ce code dans `liste.php` et l’exécutez, vous obtiendrez une liste HTML avec les 4 technologies.

**Ce que l’on a appris jusqu’ici :** créer des variables, afficher du texte, récupérer des entrées utilisateur, utiliser des boucles pour générer du HTML dynamique. Avec ces bases, nous pourrons lire et afficher des données issues de fichiers JSON dans le prochain module.

## Module 3 : Manipulation de fichiers JSON en PHP

Dans de nombreuses applications, on a besoin de stocker et lire des données. Plutôt que d’utiliser une base de données (complexe à mettre en place pour nos besoins actuels), nous utiliserons des **fichiers JSON** pour stocker des informations de manière simple et structurée. Le format JSON (JavaScript Object Notation) est un format texte très répandu pour représenter des objets ou tableaux de données, que vous connaissez sans doute via JavaScript. PHP offre des fonctions natives pour lire et écrire du JSON facilement.

### 3.1 Lire un fichier JSON

Supposons que vous avez un fichier `team.json` contenant une liste de membres d’équipe avec leur rôle, par exemple :

```json
[
  { "nom": "Alice", "role": "Développeuse" },
  { "nom": "Bob",   "role": "Designer" }
]
```

*(Créez ce fichier `team.json` à la racine de votre projet et copiez-y ce contenu pour suivre l’exemple.)*

Pour lire ce fichier en PHP et exploiter les données, suivez ces étapes :

1. **Lire le contenu brut** : on utilise `file_get_contents()` qui lit un fichier et renvoie une chaîne de caractères avec tout son contenu.
2. **Convertir le JSON en structure PHP** : on utilise `json_decode()` qui transforme une chaîne JSON en équivalent PHP (typiquement un tableau ou un objet PHP). On peut demander à obtenir un tableau associatif en passant `true` en second argument.

Exemple de code pour lire et afficher les membres de l’équipe :

```php
<?php
   $json = file_get_contents('team.json');            // 1. Lecture du fichier
   $team = json_decode($json, true);                  // 2. Décodage JSON en tableau PHP

   foreach ($team as $member) {
       echo $member['nom'] . " - " . $member['role'] . "<br>";
   }
?>
```

Si vous exécutez ce script (par exemple dans une page temporaire `test_json.php`), vous devriez voir s’afficher chaque membre et son rôle (chaque paire nom-role suivie de `<br>` pour le saut de ligne). Vous pouvez utiliser cette technique pour peupler du HTML. Par exemple, au lieu d’un simple `echo`, vous pourriez générer des `<li>` comme montré précédemment.

**Vérification** : Assurez-vous que `json_decode` a bien fonctionné. S’il y a une erreur de syntaxe dans le JSON, `$team` vaudra `null`. Dans ce cas, utilisez `json_last_error_msg()` pour afficher le message d’erreur JSON. Ici, si vous avez copié correctement le JSON fourni, tout devrait bien fonctionner.

### 3.2 Écrire dans un fichier JSON

Maintenant, voyons comment ajouter des données et les sauvegarder dans le fichier. En PHP, on pourra :

- Transformer nos données en JSON avec `json_encode()`.
- Écrire dans le fichier avec `file_put_contents()`.

Imaginons qu’on veuille ajouter un nouveau membre à `team.json` via PHP. On peut procéder ainsi :

```php
<?php
   $json = file_get_contents('team.json');
   $team = json_decode($json, true);

   // Nouvelle entrée à ajouter
   $newMember = ["nom" => "Charlie", "role" => "Chef de projet"];
   $team[] = $newMember;  // on ajoute au tableau

   // Sauvegarde du tableau mis à jour en JSON dans le fichier
   $newJson = json_encode($team, JSON_PRETTY_PRINT);
   file_put_contents('team.json', $newJson);
   echo "Nouveau membre ajouté !";
?>
```

Ici on lit et décode le fichier comme précédemment, on ajoute un élément au tableau (`$team[] = ...`), puis on ré-encode en JSON. L’option `JSON_PRETTY_PRINT` n’est pas obligatoire mais rend le fichier JSON écrit plus lisible. Enfin, `file_put_contents` écrase le fichier existant avec le nouveau contenu JSON.

> ⚠️ **Attention** : si deux utilisateurs écrivaient en même temps, on pourrait perdre des données (condition de course). Pour notre apprentissage ce n’est pas un problème, mais gardez en tête que ce n’est pas une solution de stockage robuste en production. Ici, cela remplace une base de données de façon *simplifiée* pour apprendre.

Après avoir exécuté ce code, ouvrez `team.json` : vous devriez voir le nouveau membre ajouté dans le JSON. Vous savez maintenant lire et écrire des fichiers JSON avec PHP, ce qui nous sera utile pour stocker des informations (par exemple, l’historique des questions posées à l’API plus tard).
