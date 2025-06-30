## Rapport d’analyse réseau avec la stack ELK et Packetbeat

---

### 1. Environnement construit

#### Technologie utilisée :

* **Docker** pour la virtualisation légère (conteneurs)
* **Docker Compose** pour orchestrer plusieurs services (ELK stack + Packetbeat)

#### Machine hôte :

* OS : Windows 11 avec **WSL2**
* Docker Desktop installé

#### Conteneurs :

* `elasticsearch` (indexation et recherche)
* `kibana` (interface graphique)
* `packetbeat` (analyse de trafic réseau depuis un fichier `.pcap`)


### 2. Processus d’installation de la stack

#### Structure du projet :

```
Manipulating-ELK-Stack/
├── docker-compose.yml
├── dockerfile.packetbeat
├── packetbeat/
│   └── packetbeat.yml
├── pcap/
│   └── 4SICS-GeekLounge-151022.pcap
```

#### Commandes utilisées :

```bash
# Cloner le projet ou créer les fichiers nécessaires
cd Manipulating-ELK-Stack/

# Lancer tous les services
docker-compose up --build

# Vérifier les conteneurs
docker ps

# Accéder au shell d’un conteneur pour debug
docker exec -it packetbeat bash
```


### 3. Parsing et indexation du fichier PCAP

#### Fonctionnement :

Packetbeat lit un fichier `.pcap`, extrait les paquets réseau et les indexe dans Elasticsearch. Chaque flux ou protocole reconnu est enrichi (source, destination, durée, etc.).

#### Configuration du parsing dans `packetbeat.yml` :

```yaml
packetbeat.pcap:
  file: "/pcap/4SICS-GeekLounge-151022.pcap"
  loop: false
  snaplen: 65535
  buffer_size_mb: 100

output.elasticsearch:
  hosts: ["http://elasticsearch:9200"]

setup.kibana:
  host: "http://kibana:5601"
```

#### 🛠️ Commande clé :

```bash
docker exec -it packetbeat bash
packetbeat -e -c /etc/packetbeat/packetbeat.yml -path.data /tmp/beatdata -d "*"
```

---

### 4. Architecture et Workflow

#### Architecture :

```
        +---------------------+
        |     Machine Hôte   |
        |  (Docker Desktop)  |
        +---------------------+
          |         |         |
          ↓         ↓         ↓
     +----------+ +----------+ +-----------+
     |Elasticsea| |  Kibana  | | Packetbeat|
     |rch       | |          | |           |
     +----------+ +----------+ +-----------+
                          ↑
                      PCAP File
```

#### Workflow de traitement :

```
[PCAP File] → [Packetbeat] → [Elasticsearch] → [Kibana Dashboard]
```

---

### 5. Fichier PCAP utilisé

* Nom : `4SICS-GeekLounge-151022.pcap`
* Taille : environ 200 Mo
* Source : trafic industriel SCADA/ICS issu d’un environnement simulé

---

### 6. Résultats dans Kibana

#### Index détecté :

* `packetbeat-*`


### Conclusion

Cet environnement a permis de :

* Déployer une stack ELK containerisée et reproductible
* Rejouer un fichier `.pcap` en boucle ou une seule fois
* Analyser les protocoles, les ports et les hôtes du trafic contenu
* Visualiser dynamiquement les événements réseau avec Kibana

