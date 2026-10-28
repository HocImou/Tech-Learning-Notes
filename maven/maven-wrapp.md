# Maven Wrapper : comprendre et utiliser `mvnw`

## 1. Qu'est-ce que Maven Wrapper ?

Maven Wrapper est un mécanisme qui permet à un projet Java de fournir ses propres scripts de lancement de Maven.

Au lieu de demander à chaque développeur d'installer Maven et de gérer sa version manuellement, le projet définit la version de Maven à utiliser.

Le Wrapper télécharge automatiquement cette version si elle n'est pas déjà disponible localement.

**Objectif :** garantir que les développeurs et les pipelines CI/CD utilisent une version cohérente de Maven.

## 2. Maven classique vs Maven Wrapper

### Maven classique

```bash
mvn clean verify
```

Cette commande utilise l'installation de Maven disponible sur la machine.

Pour connaître la version utilisée :

```bash
mvn -version
```

Si deux développeurs possèdent des versions différentes de Maven, ils peuvent rencontrer des différences de comportement.

### Maven Wrapper

Sous Windows PowerShell :

```powershell
.\mvnw.cmd clean verify
```

Sous Linux ou macOS :

```bash
./mvnw clean verify
```

Le Wrapper utilise la distribution Maven définie dans le projet. Si nécessaire, il la télécharge automatiquement.

Il n'est donc pas nécessaire d'installer Maven séparément pour utiliser le Wrapper, mais un accès réseau peut être requis lors du premier lancement.

## 3. Les fichiers du Maven Wrapper

Un projet utilisant Maven Wrapper contient généralement les fichiers suivants :

```text
mon-projet/
├── .mvn/
│   └── wrapper/
│       └── maven-wrapper.properties
├── mvnw
├── mvnw.cmd
└── pom.xml
```

| Fichier | Rôle |
|---|---|
| `mvnw` | Script de lancement pour Linux et macOS |
| `mvnw.cmd` | Script de lancement pour Windows |
| `maven-wrapper.properties` | Configuration de la distribution Maven |
| `pom.xml` | Configuration du projet Maven : dépendances, plugins, compilation, tests, etc. |

Le fichier `maven-wrapper.properties` contient notamment la propriété `distributionUrl`, qui indique la distribution Maven à utiliser.

Exemple simplifié :

```properties
distributionUrl=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.9.9/apache-maven-3.9.9-bin.zip
```

La version ci-dessus est illustrative : la valeur réelle dépend de la configuration du projet.

## 4. Comprendre une commande Maven Wrapper

Exemple :

```powershell
.\mvnw.cmd clean verify
```

Décomposition :

- `.\mvnw.cmd` : exécute le script Maven Wrapper sous Windows.
- `clean` : supprime les fichiers générés lors des builds précédents, notamment dans `target/`.
- `verify` : exécute les phases précédentes du cycle Maven, dont la compilation et les tests, puis les vérifications configurées jusqu'à cette phase.

Autre exemple :

```powershell
.\mvnw.cmd spring-boot:run
```

- `spring-boot:run` : demande au plugin Maven de Spring Boot de lancer l'application.
- Le Wrapper fournit la version de Maven, tandis que le plugin Spring Boot fournit l'objectif `run`.

**À retenir :** Maven Wrapper ne remplace ni le `pom.xml`, ni les plugins Maven. Il facilite simplement l'utilisation d'une version déterminée de Maven.

## 5. Pourquoi utiliser Maven Wrapper en équipe ?

### Cohérence des environnements

Tous les développeurs utilisent la version de Maven définie par le projet.

### Intégration continue (CI/CD)

Dans un pipeline GitLab CI/CD, le Wrapper permet d'exécuter les builds sans dépendre de la version de Maven installée globalement sur le runner.

Exemple :

```yaml
build:
  script:
    - ./mvnw clean verify
```

Cet exemple suppose un runner Linux disposant d'un JDK compatible avec le projet et des outils nécessaires.

### Onboarding des développeurs

Un nouveau développeur peut compiler et tester le projet sans devoir installer et configurer Maven manuellement.

### Reproductibilité améliorée

Le Wrapper réduit les différences liées à la version de Maven. Il ne garantit toutefois pas, à lui seul, une build entièrement reproductible : il faut aussi maîtriser le JDK, les dépendances, les plugins et les autres éléments de l'environnement.

## 6. Bonnes pratiques

- Versionner `mvnw`, `mvnw.cmd` et les fichiers nécessaires dans `.mvn/wrapper/`.
- Utiliser le Wrapper dans la documentation du projet et les pipelines CI/CD.
- Vérifier la version utilisée avec `.\mvnw.cmd -version` sous Windows ou `./mvnw -version` sous Linux.
- Choisir une version de Maven compatible avec le projet et la conserver de manière cohérente.
- Ne pas modifier arbitrairement `distributionUrl` sans vérifier la version et la provenance de la distribution.
- Vérifier que le JDK utilisé est compatible avec la version de Java du projet.

## 7. Commandes utiles

| Besoin | Windows PowerShell | Linux/macOS |
|---|---|---|
| Afficher la version de Maven | `.\mvnw.cmd -version` | `./mvnw -version` |
| Nettoyer et vérifier le projet | `.\mvnw.cmd clean verify` | `./mvnw clean verify` |
| Lancer une application Spring Boot | `.\mvnw.cmd spring-boot:run` | `./mvnw spring-boot:run` |

## 8. Conclusion

Maven Wrapper permet de standardiser la version de Maven utilisée par un projet et simplifie son utilisation par les développeurs comme par les pipelines CI/CD.

**Règle pratique :** lorsqu'un projet contient Maven Wrapper, privilégier `./mvnw` sous Linux/macOS ou `.\mvnw.cmd` sous Windows, plutôt que de dépendre de la commande globale `mvn`.

### Références officielles

- Maven Wrapper : https://maven.apache.org/wrapper/
- Maven Build Lifecycle : https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html
- Spring Boot Maven Plugin : https://docs.spring.io/spring-boot/maven-plugin/