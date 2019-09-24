# Compétences pro orchestration

## Configuration

Pour accéder à l'api, il faut rajouter une entrée dans votre fichier `/etc/hosts` :

    127.0.0.1 api.localhost

## Usage

Pour démarrer l'environnement de développement :

Pre-requis Mac : `docker-sync` (install avec `gem install docker-sync`)


    git clone git@github.com:betagouv/competences-pro.git client
    git clone git@github.com:betagouv/competences-pro-serveur.git serveur

    ./script/demarre

Et se rendre sur http://localhost:4000

## Déploiement

Pour déployer l'application, des *playbooks* [ansible][] sont fournis.

Pre-requis : `ansible`

Créer un fichier `hotes` en rajoutant l'adresse du serveur déployé :

    example.net:22

## Déploiement de traefik

Nous utilisons [Traefik][] pour exposer les différents containers vers l'extérieur. Vous pouvez personaliser sa configuration en éditant le fichier `traefik.toml`.

Puis lancer le déploiement (avec l'utilisateur distant `utilisateur`):

    ansible-playbook --user=utilisateur --inventory=hotes traefik.yml

### Déploiement du site site web vitrine

    ansible-playbook --user=utilisateur --inventory=hotes site.yml

### Déploiement de l'application

#### Pre-requis de la machine cible

La machine cible doit avoir *docker*, *docker-compose* et *git* installés.
De plus, un fichier de configuration doit également être présent.

Un fichier `.env.serveur.prod`. `SECRET_KEY_BASE` et `DATABASE_URL` devraient être modifiés.

    SECRET_KEY_BASE=ICI_METTRE_UNE_VRAI_SECRET_KEY_BASE
    RAILS_SERVE_STATIC_FILES=true
    RAILS_LOG_TO_STDOUT=true
    DATABASE_URL=postgres://postgres:@postgres/postgres
    JETON_SERVEUR_ROLLBAR=

    ## information de connexion pour la sauvegarde BDD
    POSTGRES_USER=postgres
    POSTGRES_PASSWORD=
    POSTGRES_DB=postgres

#### Sur la machine qui fait le déploiement

Puis lancer le déploiement (avec l'utilisateur distant `utilisateur`):

    ansible-playbook --user=utilisateur --inventory=hotes deploiement.yml

Il est également possible de personnaliser les chemins, les branches ou les hôtes :

    ansible-playbook --inventory=hotes --user=utilisateur --extra-vars="chemin_racine={{ansible_env.HOME}}/preprod hote_client=preprod.competences-pro.beta.gouv.fr hote_serveur=apipreprod.competences-pro.beta.gouv.fr" deploiement.yml

Voici la liste des variables typique à personnaliser :

- `chemin_racine`
- `version_orchestrateur`
- `version_serveur`
- `version_client`
- `hote_client`
- `hote_serveur`

**Attention : il faut impérativement spécifier un `chemin_racine` différent de `{{ ansible_env.HOME }}` si on souhaite déployer d'autres versions que `master`. Dans le cas contraire, l'application pointera vers la base de données de production, avec tous les risques de corruption que cela peut entraîner.**

## Licence

Ce logiciel et son code source sont distribués sous [licence AGPL](https://www.gnu.org/licenses/why-affero-gpl.fr.html).

[ansible]: https://www.ansible.com/
[traefik]: https://traefik.io/
