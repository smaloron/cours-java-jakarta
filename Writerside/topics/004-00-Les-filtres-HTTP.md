# Les filtres HTTP

Dans Jakarta, un filtre web ou filtre HTTP est une classe 
qui intercepte la requête ou la réponse.
Elle agit donc avant ou après la résolution de la route.

**Schéma de l'exécution des filtres**
```
[Client HTTP]
     |
     v
[Filtre(s)] ---> peut bloquer ou modifier la requête
     |
     v
[Servlet / JSP / Static file]
     |
     v
[Filtre(s)] ---> peut modifier la réponse
     |
     v
[Client HTTP]
```

### Quelques cas d'utilisation fréquents

- Authentifier ou autoriser l’accès à des ressources

- Compresser les réponses (gzip)

- Gérer les entêtes CORS

- Logger les accès

- Modifier les requêtes/réponses

- Ajouter des en-têtes HTTP

- Compter les visites

## La syntaxe des filtres

Les filtres sont définis par l'annotation `@WebFilter` et doivent implémenter l'interface `Filter`
du package `Jakarta.Servlet`.

### Un filtre de requête simple

Ce filtre ajoute un attribut `role` à toutes les routes qui commencent par `/dashboard/`.
Attention, l'instruction `chain.doFilter(request, response);` est importante, sans elle la requête restera bloquée 
sur le filtre et la route ne pourra être résolue.

```java
package fr.formation.jakarta.filters;

import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletRequest;
import java.io.IOException;

@WebFilter(urlPatterns = {"/dashboard/*"})
public class RoleInjectFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain)
            throws IOException, ServletException {

        // On cast vers HttpServletRequest pour accéder aux méthodes HTTP
        HttpServletRequest httpRequest = (HttpServletRequest) request;

        // Injection d'un attribut dans la requête
        httpRequest.setAttribute("userRole", "admin");

        // On passe la requête modifiée à la suite de la chaîne
        chain.doFilter(request, response);
    }
}

```

### Un filtre qui modifie la réponse HTTP

```java
package fr.formation.jakarta.filters;

import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebFilter(urlPatterns = {"/*"}) // Toutes les URL
public class HeaderInjectionFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain)
                         throws IOException, ServletException {

        // On cast en HttpServletResponse
        HttpServletResponse httpResp = (HttpServletResponse) response;

        // Ajout d’un en-tête personnalisé
        httpResp.setHeader("X-App-Name", "MySuperApp");

        // Poursuite de la chaîne
        chain.doFilter(request, response);
    }
}
```

### Les attributs de @WebFilter

| Attribut           | Type               | Description                                         |
|--------------------|--------------------|-----------------------------------------------------|
| `filterName`       | `String`           | Nom du filtre (facultatif).                         |
| `urlPatterns`      | `String[]`         | URLs ciblées (ex. `/admin/*`).                      |
| `value`            | `String[]`         | Alias de `urlPatterns` (à ne pas combiner).         |
| `servletNames`     | `String[]`         | Noms de servlets ciblées.                           |
| `dispatcherTypes`  | `DispatcherType[]` | Types d’interception (voir ci-dessous).             |
| `initParams`       | `WebInitParam[]`   | Paramètres d'initialisation.                        |

**Les types de dispatch**

| Type      | Quand il s’applique                            |
|-----------|------------------------------------------------|
| `REQUEST` | Requête HTTP classique.                        |
| `FORWARD` | Via `RequestDispatcher.forward()`.             |
| `INCLUDE` | Via `RequestDispatcher.include()`.             |
| `ERROR`   | Lors d’une redirection vers une page d’erreur. |
| `ASYNC`   | Lors d’un traitement asynchrone.               |

## Quelques utilisations standards

### Test de l'authentification

```java
package fr.formation.jakarta.filters;

import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;
import java.io.IOException;

// Sécurise toutes les routes /admin
@WebFilter(urlPatterns = {"/admin/*"})
public class AuthFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain)
                         throws IOException, ServletException {

        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse res = (HttpServletResponse) response;

        // On récupère la session sans la créer
        HttpSession session = req.getSession(false);

        boolean isLoggedIn = (session != null &&
                              session.getAttribute("user") != null);

        if (isLoggedIn) {
            // L'utilisateur est connecté : on continue
            chain.doFilter(request, response);
        } else {
            // L'utilisateur n'est pas connecté : redirection
            res.sendRedirect(req.getContextPath() + "/login");
        }
    }
}
```

> Sur le même principe, il est possible de tester la propriété `role` de `User` dans un filtre d'autorisation.

### Injection de CORS

