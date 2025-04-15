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