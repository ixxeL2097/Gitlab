[Retour menu principal](../README.md)

## 4. Configurer les logs
### Dossiers

Par défaut les logs sont stockés dans le volume mappé _**'/srv/gitlab/logs'**_. Voici l'arborescence par défaut:

```bash
logs/
├── alertmanager
├── gitaly
├── gitlab-exporter
├── gitlab-rails
├── gitlab-shell
├── gitlab-workhorse
├── grafana
├── logrotate
├── nginx
├── postgres-exporter
├── postgresql
├── prometheus
├── reconfigure
├── redis
├── redis-exporter
├── sidekiq
├── sshd
└── unicorn

18 directories
```
Il est bien sûr possible de gérer chaque dossier de logs indépendamment via des options à passer dans la variable d'environnement GITLAB_OMNIBUS_CONFIG. Voici un exemple de syntaxe pour quelques répertoires:
```bash
gitlab_rails['log_directory'] = "/var/log/gitlab/gitlab-rails"
unicorn['log_directory'] = "/var/log/gitlab/unicorn"
registry['log_directory'] = "/var/log/gitlab/registry"
```
De manière générale:
```bash
<nom_du_service>['log_directory'] = "/path/to/the/logs/directory"
```
### runit logs

Le service runit-managed dans Gitlab Omnibus génére des données de log en utilisant **svlogd**. La documentation de svlogd est disponible ici : http://smarden.org/runit/svlogd.8.html

Il est possible de modifier la configuration svlogd via _**'/etc/gitlab/gitlab.rb'**_ avec les paramètres suivants:

```bash
# Ci-dessous les valeurs par défaut
logging['svlogd_size'] = 200 * 1024 * 1024            # rotation après 200 MB de données log
logging['svlogd_num'] = 30                            # garde 30 fichiers de rotation log
logging['svlogd_timeout'] = 24 * 60 * 60              # rotation après 24 heures
logging['svlogd_filter'] = "gzip"                     # compression des logs en GZIP
logging['svlogd_udp'] = nil                           # Transmission des logs via UDP
logging['svlogd_prefix'] = nil                        # prefix personnalisé pour les messages logs

# Optionnellement, on peut écraser le prefix dans les logs (ici nginx) 
nginx['svlogd_prefix'] = "nginx"
```
---------------------------------------------------------------------------------------------------------------------------------

[Retour menu principal](../README.md)

[Suivant](05-PostgreSQL-BDD.md)
