# Jakarta Intro

Jakarta EE (anciennement Java EE, puis JEE) est une plateforme pour le développement d'applications d'entreprise en Java. Elle fournit un ensemble d'APIs et de spécifications permettant de créer des applications évolutives, sécurisées et robustes, notamment pour les environnements cloud et microservices.

### Principales caractéristiques de Jakarta EE
1. **Développement d’applications d’entreprise**

Permet de créer des applications web, distribuées et transactionnelles.

2. **Composants clés**

- Jakarta Servlet (ex-Java Servlet) : gestion des requêtes HTTP.
- Jakarta RESTful Web Services (JAX-RS) : création d'APIs RESTful.
- Jakarta Persistence (JPA) : accès aux bases de données via ORM.
- Jakarta Faces (JSF) : framework MVC pour applications web.
- Jakarta Transactions (JTA) : gestion des transactions distribuées.
- Jakarta CDI : injection de dépendances et gestion du cycle de vie.

3. **Interopérabilité et Standardisation**

Basé sur des standards ouverts, facilitant la portabilité entre serveurs d'applications (WildFly, Payara, TomEE, OpenLiberty, etc.).

4. **Support Cloud et Microservices**

Compatible avec MicroProfile, qui ajoute des fonctionnalités comme la configuration dynamique et l’observabilité.

5. **Évolution et Open Source**

Depuis 2018, Jakarta EE est géré par Eclipse Foundation, après son transfert depuis Oracle.

## Générer un projet Jakarta

```shell
mvn archetype:generate \
  -DgroupId=fr.formation.jakarta \
  -DartifactId=jakarta-app \
  -DarchetypeArtifactId=maven-archetype-webapp \
  -DinteractiveMode=false
```

Dans le dossier `jakarta-app`, ajouter la dépendance au fichier `pom.xml`

```xml
<dependencies>
    <!-- Jakarta EE API -->
    <dependency>
        <groupId>jakarta.platform</groupId>
        <artifactId>jakarta.jakartaee-web-api</artifactId>
        <version>10.0.0</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>jakarta.servlet</groupId>
        <artifactId>jakarta.servlet-api</artifactId>
        <version>6.0.0</version> <!-- Jakarta EE 10 utilise Servlet 6.0 -->
        <scope>provided</scope>
    </dependency>
</dependencies>
```

## Les servlets

Les servlets sont des classes Java utilisées pour recevoir des requêtes HTTP et renvoyer des réponses.

```Java
package fr.formation.web;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(
        HttpServletRequest req, 
        HttpServletResponse resp
    ) throws ServletException, IOException 
    {
        resp.setContentType("text/html");
        resp.getWriter().write("<h1>Hello Jakarta EE!</h1>");
    }
}
```

- Le servlet hérite de `HttpServlet`.
- La route est spécifiée dans l'annotation `@WebServlet`
- La méthode `doGet` gère les requêtes HTTP GET sur cette route et retourne une réponse HTTP.
  - Cette méthode admet en argument deux objets modélisant la requête (`HttpServletRequest`) et la réponse (`HttpServletResponse`) HTTP.
  - `response.setContentType("text/html")` : Définit le type de réponse.
  - `PrintWriter out = response.getWriter()` : Permet d'écrire la réponse.

### Déploiement d'un servlet

Pour déployer l'application, il faut tout d'abord la compiler et l'empaqueter dans un fichier `.war` (web archive). 

```shell
mvn clean package
```

## Les serveurs d'application

Jakarta EE repose sur des serveurs d’applications compatibles qui implémentent ses spécifications. En fait il y en a deux : 
- `web profile`, plus légère 
- `full profile`, plus complète

### Web profile
**Cible** : Destiné aux applications web légères.

**Contenu** : Un sous-ensemble des spécifications Jakarta EE, suffisant pour les applications web classiques.

**Principaux composants inclus :**

- Servlet (Jakarta Servlet)
- JSP & EL (Jakarta Server Pages & Expression Language)
- JSF (Jakarta Faces)
- CDI (Contexts and Dependency Injection)
- JAX-RS (API RESTful)
- Jakarta Transactions (JTA) (gestion des transactions)
- Jakarta Security (authentification et autorisation)

**✅ Avantages :**

