## Créer un nouveau capteur pour HomeAssistant avec ESPHome


j'ai acheté pour quelques euros des cartes esp32 pour tester de créer mon propre module compatible avec Home assistant.
Je me suis rendu compte que la version de mon ESP32 n'était pas encore "très" compatible avec ESPHome: c'est un ESP32c6.

J'ai trouvé de quoi me le connecter après avoir pas mal galéré. Je vous conseille d'aller vérifier sur le [site d'ESPhome](http://esphome.io) les microcontroleurs compatibles avant de les acheter, celà vous évitera des galères.

Pour ce qui concerne l'esp32c6 voilà l'adresse du forum qui m'a guidé:

https://www.reddit.com/r/esp32/comments/1fh8nup/can_you_run_esphome_on_esp32c6/

Il faut déjà se rendre sur esphome.io puis installer le cli esphome.

plus d'infos sur le site [esphome.io]()

On installe le Cli esphome puis on peut installer le firmware sur l'esp avec la commande:

`esphome run config.yml`

Gestion des mots de passes.
C'est une bonne pratique de ne pas mettre les mots de passe dans le fichier de code principal (celui-ci peut être partagé sur github par ex)

On crée un fichier secrets.yml ou l'on définit les clés et leurs valeurs puis on y fait reférence avec la variable !secrets dans le fichier config.yml


