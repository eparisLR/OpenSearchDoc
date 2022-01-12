# Tips OpenSearch

## Docker-compose

```
version: '3'
services:
  opensearch-node1:
    image: opensearchproject/opensearch:latest
    container_name: opensearch-node1
    environment:
      - discovery.type=single-node
      - bootstrap.memory_lock=true 
      - "OPENSEARCH_JAVA_OPTS=-Xms2048m -Xmx2048m" 
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536 # maximum number of open files for the OpenSearch user, set to at least 65536 on modern systems
        hard: 65536
    volumes:
      - opensearch-data1:/usr/share/opensearch/data
    ports:
      - 9200:9200
      - 9600:9600 # perf analyzer
    expose:
      - "9200"
    networks:
      - opensearch-net
  opensearch-dashboards:
    image: opensearchproject/opensearch-dashboards:latest
    container_name: opensearch-dashboards
    ports:
      - 5601:5601
    expose:
      - "5601"
    environment:
      OPENSEARCH_HOSTS: '["https://opensearch-node1:9200"]'
    networks:
      - opensearch-net

volumes:
  opensearch-data1:

networks:
  opensearch-net:

```

## Installation

Pour lancer une instance d'OpenSearch, il faut up le docker-compose.

Ensuite il faut s'y connecter une première fois :
Soit avec CURL : 
```
curl -u admin:admin --insecure -XGET https://localhost:9200
```

Soit à l'Url : https://localhost:9200

Pour créer un index : 
```
curl -u admin:admin --insecure -XPUT "https://localhost:9200/{index}?pretty"
```

Pour importer un document :
```
curl -u admin:admin --insecure -XPUT "https://localhost:9200/movies/_doc/1?pretty" -H 'Content-Type: application/json' -d '{document}'
```

## Accéder à la données

Pour accéder à ce document :
```
https://localhost:9200/movies/_doc/1?pretty
```

Pour accéder aux informations de l'index que nous avons créé :
```
https://localhost:9200/movies?pretty
```

Remarque : On peut voir qu'OpenSearch a défini la structure de données d'un film et de chaque champ

## Configuration

Pour définir le mapping manuellement :
```
curl -u admin:admin --insecure -XPUT "https://localhost:9200/movies/_mapping?pretty" -H 'Content-Type: application/json' -d @mapping_refactored.json
```

Pour vérifier que le mapping est bien effectué : 
```
https://localhost:9200/movies/_mapping?pretty
```


Pour importer des données dans l'index : 
```
curl -u admin:admin --insecure -XPUT https://localhost:9200/_bulk -H "Content-Type: application/json" --data-binary @movies_refactored.json
```

## Query

Pour accéder à l'URL de recherche :
```
curl -u admin:admin --insecure -XGET https://localhost:9200/_search?pretty
```

Pour faire une query via l'URL :
```
https://localhost:9200/_search?q={param}
```

Pour faire une query via l'URL approximativement :
```
https://localhost:9200/_search?q={param}~{number}
```

Pour faire une query via l'URL sur un champ :
```
https://localhost:9200/_search?q={param}:{param}
```

Pour faire une recherche multi critères :
```
https://localhost:9200/_search?q={param} {param}
```

Remarque : Par défaut lors d'une recherche multi critères OpenSearch cherche l'un ou l'autre.
Pour y remédier :

```
https://localhost:9200/_search?q="{param} {param}"
```

Pour faire une recherche avec deux conditions :
```
https://localhost:9200/_search?q=title:Batman AND directors: "Tim Burton"
```

Pour faire une query avec conditions mais en retirant ceux qui contiennent un certain mot :
```
https://localhost:9200/_search?q=title:Batman AND directors: "Tim Burton" -plot:"joker"
```

Pour faire une query en donnant plus d'importance à un critère :
```
https://localhost:9200/_search?q=title:Batman OR% plot:"penguin"^10
```

# Le Dashboard

Pour y accéder : http://localhost:5601

## Création d'index

Aller dans le menu à gauche puis Discover.
Ensuite créer un index qui correspond soit à un index que nous avons créé.

Par exemple si l'on a un index movies, on peut créer un index mov* qui pourra contenir plusieurs indexs contenant des movies.

Ensuite il faut sélectionner un champ temporel

## Visualisation

Retournez sur le menu discover et changer la période pour afficher les datas.
Par défaut il affiche les datas des 15 dernières minutes.