```java
package fr.formation.jakarta.filters;

import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebFilter("/*") // Appliquer à toutes les requêtes
public class CorsFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain)
                         throws IOException, ServletException {

        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse res = (HttpServletResponse) response;

        // ⚙️ En-têtes CORS standards
        res.setHeader("Access-Control-Allow-Origin", "*");
        res.setHeader("Access-Control-Allow-Methods",
                      "GET, POST, PUT, DELETE, OPTIONS");
        res.setHeader("Access-Control-Allow-Headers",
                      "Content-Type, Authorization");
        res.setHeader("Access-Control-Max-Age", "3600");

        // Réponse directe aux requêtes OPTIONS (pré-vol CORS)
        if ("OPTIONS".equalsIgnoreCase(req.getMethod())) {
            res.setStatus(HttpServletResponse.SC_OK);
            return; // Pas besoin de passer au filtre suivant
        }

        // Requête normale : on continue
        chain.doFilter(request, response);
    }
}
```

### Mode maintenance

```java
package fr.formation.jakarta.filters;

import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.annotation.WebInitParam;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebFilter(
    urlPatterns = {"/*"},
    initParams = {
        @WebInitParam(name = "maintenance", value = "true") // toggle ici
    }
)
public class MaintenanceFilter implements Filter {

    private boolean maintenanceMode = false;

    @Override
    public void init(FilterConfig filterConfig) {
        String param = filterConfig.getInitParameter("maintenance");
        maintenanceMode = Boolean.parseBoolean(param);
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain)
                         throws IOException, ServletException {

        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse res = (HttpServletResponse) response;

        String path = req.getRequestURI();
        String ip = req.getRemoteAddr();

        // URLs autorisées même en maintenance
        boolean isAllowedPath = path.contains("/admin") || path.contains("/login");
        boolean isTrustedIP = ip.equals("127.0.0.1"); // Dev local autorisé

        if (maintenanceMode && !isAllowedPath && !isTrustedIP) {
            // Réponse personnalisée
            res.setContentType("text/html");
            res.setStatus(HttpServletResponse.SC_SERVICE_UNAVAILABLE);
            res.getWriter().write(
                "<h1>Maintenance en cours</h1>" +
                "<p>Le site est temporairement indisponible.</p>"
            );
        } else {
            // Requête normale
            chain.doFilter(request, response);
        }
    }
}
```

#### Variante avec une variable d'environnement`


```shell
export MAINTENANCE_MODE=true
```

```java
package fr.formation.jakarta.filters;

import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebFilter("/*")
public class MaintenanceFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain)
                         throws IOException, ServletException {

        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse res = (HttpServletResponse) response;

        String path = req.getRequestURI();
        String ip = req.getRemoteAddr();

        // Lecture dynamique de la variable d'environnement
        String mode = System.getenv("MAINTENANCE_MODE");
        boolean maintenance = "true".equalsIgnoreCase(mode);

        boolean isAllowedPath =
            path.contains("/admin") || path.contains("/login");

        boolean isTrustedIP = ip.equals("127.0.0.1");

        if (maintenance && !isAllowedPath && !isTrustedIP) {
            res.setContentType("text/html");
            res.setStatus(HttpServletResponse.SC_SERVICE_UNAVAILABLE);
            res.getWriter().write(
                "<h1>Maintenance en cours</h1>" +
                "<p>Le site sera bientôt de retour.</p>"
            );
        } else {
            chain.doFilter(request, response);
        }
    }
}
```

#### Variante avec un fichier properties

```java
package fr.formation.jakarta.filters;

import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.FileInputStream;
import java.io.IOException;
import java.util.Properties;

@WebFilter("/*")
public class MaintenanceFilterProperties implements Filter {

    private boolean maintenanceMode = false;

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        try {
            String path = filterConfig.getServletContext()
                .getRealPath("/WEB-INF/config/maintenance.properties");

            Properties props = new Properties();
            props.load(new FileInputStream(path));

            maintenanceMode = "true".equalsIgnoreCase(
                props.getProperty("maintenance", "false")
            );
        } catch (IOException e) {
            throw new ServletException("Impossible de lire le fichier de configuration", e);
        }
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain)
                         throws IOException, ServletException {

        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse res = (HttpServletResponse) response;

        String path = req.getRequestURI();
        String ip = req.getRemoteAddr();

        boolean isAllowedPath = path.contains("/admin") || path.contains("/login");
        boolean isTrustedIP = ip.equals("127.0.0.1");

        if (maintenanceMode && !isAllowedPath && !isTrustedIP) {
            res.setContentType("text/html");
            res.setStatus(HttpServletResponse.SC_SERVICE_UNAVAILABLE);
            res.getWriter().write("<h1>Site en maintenance</h1>");
        } else {
            chain.doFilter(request, response);
        }
    }
}
```

### La gestion des exceptions

Cette classe intercepte les exceptions levées par les Servlets. Elle permet donc de centraliser la gestion des erreurs.


