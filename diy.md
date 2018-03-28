# DIY: ELK par l'exemple

?> Tutoriel basé sur le livre d'Adrien Mouat: *Using Docker - Developing and Deploying Software with Containers*

Le but va être de pouvoir déployer un pipeline ELK dans un environnement Docker afin de récolter, stocker et analyser les logs d'une application.

Dans un second temps, nous monitorerons l'état général de notre instance OpenShift et l'état des noeuds et pods.

## Monitorer une application

Création d'un premier fichier Compose `prod-with-logging.yml`:
 ```
proxy:
    image: amouat/proxy:1.0
    links:
        - identidock
    ports:
        - "80:80"
    environment:
        - NGINX_HOST=10.0.2.15
        - NGINX_PROXY=http://identidock:9090
identidock:
    image: amouat/identidock:1.0
    links:
        - dnmonster
        - redis
    environment:
        ENV: PROD
dnmonster:
    image: amouat/dnmonster
redis:
    image: redis
logspout:
    image: amouat/logspout-logstash
    volumes:
        - /var/run/docker.sock:/tmp/docker.sock
    ports:
        - "8000:80"
 ```

 Lancer la composition des conteneurs:
 ```bash
 $ docker-compose -f prod-with-logging.yml up -d
 ```

### Problème fréquent

`Get https://registry-1.docker.io/v2/: net/http: request canceled while waiting for connection`


Problème lors du pull des images Docker derrière un proxy.

Créer un fichier de configuration pour le service Docker:

`mkdir /etc/systemd/system/docker.service.d`

Maintenant, on va créer la variable d'environnement du proxy dans un nouveau fichier `/etc/systemd/system/docker.service.d/http-proxy.conf` comme suit:

```
[Service]
Environment="HTTP_PROXY=http://PROXY:PORT/"
```

Remplacer PROXY:PORT par le proxy et le port derrière lequel nous sommes (par exemple 193.56.47.8:8080)



## Monitorer OpenShift

?> Sources: https://blog.openshift.com/openshift-logs-metrics-management-logstash-graphite/