- Plus léger et plus rapide que le Full Profile.
- Moins de dépendances inutiles si ton application est uniquement web.
- Démarrage plus rapide des serveurs.

**❌ Inconvénients :**

- Pas de support natif pour EJB (Enterprise JavaBeans).
- Pas de JMS (Java Message Service) pour la messagerie asynchrone.
- Pas de Jakarta Batch ni de Jakarta Concurrency pour les traitements en arrière-plan.

### Full profile

**Cible** : Destiné aux applications d'entreprise complexes.

**Contenu** : Inclut toutes les spécifications de Jakarta EE.

**En plus du Web Profile, il inclut :**

- EJB (Enterprise JavaBeans) (gestion de la logique métier transactionnelle)
- Jakarta Messaging (JMS) (messagerie asynchrone)
- Jakarta Batch (traitement de lots)
- Jakarta Concurrency (gestion des threads)
- Jakarta Connector (JCA) (intégration avec des systèmes externes)
- Jakarta Persistence (JPA) (gestion des bases de données avec ORM)

**✅ Avantages :**

- Plus de fonctionnalités pour des applications complexes et distribuées.
- Supporte la messagerie asynchrone et les transactions avancées.
- Idéal pour les applications nécessitant JPA, EJB, et JMS.

**❌ Inconvénients :**

- Plus lourd que le Web Profile.
- Temps de démarrage plus long et plus gourmand en mémoire.

#### Liste des serveurs Jakarta EE

| Serveur       | Full Profile | Web Profile | Microservices      | Support commercial |
|---------------|--------------|-------------|--------------------|--------------------|
| WildFly       | ✅            | ✅           | ⚠️ (via Thorntail) | Red Hat            |
| Payara Server | ✅            | ✅           | ✅ (Payara Micro)   | Payara             |
| GlassFish     | ✅            | ✅           | ❌                  | Non officiel       |
| Open Liberty  | ✅            | ✅           | ✅                  | IBM                |
| TomEE         | ⚠️ (partiel) | ✅           | ✅                  | Apache             |
| Helidon       | ❌            | ❌           | ✅                  | Oracle             |
| Quarkus       | ❌            | ❌           | ✅                  | Red Hat            |


### Un serveur léger pour le dev

Pour tester une application web simple (sans JMS ni JPA), il existe une solution très légère, `Payara micro`.

[Télécharger Payara](https://www.payara.fish/downloads/payara-platform-community-edition/)

Ce serveur est distribué sous la forme d'un fichier `.jar` et le déploiement s'effectue en ligne de commande.

```shell
java -jar payara-micro.jar --deploy target/jakarta-app.war --contextRoot /
```

Tester l'url suivante :

[](http://localhost:8080)

> Par défaut Payara déploie l'application sur une route portant le nom du fichier `.war`. 
> Pour déployer à la racine du serveur il faut ajouter le paramètre `--contextRoot /`.

#### Script de déploiement

**Mac OS et Linux**

```shell
#!/bin/bash

mvn clean package
pkill -f payara-micro
sleep 2
java -jar payara-micro.jar --deploymentDir ./target --contextRoot / --port 8080
```

**Windows**

```
@echo off

mvn clean package

for /f "tokens=5" %%i in ('netstat -aon ^| findstr :8181') do taskkill /F /PID %%i

timeout /t 2 /nobreak > nul

java -jar payara-micro.jar --deploymentDir ./target --contextRoot / --port 8181
```

<!--

### Un serveur complet

Paraya micro est intéressant pour du déploiement léger, mais il n'implémente que la spécification `web profile`.
Pour travailler avec un ORM par exemple, il faut une implémentation de `full profile`. 
Le plus simple, en phase de développement, est d'utiliser Docker pour instancier une image de ce serveur dans un conteneur.

```yaml
services:
  payara:
    image: payara/server-full
    container_name: payara-server
    ports:
      - "8080:8080"     # HTTP port
      - "4848:4848"     # Admin Console
    environment:
      PAYARA_PASSWORD: admin
      PAYARA_DOMAIN: domain1
      # Limites basses et hautes de la consommation de mémoire
      JVM_ARGS: '-Xms512m -Xmx1024m'
    volumes:
      - ./target:/opt/payara/deployments   # Dossier de déploiement
      - ./payara-config:/opt/payara/config  # Dossier de configuration
    restart: unless-stopped

```
-->