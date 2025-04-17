Je viens de démarrer un projet de publication de mes prises de notes; Dans la même philosophie des autres sujet. Local first (pas de risque de lock in cloud). Mais moderne, sans distraction et interconnecté.

J'ai choisi obsidian car même si le produit n'est pas 100% opensource il se contente de traduire des fichiers Markdown, standard ouvert et répandu de mise en page. Il y a également un certain nombre de plugins intéressant qui facilitent l'organisation.

Il a l'avantage de permettre de connecter facilement des notes et de les gérer à la manière d'un graph de connaissance, sujet qui me tient à coeur au travail depuis un moment déjà.


L'un des aspects intéressants est de permettre la publication sur internet de notes prises. Obsidian propose un abonnement payant mais il y a une autre manière de les publier, grâce à github pages et github action: des outils fourni par le site de partage de code Github.

Pour rendre les notes plus facile à lire sur internet, un projet opensource (quartz, permet de transformer des fichiers markdown en un site HTMML tout propre)

Ci joint le lien vers "Quartz" https://quartz.jzhao.xyz/ et un tuto sur comment le déployer.

Pour éviter le "bazar" de tous les fichiers du framework de Quartz, et pour qu'ils soient bien séparés de mon dossier Obsidian, je n'ai pas intégrer les notes obsidian au dossier "content" de Quartz, mais j'ai remplacé ce dossier par un lien dynamique vers mon dossier Vault Obsidian que je souhaite publier.

Une fois dans le dossier Quartz, je supprime le dossier content puis je crée un lien vers le "vault" que je veux publier

ln -s ../../mes-notes-perso content

==> pour publier mes notes sur le web, après avoir fait des modifications sur obsidian, je lance la commande suivante:

npx quartz sync  

et quelques minutes plus tard le site est publié sur mon espace github

gbelbe.github.io/my-notes






