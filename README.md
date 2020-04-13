# Mise en place d'un serveur DNS / DHCP / LDAP

## Prérequis

Pour la mise en place de nos services, il est necessaire d'avoir:
- 3 machines virtuelles avec une adresse IP de configurer pour chaque
- L'outil Ansible d'installé pour lancer le déploiement

## Adapter l'infrastrcture

Avant de lancer le déploiement, il faut s'assurer de plusieurs choses.
1. Vérifier que les différentes informations dans le ansible.cfg sont correctes (emplacement de la clé SSH).
2. Vérifier que les adresses IP dans inventory.ini sont bien correctes pour notre infrastrcture.
3. Vérifier dans playbook.yml que remote_user soit bien identique à l'utilisateur avec les privilèges de chaques machines.

## Lancement du playbook

Une fois toutes les étapes précédentes validées, on peut lancer notre déploiement:
`ansible-playbook playbook.yml -i inventory.ini -K`

Il ne reste plus qu'à attendre la fin avant de pouvoir tester nos différents services.

## Test à effectuer

On peut retrouver les différents tests à effectuer pour vérifier le bon fonctionnement de nos services dans le dossier "Documentation" à la racine de notre projet.