```java
package fr.formation.jakarta.filter;

import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;
import java.io.PrintWriter;
import java.io.StringWriter;

@WebFilter("/*")
public class GlobalExceptionFilter implements Filter {

    // Passe à false pour le mode production
    private static final boolean IS_DEV = true;

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain chain
    ) throws IOException, ServletException {
        try {
            chain.doFilter(request, response);
        } catch (Throwable ex) {
            HttpServletResponse resp = (HttpServletResponse) response;
            resp.setContentType("text/html");
            resp.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);

            PrintWriter writer = resp.getWriter();

            if (IS_DEV) {
                // En mode dev : afficher le détail de l'erreur
                writer.println("<html><head><title>Erreur serveur</title></head><body>");
                writer.println("<h1>Erreur serveur</h1>");
                writer.println("<p><strong>Exception :</strong> " +
                        ex.getClass().getName() + "</p>");
                writer.println("<p><strong>Message :</strong> " +
                        ex.getMessage() + "</p>");

                // Stack trace dans une balise <pre>
                StringWriter sw = new StringWriter();
                ex.printStackTrace(new PrintWriter(sw));
                writer.println("<pre>" + sw.toString() + "</pre>");
                writer.println("</body></html>");
            } else {
                // En mode prod : page plus simple
                writer.println("<html><head><title>Erreur</title></head><body>");
                writer.println("<h1>Une erreur est survenue</h1>");
                writer.println("</body></html>");
            }

            writer.close();
        }
    }
}
```

**Améliorations possibles**

- Utiliser un fichier .env pour gérer le mode d'exécution (dev ou prod)
- Utiliser Pebble pour l'affichage
- Enregistrer les erreurs dans un log en mode prod

#### Ajout d'un fichier .env

Fichier `.env` à la racine du projet, ajouter ce fichier à gitignore.
```
APP_MODE=dev
```

Pour cela, il faut ajouter la bibliothèque dotenv.

```xml
<dependency>
    <groupId>io.github.cdimascio</groupId>
    <artifactId>java-dotenv</artifactId>
    <version>5.2.2</version>
</dependency>
```

```java
package fr.formation.jakarta.filter;

import io.github.cdimascio.dotenv.Dotenv;
import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;
import java.io.PrintWriter;
import java.io.StringWriter;

@WebFilter("/*")
public class GlobalExceptionFilter implements Filter {

    private boolean isDev;

    @Override
    public void init(FilterConfig filterConfig) {
        // Chargement du fichier .env
        Dotenv dotenv = Dotenv.configure()
                              .ignoreIfMissing()
                              .load();

        String env = dotenv.get("APP_ENV", "prod").toLowerCase();
        isDev = env.equals("dev");
    }

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain chain
    ) throws IOException, ServletException {
        try {
            chain.doFilter(request, response);
        } catch (Throwable ex) {
            HttpServletResponse resp = (HttpServletResponse) response;
            resp.setContentType("text/html");
            resp.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);

            PrintWriter writer = resp.getWriter();

            if (isDev) {
                writer.println("<html><head><title>Erreur serveur</title></head><body>");
                writer.println("<h1>Erreur serveur</h1>");
                writer.println("<p><strong>Exception :</strong> " +
                        ex.getClass().getName() + "</p>");
                writer.println("<p><strong>Message :</strong> " +
                        ex.getMessage() + "</p>");

                StringWriter sw = new StringWriter();
                ex.printStackTrace(new PrintWriter(sw));
                writer.println("<pre>" + sw.toString() + "</pre>");
                writer.println("</body></html>");
            } else {
                writer.println("<html><head><title>Erreur</title></head><body>");
                writer.println("<h1>Une erreur est survenue</h1>");
                writer.println("</body></html>");
            }

            writer.close();
        }
    }
}
```

#### Utilisation de Pebble

```java
package fr.formation.jakarta.filter;

import io.github.cdimascio.dotenv.Dotenv;
import io.pebbletemplates.pebble.PebbleEngine;
import io.pebbletemplates.pebble.loader.ClasspathLoader;
import io.pebbletemplates.pebble.template.PebbleTemplate;
import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletResponse;

import java.io.*;
import java.util.HashMap;
import java.util.Map;

@WebFilter("/*")
public class GlobalExceptionFilter implements Filter {

    private boolean isDev;
    private PebbleEngine pebble;

    @Override
    public void init(FilterConfig filterConfig) {
        Dotenv dotenv = Dotenv.configure()
                              .ignoreIfMissing()
                              .load();
        String env = dotenv.get("APP_ENV", "prod").toLowerCase();
        isDev = env.equals("dev");

        ClasspathLoader loader = new ClasspathLoader();
        loader.setPrefix("templates");

        pebble = new PebbleEngine.Builder()
                .loader(loader)
                .build();
    }

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain chain
    ) throws IOException, ServletException {
        try {
            chain.doFilter(request, response);
        } catch (Throwable ex) {
            HttpServletResponse resp = (HttpServletResponse) response;
            resp.setContentType("text/html");
            resp.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);

            Map<String, Object> context = new HashMap<>();
            if (isDev) {
                context.put("exception", ex.getClass().getName());
                context.put("message", ex.getMessage());

                StringWriter sw = new StringWriter();
                ex.printStackTrace(new PrintWriter(sw));
                context.put("stacktrace", sw.toString());

                renderTemplate(resp, "error-dev.peb", context);
            } else {
                renderTemplate(resp, "error.peb", context);
            }
        }
    }

    private void renderTemplate(
            HttpServletResponse response,
            String templateName,
            Map<String, Object> context
    ) throws IOException {
        try (Writer writer = response.getWriter()) {
            PebbleTemplate template = pebble.getTemplate(templateName);
            template.evaluate(writer, context);
        } catch (Exception e) {
            throw new IOException("Erreur de rendu du template", e);
        }
    }
}

```

