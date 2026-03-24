# 🚀 Guide complet de déploiement : Flask (Factory Pattern) + SQLite sur PythonAnywhere

Félicitations pour avoir finalisé votre application ! Ce guide vous montre comment prendre votre projet sur votre PC Windows, le sauvegarder sur GitHub en utilisant VS Code, et le déployer en ligne sur PythonAnywhere.

---

## Phase 1 : Préparation du projet en local (Windows)

Avant de publier votre code, il faut préparer l'environnement local pour la production.

1. **Créer le fichier `requirements.txt` :**
   Ouvrez le terminal intégré de VS Code (`Ctrl` + `\``), assurez-vous que votre environnement virtuel est activé, et exécutez :
   ```bash
   pip freeze > requirements.txt
   ```
2. **Créer un fichier `.gitignore` :**
   Vous ne devez **jamais** envoyer votre environnement virtuel, votre base de données locale ou vos mots de passe (le fichier `.env`) sur GitHub. Créez un fichier `.gitignore` à la racine de votre projet et ajoutez-y strictement :
   ```text
   venv/
   env/
   __pycache__/
   instance/
   .env
   *.sqlite3
   *.db
   ```

---

## Phase 2 : Versionning avec VS Code et GitHub

Nous allons maintenant sauvegarder votre code sur GitHub en utilisant l'interface de Visual Studio Code. Assurez-vous d'avoir [Git installé](https://git-scm.com/) sur votre machine.

### Étape A : Préparer et valider le code localement
1. **Initialiser le dépôt :** Dans la barre latérale gauche de VS Code, cliquez sur l'icône **Source Control** (qui ressemble à un petit graphe). Cliquez ensuite sur le bouton **Initialize Repository**.
2. **Préparer les fichiers (Staging) :** Une liste de vos fichiers va apparaître sous l'onglet "Changes". Survolez la ligne "Changes" et cliquez sur le petit bouton **+** (Stage All Changes). Vos fichiers passent dans la section "Staged Changes". *(Si votre `.gitignore` est correct, le dossier `venv` et le fichier `.env` n'apparaîtront pas).*
3. **Créer le Commit :** Dans le champ de texte, tapez un message (ex: *"Premier commit de l'application Flask"*), puis cliquez sur le bouton bleu **Commit**.

### Étape B : Relier à un dépôt GitHub existant
Si vous avez déjà créé un dépôt vide sur le site GitHub.com (sans fichier README ni `.gitignore`), voici comment le lier :
1. **Récupérer l'URL :** Sur la page GitHub de votre dépôt, copiez l'adresse web fournie (ex: `https://github.com/votre-pseudo/nom-du-depot.git`).
2. **Ajouter le lien (Remote) :** Dans le terminal de VS Code, tapez la commande suivante (avec votre lien) :
   ```bash
   git remote add origin https://github.com/votre-pseudo/nom-du-depot.git
   ```
