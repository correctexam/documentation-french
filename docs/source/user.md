(user1)=
# Vidéo de démontrations

Une vidéo de démonstration est accessible [ici](https://www.youtube.com/watch?v=bYTBlTg37RM&t)

# Foire aux questions

## Comment créer un compte sur la plateforme ?

Vous pouvez vous rendre sur la [plateforme](https://correctexam.github.io/corrigeExamFront/), cliquer sur **s'inscrire**, remplir quelques informations. Vous recevrez un e-mail pour valider votre compte.

![](Register.png)
![](Register1.png)

Vous pouvez ensuite remplir les informations relatives à votre compte dans le menu. *Compte* -> *Réglages*


## Comment créer un nouveau cours/unité d'enseignement ?

Une fois authentifié, sur la page d'accueil, cliquez sur le symbole **+** *créer un cours*.
Il est nécessaire de donner un nom au cours.


## Puis-je partager une correction avec un ou plusieurs collègues ?

Lorsque vous cliquez sur le module, vous avez accès dans la liste des actions à une action partagée qui vous permet de partager ce module avec un ou plusieurs collègues. Ces derniers verront alors ce module dans la liste de leurs modules.

![](share.png)
![](share1.png)

## Puis-je limiter les droits d'un collègue afin qu'il ne puisse accéder qu'à un sous-ensemble de questions ?

Non, un collègue, nous pouvons lui faire confiance ;). Nous n'implémentons pas de règles RBAC spécifiques par examen pour le moment.

## Avez-vous un diagramme des principales étapes à suivre pour utiliser cette application ?

![](./ScanExam.png)



## Comment créer un examen ?

Pour créer un examen, une fois entré dans la page d'un module, il est possible de créer un nouvel examen avec la commande (+) ou importer un examen existant. 

## Où puis-je trouver des modèles pour Word latex et libreoffice ?

Dans la vue qui vous permet de créer un examen, vous avez accès à un certain nombre de modèles pour créer votre examen. La philosophie de l'application est de permettre à chaque enseignant de créer son examen avec l'outil qui lui convient.


## Pourquoi suis-je obligé de télécharger une liste d'élèves ?

Pour corriger, nous associons chaque clé de réponse à un étudiant. Cette liste est nécessaire pour effectuer la tâche d'affectation. (Au pire si vous voulez aucun , nom vous pouvez toujours associé avec des numéros entrés sur la copie et une liste d'étudiant avec comme nom étudiant1/étudiant2, ...)

## Dois-je supprimer un étudiant qui n'a pas composé ? 

Non, il sera marqué comme ABI par défaut. 

## Pouvez-vous expliquer les différents types de questions disponibles ?

Pour l'instant, il y a grosso modo quatre types de questions. 

- Les **QCM** pour lesquelles l'application fournit une aide à la notation. 
- La notation **DIRECTE** (*Manuelle et Directe*) pour laquelle l'enseignant note manuellement les réponses à cette question. 
- La notation **POSITIVE** (*manuelle et POSITIVE*). Il s'agit d'un élément pour lequel l'enseignant peut définir un ensemble de commentaires en cours de route qui donne des points aux réponses à cette question (on part de zéro). Le nombre total de points obtenus ne peut dépasser le nombre maximum de points associés à cette question.
- La notation **NEGATIVE** (*Manuel et NEGATIVE*). Il s'agit d'un élément pour lequel le correcteur peut définir un ensemble de commentaires en cours de route qui enlève des points à la réponse en question (on part du nombre de points maximum possible pour cette question). Le nombre total de points obtenus ne peut être inférieur à zéro.

Ce type de questions sera enrichi à l'avenir. N'hésitez pas à nous faire part de vos bonnes idées.

## Puis-je changer le type de question lorsque la correction d'une question a commencé ?

Honnêtement, ce n'est pas recommandé car pas assez testé.

## Comment nettoyer le scan des élèves s'il manque des pages, s'il y a des doublons, s'il y a des pages retournées... ?

Utilisez [pdfarranger](https://github.com/pdfarranger/pdfarranger). C'est un excellent outil pour cette tâche.

## Puis-je recharger un scan propre (voir question précédente) et le réaligner si j'ai déjà commencé à corriger un examen ?

Oui, pas de problème, si vous partagez la correction avec des collègues ou avec différents appareils, vous pouvez forcer le chargement et le téléchargement vers le serveur plus tard. 

## Que se passe-t-il si mon modèle a un nombre impair de pages (*e.g.* 3 pages) et que le scan des feuilles des étudiants est un multiple de 2 (*e.g.* 4 pages par étudiant) ?

Pour l'instant, cela ne se gère pas correctement au moment de l'alignement. Il sera nécessaire de supprimer les pages blanches des scans des élèves au préalable avec des outils de manipulation de pdf comme [pdftk](https://www.pdflabs.com/tools/pdftk-the-pdf-toolkit/) ou [pdfarranger](https://github.com/pdfarranger/pdfarranger).

## Comment puis-je numériser les feuilles des élèves ?

Nous recommandons d'utiliser des niveaux de gris à 150 DPI pour que la taille du fichier reste raisonnable, mais il n'y a pas de problème pour un scan en couleur à 300 DPI si nécessaire. Évitez le noir et blanc pur qui pourrait nuire à l'algorithme qui reconnaît les noms/prénoms/identifiants (INE) des étudiants.



## Puis-je rédiger une déclaration avec un autre outil que word, excel ou latex ?

Bien sûr, le seul point important est un marqueur circulaire dans les coins pour faciliter l'alignement et des carrés gris clair pour mettre les noms, prénoms et identifiants (INE) des étudiants. 

## Que faire lorsque je rencontre un bug dans l'application ?

Utilisez le système de ticket github sur le projet https://github.com/correctexam/corrigeExamFront

## Puis je installer sur mon ordinateur la solution pour éviter d'utiliser la version en ligne et corriger hors ligne (dans le train par example)


Nous fournissons des [releases packagés](https://github.com/correctexam/corrigeExamBack/releases) pour fonctionner sur les trois systèmes d'exploitation (windows, macos, linux) pour les architecture AMD64 muni d'une base de données intégré. 

Sous linux et macos, vous pouvez juste télécharger le binaire pour votre os, rendre se binaire exécutable, lancer l'application et aller sur votre navigateur à l'adresse http://localhost:8080 (utilisateur par défaut user/user ou admin/admin).

Sous windows, il sera nécessaire de télécharger l'exécutable mais aussi les deux fichiers *mydb...* correspondant à la base de données. Placer ces trois fichiers dans le même répertoire et lancer l'exécutable.  Aller sur votre navigateur à l'adresse http://localhost:8080 (utilisateur par défaut user/user ou admin/admin).

:::{note}
Les données du projet peuvent ensuite être exporté puis importé sur la plateforme en ligne, entres autres si vous souhaité testé l'envoi de mail aux étudiants.
:::

## Puis je détruire mes données sur la plate-forme

Oui à tout moment, nous vous conseillons de faire un backup (cours par cours) si vous souhaitez un jour réutliliser certaines données. Quand vous détruisez vos données, aucun backup n'est conservé donc nous ne pourrons restituer les données. 

## Quelles garanties j'ai de ne pas perdre de données en cas de problème sur la plateforme 

Pour le moment, la plateforme accessible en ligne https://correctexam.github.io/corrigeExamFront/ est proposé en mode *best effort* sur un serveur de l'Université de Rennes 1. Aucune garantie sur les donénes n'est fournie. Nous vous invitons à vous rapprocher du service informatique de votre établissement si vous souhaitez un meilleur niveau de garantie

