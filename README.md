# 🌤️ Analyse de Données Météorologiques avec Kafka Streams

Application de traitement de flux en temps réel pour l'analyse de données météorologiques collectées depuis des stations météo via Apache Kafka.

##  Table des Matières

- [Description du Projet](#description-du-projet)
- [Architecture](#architecture)
- [Prérequis](#prérequis)
- [Installation](#installation)
- [Configuration](#configuration)
- [Utilisation](#utilisation)
- [Tests](#tests)
- [Structure du Projet](#structure-du-projet)
- [Technologies Utilisées](#technologies-utilisées)
- [Auteur](#auteur)

---

##  Description du Projet

Une entreprise collecte des données météorologiques en temps réel via Kafka. Chaque station météorologique envoie des messages dans le topic Kafka `weather-data`. Cette application Kafka Streams traite ces données pour :

1. **Filtrer** les températures élevées (> 30°C)
2. **Convertir** les températures de Celsius en Fahrenheit
3. **Agréger** les moyennes par station
4. **Publier** les résultats dans le topic `station-averages`

### Format des Messages d'Entrée

```csv
station,temperature,humidity
```

- **station** : Identifiant de la station (ex: Station1, Station2)
- **temperature** : Température en °C (ex: 25.3)
- **humidity** : Pourcentage d'humidité (ex: 60)

### Format des Messages de Sortie

```
StationX : Température Moyenne = XX.XX°F, Humidité Moyenne = XX.XX%
```

---

##  Architecture

```
<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/c2f442de-8353-4c6d-9753-ae4623865bbf" />

```

---

## 🔧 Prérequis

### Logiciels Requis

- **Java JDK 11** ou supérieur
- **Maven 3.6+**
- **Docker** et **Docker Compose**
- **Git**

### Vérification des Installations

```bash
# Vérifier Java
java -version

# Vérifier Maven
mvn -version

# Vérifier Docker
docker --version
docker-compose --version
```

---

##  Installation

### 1. Cloner le Projet

```bash
git clone <votre-repo>
cd weather-data-streaming
```

### 2. Compiler le Projet

```bash
mvn clean package
```

Le JAR sera créé dans : `target/weather-streams-1.0-SNAPSHOT.jar`

### 3. Démarrer Kafka avec Docker

```bash
# Démarrer Zookeeper et Kafka
docker-compose up -d

# Attendre 30-40 secondes que Kafka démarre
```

### 4. Créer les Topics Kafka

```bash
# Topic d'entrée
docker exec kafka kafka-topics --create \
  --topic weather-data \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1 \
  --if-not-exists

# Topic de sortie
docker exec kafka kafka-topics --create \
  --topic station-averages \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1 \
  --if-not-exists

# Vérifier les topics
docker exec kafka kafka-topics --list --bootstrap-server localhost:9092
```

---

## ⚙️ Configuration

### Variables d'Environnement

L'application utilise les variables d'environnement suivantes (avec valeurs par défaut) :

| Variable | Valeur par Défaut | Description |
|----------|-------------------|-------------|
| `KAFKA_BOOTSTRAP_SERVERS` | `localhost:9092` | Adresse du serveur Kafka |
| `KAFKA_STATE_DIR` | `/tmp/kafka-streams-weather` | Répertoire du state store |

### Exemple de Configuration Personnalisée

**Windows PowerShell :**
```powershell
$env:KAFKA_BOOTSTRAP_SERVERS="kafka-server:9092"
$env:KAFKA_STATE_DIR="D:\kafka-streams-state"
java -jar target\weather-streams-1.0-SNAPSHOT.jar
```

**Linux/Mac :**
```bash
export KAFKA_BOOTSTRAP_SERVERS="kafka-server:9092"
export KAFKA_STATE_DIR="/var/lib/kafka-streams"
java -jar target/weather-streams-1.0-SNAPSHOT.jar
```

---

##  Utilisation

### Démarrage Complet

#### Terminal 1 : Lancer l'Application

```bash
java -jar target/weather-streams-1.0-SNAPSHOT.jar
```

Vous devriez voir :
```
Démarrage de l'application Weather Streams...
Topologie construite:
...
Application Weather Streams démarrée avec succès
```

#### Terminal 2 : Envoyer des Données

```bash
# Démarrer un producteur
docker exec -it kafka kafka-console-producer \
  --topic weather-data \
  --bootstrap-server localhost:9092
```

Puis tapez (ligne par ligne) :
```
Station1,31.5,65
Station2,35.0,70
Station1,32.0,68
Station3,40.0,55
Station2,33.5,72
```

Appuyez sur `Ctrl+C` pour quitter.

#### Terminal 3 : Voir les Résultats

```bash
# Démarrer un consumer
docker exec -it kafka kafka-console-consumer \
  --topic station-averages \
  --bootstrap-server localhost:9092 \
  --from-beginning
```

**Résultats attendus :**
```
Station1 : Température Moyenne = 88.70°F, Humidité Moyenne = 65.00%
Station2 : Température Moyenne = 95.00°F, Humidité Moyenne = 70.00%
Station1 : Température Moyenne = 89.15°F, Humidité Moyenne = 66.50%
Station3 : Température Moyenne = 104.00°F, Humidité Moyenne = 55.00%
Station2 : Température Moyenne = 94.25°F, Humidité Moyenne = 71.00%
```

---

## Tests

### Scénario 1 : Filtrage des Températures

**Données d'entrée :**
```
Station1,25.3,60    # Température ≤ 30°C → FILTRÉ
Station2,35.0,50    # Température > 30°C → ACCEPTÉ
Station3,29.9,55    # Température ≤ 30°C → FILTRÉ
Station2,40.0,45    # Température > 30°C → ACCEPTÉ
```

**Résultat attendu :**
Seules Station2 apparaît dans `station-averages` (Station1 et Station3 sont filtrées).

### Scénario 2 : Conversion Celsius → Fahrenheit

**Formule :** `F = (C × 9/5) + 32`

| Celsius | Fahrenheit |
|---------|------------|
| 30°C    | 86°F       |
| 35°C    | 95°F       |
| 40°C    | 104°F      |

### Scénario 3 : Agrégation par Station

**Données pour Station1 :**
```
Station1,31.0,60  → 87.8°F, 60%
Station1,33.0,70  → 91.4°F, 70%
Station1,35.0,65  → 95.0°F, 65%
```

**Moyenne attendue :**
- Température : (87.8 + 91.4 + 95.0) / 3 = 91.4°F
- Humidité : (60 + 70 + 65) / 3 = 65%

### Test avec Données de Test

Créez un fichier `test-data.txt` :
```
Station1,31.5,65
Station2,35.0,70
Station1,32.0,68
Station3,40.0,55
Station2,33.5,72
Station1,31.0,67
Station3,38.5,58
Station2,34.0,71
```

Envoyez les données :
```bash
# Windows PowerShell
Get-Content test-data.txt | docker exec -i kafka kafka-console-producer --topic weather-data --bootstrap-server localhost:9092

# Linux/Mac
cat test-data.txt | docker exec -i kafka kafka-console-producer --topic weather-data --bootstrap-server localhost:9092
```

---

## 📁 Structure du Projet

```
weather-data-streaming/
├── pom.xml                          # Configuration Maven
├── docker-compose.yml               # Configuration Docker (Kafka + Zookeeper)
├── README.md                        # Ce fichier
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── ma/enset/data/
│   │   │       └── WeatherStreamsApp.java    # Application principale
│   │   └── resources/
│   │       └── logback.xml          # Configuration des logs
│   └── test/
│       └── java/
│           └── ma/enset/data/
│               └── WeatherStreamsAppTest.java # Tests unitaires
├── target/
│   └── weather-streams-1.0-SNAPSHOT.jar      # JAR exécutable
└── logs/
    └── weather-streams.log          # Fichiers de logs
```

---

##  Technologies Utilisées

| Technologie | Version | Usage |
|-------------|---------|-------|
| **Apache Kafka** | 3.5.1 | Plateforme de streaming |
| **Kafka Streams** | 3.5.1 | API de traitement de flux |
| **Java** | 11 | Langage de programmation |
| **Maven** | 3.11.0 | Gestion de build |
| **Jackson** | 2.15.2 | Sérialisation JSON |
| **SLF4J + Logback** | 1.7.36 / 1.2.11 | Logging |
| **Docker** | - | Conteneurisation |
| **Kafka UI** | latest | Interface web pour Kafka |

---
## Les captures d'écrans
<img width="1466" height="535" alt="Capture d&#39;écran 2025-11-26 015053" src="https://github.com/user-attachments/assets/b7c39e4f-e777-4bda-b586-195b3ee2f0db" />

<img width="1474" height="382" alt="Capture d&#39;écran 2025-11-26 015104" src="https://github.com/user-attachments/assets/f1ebbff8-83b8-4adf-9d93-cf523540bcb5" />

<img width="1448" height="522" alt="Capture d&#39;écran 2025-11-26 015110" src="https://github.com/user-attachments/assets/872cb9dc-bc5b-4186-bc16-f08bbb4fc177" />

<img width="1918" height="611" alt="Capture d&#39;écran 2025-11-26 020113" src="https://github.com/user-attachments/assets/bbc59b97-2d93-4453-a604-a9f287b73928" />

<img width="1913" height="866" alt="Capture d&#39;écran 2025-11-26 020125" src="https://github.com/user-attachments/assets/651c91b5-6c05-4a99-a74d-c9e74d50a6f8" />

<img width="1898" height="763" alt="Capture d&#39;écran 2025-11-26 020142" src="https://github.com/user-attachments/assets/9984c53f-f1b3-46fb-8954-6f53ae95dae6" />

<img width="1902" height="676" alt="Capture d&#39;écran 2025-11-26 020304" src="https://github.com/user-attachments/assets/c8c72958-a25f-45c3-a646-dddfd72a290e" />

<img width="1649" height="819" alt="Capture d&#39;écran 2025-11-26 020419" src="https://github.com/user-attachments/assets/2057b3a4-1bea-4c1d-beb8-8d43d74559f3" />

<img width="850" height="879" alt="Capture d&#39;écran 2025-11-26 020449" src="https://github.com/user-attachments/assets/1e62429a-27ad-4e37-a0f0-f1ad7ea1d3b7" />

<img width="1876" height="761" alt="Capture d&#39;écran 2025-11-26 020838" src="https://github.com/user-attachments/assets/517edbd6-4839-4862-b27a-dc0eaf73d48c" />

---
##  Interface Web Kafka UI

L'application inclut Kafka UI pour une visualisation en temps réel.

**URL :** http://localhost:8090

### Fonctionnalités :

- 📋 **Topics** : Voir tous les topics et leurs messages
- 👥 **Consumers** : Monitorer les consumer groups et leur LAG
- 📊 **Statistics** : Graphiques de performance
- 📤 **Produce** : Envoyer des messages directement depuis l'interface
- 🔍 **Search** : Rechercher dans les messages

---

##  Commandes Utiles

### Gestion de Kafka

```bash
# Lister les topics
docker exec kafka kafka-topics --list --bootstrap-server localhost:9092

# Décrire un topic
docker exec kafka kafka-topics --describe --topic weather-data --bootstrap-server localhost:9092

# Voir les consumer groups
docker exec kafka kafka-consumer-groups --list --bootstrap-server localhost:9092

# Détails d'un consumer group
docker exec kafka kafka-consumer-groups --describe --group weather-streams-app --bootstrap-server localhost:9092

# Réinitialiser les offsets (application arrêtée)
docker exec kafka kafka-consumer-groups --reset-offsets \
  --group weather-streams-app \
  --all-topics \
  --to-earliest \
  --bootstrap-server localhost:9092 \
  --execute
```

### Gestion de Docker

```bash
# Voir les conteneurs actifs
docker ps

# Voir les logs
docker logs kafka
docker logs zookeeper
docker logs kafka-ui

# Arrêter tout
docker-compose down

# Arrêter et supprimer les volumes (nettoie les données)
docker-compose down -v

# Redémarrer un service
docker-compose restart kafka
```

### Nettoyage

```bash
# Supprimer le state store local
rm -rf /tmp/kafka-streams-weather  # Linux/Mac
Remove-Item -Recurse -Force C:\tmp\kafka-streams-weather  # Windows

# Nettoyer Docker
docker system prune -f
```

---

##  Dépannage

### Problème : Application ne démarre pas

**Solution :**
```bash
# Vérifier que Kafka est en cours d'exécution
docker ps

# Vérifier les logs Kafka
docker logs kafka

# Vérifier la connexion
docker exec kafka kafka-broker-api-versions --bootstrap-server localhost:9092
```

### Problème : Pas de résultats dans station-averages

**Vérifications :**
1. L'application Java est-elle en cours d'exécution ?
2. Les températures sont-elles > 30°C ?
3. Le format CSV est-il correct ?

**Debug :**
```bash
# Voir les données brutes dans weather-data
docker exec kafka kafka-console-consumer --topic weather-data --bootstrap-server localhost:9092 --from-beginning --max-messages 10

# Vérifier le consumer group
docker exec kafka kafka-consumer-groups --describe --group weather-streams-app --bootstrap-server localhost:9092
```

### Problème : Kafka UI ne charge pas

**Solution :**
```bash
# Vérifier les logs
docker logs kafka-ui

# Redémarrer Kafka UI
docker-compose restart kafka-ui

# Vérifier la connexion réseau
docker exec kafka-ui ping kafka -c 3
```

---

##  Performance et Scalabilité

### Configuration de Production

Pour la production, modifiez les propriétés Kafka Streams :

```java
props.put(StreamsConfig.NUM_STREAM_THREADS_CONFIG, 4);
props.put(StreamsConfig.PROCESSING_GUARANTEE_CONFIG, "exactly_once_v2");
props.put(StreamsConfig.REPLICATION_FACTOR_CONFIG, 3);
props.put(StreamsConfig.CACHE_MAX_BYTES_BUFFERING_CONFIG, 10 * 1024 * 1024);
```

### Monitoring

Activez JMX pour monitorer les métriques Kafka Streams :

```bash
java -Dcom.sun.management.jmxremote \
     -Dcom.sun.management.jmxremote.port=9999 \
     -Dcom.sun.management.jmxremote.authenticate=false \
     -Dcom.sun.management.jmxremote.ssl=false \
     -jar target/weather-streams-1.0-SNAPSHOT.jar
```

---

## Logs

### Emplacement des Logs

- **Console** : Logs en temps réel
- **Fichier** : `logs/weather-streams.log`

### Niveaux de Log

- **DEBUG** : Détails de traitement (ma.enset.data)
- **INFO** : Événements importants
- **WARN** : Avertissements
- **ERROR** : Erreurs critiques

### Configuration

Modifiez `src/main/resources/logback.xml` pour ajuster les niveaux de log.

---

## 🎓 Concepts Kafka Streams Utilisés

### KStream
Stream de données en temps réel, chaque événement est traité indépendamment.

### KGroupedStream
Stream groupé par clé (station dans notre cas).

### KTable
Table de résultats agrégés, représente l'état actuel des données.

### State Store
Stockage local pour maintenir l'état des agrégations.

### Serialization/Deserialization
Conversion JSON personnalisée pour les objets Java.

---

## 📚 Références

- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [Kafka Streams Documentation](https://kafka.apache.org/documentation/streams/)
- [Confluent Kafka Tutorials](https://kafka-tutorials.confluent.io/)

---

## 👨‍💻 Auteur

**Hajar Elfallaki-idrissi**
---

## 📄 Licence

Ce projet est réalisé dans le cadre d'un exercice académique.

---

##  Résumé des Commandes Essentielles

```bash
# 1. Démarrer Kafka
docker-compose up -d

# 2. Créer les topics
docker exec kafka kafka-topics --create --topic weather-data --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
docker exec kafka kafka-topics --create --topic station-averages --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1

# 3. Lancer l'application
java -jar target/weather-streams-1.0-SNAPSHOT.jar

# 4. Envoyer des données
docker exec -it kafka kafka-console-producer --topic weather-data --bootstrap-server localhost:9092

# 5. Voir les résultats
docker exec -it kafka kafka-console-consumer --topic station-averages --bootstrap-server localhost:9092 --from-beginning

# 6. Kafka UI
http://localhost:8090

# 7. Arrêter tout
docker-compose down
```

---

**✨ Bon traitement de données météorologiques ! ✨**