3. **Envoyer le code (Push) :** Pour envoyer votre code sur GitHub, tapez :
   ```bash
   git push -u origin main
   ```
   *(Note : si votre branche locale s'appelle `master` au lieu de `main`, tapez plutôt `git push -u origin master`).*

---

## Phase 3 : Configuration de PythonAnywhere et Récupération

1. **Créer un compte :** Inscrivez-vous gratuitement sur [PythonAnywhere](https://www.pythonanywhere.com). Votre nom d'utilisateur fera partie de l'URL de votre site (ex: `votre_pseudo.pythonanywhere.com`).
2. **Ouvrir une console Bash :** Sur le tableau de bord de PythonAnywhere, allez dans l'onglet **Consoles** et lancez une nouvelle console **Bash**.
3. **Cloner le dépôt GitHub :** Dans la console, tapez cette commande pour télécharger votre code :
   ```bash
   git clone https://github.com/votre-pseudo/nom-du-depot.git
   ```

---

## Phase 4 : L'Environnement Virtuel sur le Serveur

Toujours dans la console Bash de PythonAnywhere, créez un environnement isolé pour votre projet.

1. **Créer l'environnement :** Exécutez cette commande (remplacez `python3.10` par votre version, et `mon_env` par le nom souhaité) :
   ```bash
   mkvirtualenv --python=/usr/bin/python3.10 mon_env
   ```
   *(Le terminal affichera `(mon_env)`).*
2. **Installer les dépendances :** Entrez dans le dossier cloné et installez vos bibliothèques :
   ```bash
   cd nom-du-depot
   pip install -r requirements.txt
   pip install python-dotenv
   ```

---

## Phase 5 : Configuration de l'Application Web

1. Allez dans l'onglet **Web** sur le tableau de bord PythonAnywhere.
2. Cliquez sur **Add a new web app**.
3. **⚠️ TRÈS IMPORTANT :** Choisissez **Manual Configuration** (et non pas Flask).
4. Sélectionnez la même version de Python que votre environnement virtuel.
5. Dans la section **Virtualenv**, entrez le nom de votre environnement (ex: `mon_env`).
6. Dans la section **Code**, modifiez **Source code** et **Working directory** pour y mettre le chemin absolu vers votre projet (ex: `/home/votre_pseudo/nom-du-depot`).

---

## Phase 6 : Le fichier WSGI et les Variables d'Environnement

C'est ici que l'on connecte le serveur à votre `create_app()`.

1. **Créer le fichier `.env` :** Allez dans l'onglet **Files** de PythonAnywhere. Naviguez dans votre dossier de projet et créez un fichier `.env`. Ajoutez-y vos secrets (comme `SECRET_KEY=mon_super_secret`).
2. **Modifier le WSGI :** Retournez dans l'onglet **Web** et cliquez sur le lien sous **WSGI configuration file** (ex: `/var/www/votre_pseudo_pythonanywhere_com_wsgi.py`).
3. Effacez tout le contenu et collez ce code. **Adaptez les chemins et les noms :**

```python
import sys
import os
from dotenv import load_dotenv

# 1. Ajouter le dossier du projet au système
project_folder = '/home/votre_pseudo/nom-du-depot'
if project_folder not in sys.path:
    sys.path = [project_folder] + sys.path

# 2. Charger les variables d'environnement
load_dotenv(os.path.join(project_folder, '.env'))

# 3. Importer votre Factory (remplacez "nom_de_votre_dossier_app" par votre dossier contenant __init__.py)
from nom_de_votre_dossier_app import create_app

# 4. Créer l'objet "application" (Nom obligatoire pour PythonAnywhere)
application = create_app()
```
*Sauvegardez le fichier.*

---

## Phase 7 : Base de Données SQLite et Initialisation

Sur un serveur, SQLite a besoin d'un **chemin absolu** pour ne pas se perdre.

1. **Vérification du code Flask :** Assurez-vous que la configuration de votre base de données dans votre code source ressemble à ceci :
   ```python
   import os
   basedir = os.path.abspath(os.path.dirname(__file__))
   
   SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
       'sqlite:///' + os.path.join(basedir, 'app.db')
   ```

2. **Créer les tables :** Ouvrez une console **Bash** sur PythonAnywhere. Vérifiez que `(mon_env)` est activé (sinon tapez `workon mon_env`). Lancez Python et exécutez vos commandes de création dans le contexte de l'application :
   ```python
   python
   >>> from nom_de_votre_dossier_app import create_app, db
   >>> app = create_app()
   >>> app.app_context().push()
   >>> db.create_all()
   >>> exit()
   ```

---

## Phase 8 : Gérer les fichiers statiques (CSS, Images, JavaScript)

Si votre site s'affiche sans couleurs, sans mise en page (CSS manquant) ou avec des images brisées, c'est normal ! En production, il est fortement recommandé de laisser le serveur web distribuer ces fichiers directement, sans passer par Flask.

PythonAnywhere possède un outil dédié pour cela.

1. **Aller dans l'onglet Web :** Sur votre tableau de bord PythonAnywhere, allez dans la section **Web** de votre application.
2. **Trouver la section "Static files" :** Descendez au milieu de la page jusqu'à voir la rubrique **Static files** (Fichiers statiques).
3. **Configurer l'URL (le lien web) :**
   * Cliquez sur le texte bleu **Enter URL**.
   * Tapez exactement : `/static/`
   * Appuyez sur la coche bleue pour valider.
4. **Configurer le chemin (Directory) :**
   * Cliquez sur le texte bleu **Enter path** situé juste à côté.
   * Entrez le **chemin absolu** vers le dossier `static` de votre projet. Il doit ressembler à ceci (adaptez les noms !) :
     `/home/votre_pseudo/nom-du-depot/nom_de_votre_dossier_app/static`
   * Appuyez sur la coche bleue pour valider.
5. **Recharger le site :** Remontez tout en haut de la page et cliquez sur le gros bouton vert **Reload**.

*💡 **Astuce de pro :** Si après avoir rechargé la page, votre design n'apparaît toujours pas ou ne semble pas à jour, c'est souvent la faute de votre navigateur internet qui a gardé l'ancien design en mémoire (le cache). Sur votre site, faites un rechargement forcé en appuyant sur `Ctrl + F5` (Windows) ou `Cmd + Maj + R` (Mac).*

---

## 🚀 Étape Finale : Lancement !

Retournez sur l'onglet **Web** de PythonAnywhere et cliquez sur le gros bouton vert **Reload**. 
Visitez votre adresse web (ex: `votre_pseudo.pythonanywhere.com`) : votre application est en ligne ! 🎉

Voici une section de dépannage (troubleshooting) parfaite pour compléter le guide. Elle regroupe les problèmes les plus fréquents que les étudiants rencontrent lors de leur premier déploiement.

Vous pouvez ajouter cette partie à la toute fin du document précédent.

***

## 🚑 Annexe : Guide de Dépannage (Troubleshooting)

Il est très fréquent que tout ne fonctionne pas du premier coup en production. Pas de panique ! Voici comment résoudre les problèmes les plus courants.

### 1. J'ai une page "500 Internal Server Error"
C'est l'erreur la plus classique. Elle signifie simplement que votre code Python a planté, mais pour des raisons de sécurité, le serveur ne montre pas l'erreur aux visiteurs.
* **La solution :** Allez dans l'onglet **Web** de PythonAnywhere. Descendez jusqu'à la section **Log files** et cliquez sur le lien de l'**Error log**. 
* Allez tout à la fin de ce fichier texte (les erreurs les plus récentes sont en bas) pour lire le "Traceback" Python. Il vous dira exactement à quelle ligne et dans quel fichier l'erreur s'est produite.

### 2. L'erreur dans les logs est `ModuleNotFoundError: No module named 'X'`
Le serveur ne trouve pas l'une de vos bibliothèques (par exemple, `flask_sqlalchemy` ou `dotenv`).
* **La cause :** Soit vous avez oublié de l'ajouter dans votre `requirements.txt`, soit votre environnement virtuel n'est pas bien configuré dans l'onglet Web.
* **La solution :** Ouvrez une console Bash, activez votre environnement (`workon mon_env`), et installez le module manquant manuellement : `pip install nom_du_module`. N'oubliez pas de cliquer sur le bouton **Reload** dans l'onglet Web ensuite.

### 3. J'ai une erreur de Base de Données (ex: `OperationalError: no such table`)
L'application fonctionne, mais dès que vous essayez de vous connecter ou d'afficher des données, ça plante.
* **La cause :** La base de données SQLite n'a pas été créée au bon endroit, ou les tables n'ont pas été générées.
* **La solution :** 1. Vérifiez bien la Phase 7 du guide. Avez-vous utilisé un chemin absolu (`basedir`) ?
    2. Avez-vous bien fait le `app.app_context().push()` avant le `db.create_all()` dans la console ? Refaites la Phase 7 avec soin.

### 4. J'ai modifié mon code sur mon PC, mais le site ne change pas !
C'est normal, PythonAnywhere ne se met pas à jour tout seul par magie quand vous modifiez votre code local.
* **Le processus de mise à jour (Workflow) :**
    1. Sur votre PC (VS Code) : Faites vos modifications, faites un **Commit**, puis un **Push** vers GitHub.
    2. Sur PythonAnywhere : Ouvrez votre console Bash, allez dans votre dossier (`cd nom-du-depot`) et tapez **`git pull`** pour télécharger la nouvelle version.
    3. Sur PythonAnywhere (Onglet Web) : Cliquez impérativement sur le gros bouton vert **Reload**.

### 5. Je vois mon code source au lieu de mon site
Si en allant sur votre adresse, vous voyez des dossiers ou le texte de votre code Python...
* **La cause :** Vous n'avez pas choisi "Manual Configuration" lors de la création de la Web App, ou votre fichier WSGI (Phase 6) est mal configuré et ne lance pas l'objet `application`.

***