**Les modèles**

**error.peb**
```twig
{% extends "layout.peb" %}

{% block title %} Erreur {% endblock %}

{% block content %} 
    <h1>Une erreur est survenue</h1>
    <p>Veuillez réessayer plus tard.</p>
{% endblock %}
```

**error-dev.peb**
```twig
{% extends "layout.peb" %}

{% block title %} Erreur {% endblock %}

{% block content %} 
    <h1>Erreur interne</h1>
    <p><strong>Exception :</strong> {{ exception }}</p>
    <p><strong>Message :</strong> {{ message }}</p>
    <pre>{{ stacktrace }}</pre>
{% endblock %}
```

#### Ajout d'un log des erreurs

Pour cela il faut installer un logger dans les dépendances du projet.

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.12</version>
</dependency>

<!-- Implémentation avec Logback -->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.4.14</version>
</dependency>
```

Puis paramétrer le format du log dans un fichier `logback.xml` (dans `src/main/resources/`).

```xml
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss} [%thread] %-5level %logger - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="debug">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

Et enfin utiliser le logger dans la classe GlobalExceptionFilter.

```java
package fr.formation.jakarta.filter;

import io.github.cdimascio.dotenv.Dotenv;
import io.pebbletemplates.pebble.PebbleEngine;
import io.pebbletemplates.pebble.loader.ClasspathLoader;
import io.pebbletemplates.pebble.template.PebbleTemplate;
import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.*;
import java.util.HashMap;
import java.util.Map;

@WebFilter("/*")
public class GlobalExceptionFilter implements Filter {

    private static final Logger LOGGER =
            LoggerFactory.getLogger(GlobalExceptionFilter.class);

    private boolean isDev;
    private PebbleEngine pebble;

    @Override
    public void init(FilterConfig filterConfig) {
        Dotenv dotenv = Dotenv.configure()
                              .ignoreIfMissing()
                              .load();

        String env = dotenv.get("APP_ENV", "prod").toLowerCase();
        isDev = env.equals("dev");

        ClasspathLoader loader = new ClasspathLoader();
        loader.setPrefix("templates");

        pebble = new PebbleEngine.Builder()
                .loader(loader)
                .build();

        LOGGER.info("GlobalExceptionFilter initialisé (mode : {})",
                    isDev ? "dev" : "prod");
    }

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain chain
    ) throws IOException, ServletException {
        try {
            chain.doFilter(request, response);
        } catch (Throwable ex) {
            LOGGER.error("Erreur serveur interceptée", ex);

            HttpServletResponse resp = (HttpServletResponse) response;
            resp.setContentType("text/html");
            resp.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);

            Map<String, Object> context = new HashMap<>();

            if (isDev) {
                context.put("exception", ex.getClass().getName());
                context.put("message", ex.getMessage());

                StringWriter sw = new StringWriter();
                ex.printStackTrace(new PrintWriter(sw));
                context.put("stacktrace", sw.toString());

                renderTemplate(resp, "error-dev.peb", context);
            } else {
                renderTemplate(resp, "error.peb", context);
            }
        }
    }

    private void renderTemplate(
            HttpServletResponse response,
            String templateName,
            Map<String, Object> context
    ) throws IOException {
        try (Writer writer = response.getWriter()) {
            PebbleTemplate template = pebble.getTemplate(templateName);
            template.evaluate(writer, context);
        } catch (Exception e) {
            LOGGER.error("Erreur lors du rendu du template : {}", templateName, e);
            throw new IOException("Erreur de rendu du template", e);
        }
    }
}

```

#### Comment consulter les logs

- Exécution sans Docker : dans la console
- Exécution avec Docker : `docker logs -f <nom du conteneur>`
- Dans tous les cas : `<appender-ref ref="FILE"/>` génère un dossier `logs` qui contient les fichiers de log. 
  Ajouter ce dossier à `.gitignore` serait une bonne idée.