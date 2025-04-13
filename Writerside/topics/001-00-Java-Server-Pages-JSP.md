# Java Server Pages (JSP)

Java|Jakarta Server Pages est une spécification développée pour réaliser des pages dynamiques incluant du code html et du code Java. Le principe est assez similaire à PHP.

### Caractéristiques de JSP

- Intégration facile avec Java EE : Utilise les servlets et autres technologies Jakarta EE.
- 
- Séparation logique-présentation : Favorise l'utilisation du modèle MVC en combinant JSP avec les servlets.
- 
- Support des bibliothèques de balises : Permet d'utiliser des JSP Tag Libraries (JSTL) et des balises personnalisées pour simplifier le code.

- Gestion des sessions : Prend en charge la gestion des sessions utilisateur automatiquement.

- Compilation côté serveur : Les fichiers JSP sont compilés en servlets lors de la première requête.

### Un exemple de JSP

```jsp
<!-- index.jsp -->
<%@ page language="java" contentType="text/html; charset=UTF-8" %>
<!DOCTYPE html>
<html>
<head>
    <title>Exemple JSP</title>
</head>
<body>
    <h1>Bonjour, JSP!</h1>
    <p>
    La date et l'heure actuelles sont : 
    <%= new java.util.Date() %>
    </p>
</body>
</html>
```

## La syntaxe des JSP

### Directives JSP (`<%@ ... %>`)

Les directives donnent des instructions au conteneur JSP. Elles commencent par `<%@ ... %>`.

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
```

- `language="java"` : Spécifie que la page utilise Java.

- `contentType="text/html; charset=UTF-8"` : Définit le type MIME et l'encodage.

- `pageEncoding="UTF-8"` : Spécifie l'encodage du fichier JSP.

#### Autres directives utiles

**Inclure un fichier**
```jsp
<%@ include file="header.jsp" %>
```

**Importer une classe Java**

```jsp
<%@ page import="java.util.Date" %>
<%
    Date date = new Date();
    out.println("Date actuelle : " + date);
%>
```

**Définir un gestionnaire d'erreur**

```jsp
<%@ page errorPage="erreur.jsp" %>
```

### Scriplets (`<% ... %>`)

Un scriplet permet d'insérer du code Java dans une page JSP.

```jsp
<%
    String message = "Bonjour, JSP !";
    out.println("<p>" + message + "</p>");
%>
```

- On déclare et assigne une valeur à la variable message.

- `out.println()` est utilisé pour afficher du contenu HTML dans la page.

### Expressions (`<%= ... %>`)

Permet d'afficher directement une valeur dans la page HTML.

```jsp
<p>La date actuelle est : <%= new java.util.Date() %></p>
```

### Déclarations (`<%! ... %>`)

Utilisées pour définir des variables et des méthodes accessibles à toute la page JSP.

```jsp
<%! int addition(int a, int b) { return a + b; } %>
<p>Résultat : <%= addition(3, 4) %></p>
```

## Les objets implicites

Les objets implicites sont des objets pré-définis fournis par le conteneur JSP, permettant d’accéder facilement aux informations de requête, de réponse, de session et de contexte d’application sans avoir besoin de les instancier explicitement.

### L’objet request

Permet d’accéder aux paramètres de la requête HTTP.

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" %>
<html>
<head><title>Exemple Request</title></head>
<body>
    <% 
        String nom = request.getParameter("nom");
        String age = request.getParameter("age");
    %>
    <p>Nom : <%= nom %></p>
    <p>Âge : <%= age %></p>
</body>
</html>
```

> Avec JSP, la méthode `getParameter` récupère les données transmises avec la méthode GET ou POST.

#### Récupérer tous les paramètres

```jsp
<%@ page import="java.util.Enumeration" %>

<%
    Enumeration<String> paramNames = request.getParameterNames();
    while (paramNames.hasMoreElements()) {
        String paramName = paramNames.nextElement();
        String paramValue = request.getParameter(paramName);
%>
        <p><%= paramName %> : <%= paramValue %></p>
<%
    }
%>
```

### L'objet Response

Permet de gérer la réponse HTTP (redirections, cookies, type de contenu...).

```jsp
<%
    response.sendRedirect("home.jsp");
%>
```

### L'objet Session

Gère la session, équivalent au `$_SESSION` de PHP.

```jsp
<%
    session.setAttribute("user", "Alice");
%>
<p>
    Bienvenue, <%= session.getAttribute("user") %>
</p>
```

### L’objet application

L’objet application stocke des informations accessibles à toutes les sessions et requêtes. Ce concept n'existe pas en PHP.

```jsp
<%
    Integer compteur = (Integer) application.getAttribute("compteur");
    if (compteur == null) {
        compteur = 1;
    } else {
        compteur++;
    }
    application.setAttribute("compteur", compteur);
%>
<p>Nombre de visiteurs : <%= compteur %></p>

```

### L’objet pageContext

L’objet pageContext donne accès aux objets des différentes portées (page, request, session, application).

```jsp
<%
    pageContext.setAttribute(
        "message", "Bonjour JSP!", 
        PageContext.SESSION_SCOPE
    );
%>
<p>
    <%= pageContext.getAttribute(
            "message", 
            PageContext.SESSION_SCOPE
        ) 
    %>
</p>
```

### L’objet config

L’objet config est utilisé pour récupérer des paramètres d’initialisation définis dans le fichier web.xml.

```xml
<servlet>
    <servlet-name>MyJSP</servlet-name>
    <jsp-file>/myPage.jsp</jsp-file>
    <init-param>
        <param-name>emailAdmin</param-name>
        <param-value>admin@example.com</param-value>
    </init-param>
</servlet>

```

```jsp
<p>Email Admin : <%= config.getInitParameter("emailAdmin") %></p>
```

### L’objet page

L’objet page représente l’instance actuelle de la page JSP (équivalent à this en Java).

```jsp
<%
    out.println("Classe de la page : " + page.getClass().getName());
%>
```

### L’objet exception

L’objet exception est disponible uniquement dans les pages JSP déclarées comme des pages d'erreur.

```jsp
<!-- 
    Génération d'une erreur à des fins de démonstration,
    éviter de le faire en production :-) 
-->
<%@ page isErrorPage="true" %>

<p>Une erreur est survenue : <%= exception.getMessage() %></p>
```



## JSP et le pattern MVC

Les JSP se comportent comme le PHP des années 90. Les pratiques ont depuis quelque peu évolué et il est désormais impensable de ne pas utiliser un système de routage.

Pour ce faire, il faut combiner JSP et Servlet.

```java
// HelloServlet.java

@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
    protected void doGet(
        HttpServletRequest request, 
        HttpServletResponse response
    ) throws ServletException, IOException 
    {
        // Définition d'un attribut personnalisé 
        // dans l'objet Request.
        // Cet attribut sera lu par la page JSP
        request.setAttribute(
            "message", 
            "Hello depuis le Servlet!"
        );
        
        // Récupération d'un dispatcher pour la page cible
        RequestDispatcher dispatcher = request.getRequestDispatcher(
            "hello.jsp"
        );
        
        // Transmission de la requête à la page JSP
        dispatcher.forward(request, response);
    }
}
```

Grâce à cette technique, il est possible de séparer la couche, présentation de la logique métier. 
Sans cela, développer une application web avec les seules JSP nous ramène à la préhistoire du Web.

## La gestion des sessions

Une session est un mécanisme permettant de stocker des informations utilisateur côté serveur sur plusieurs requêtes HTTP. Jakarta Servlet utilise des objets de type HttpSession pour gérer les sessions.

### Création et récupération d'une session

Pour créer ou récupérer une session, on utilise la méthode `getSession()` de l'objet `HttpServletRequest`.

```java
// Création ou récupération de la session
HttpSession session = request.getSession();
```

### Stockage d'attributs dans la session

```java
HttpSession session = request.getSession();

// Ajouter un attribut
session.setAttribute("username", "joe user");

// Récupérer un attribut
String username = (String) session.getAttribute("username");

// Supprimer un attribut
session.removeAttribute("username");
```

### Stockage d'un objet

Il est possible de stocker des objets en session, il faut juste que ces objets implémentent l'interface `Serializable`.
Jakarta se charge de la sérialisation et dé-serialisation.

**Une classe sérialisable**

```java
package fr.mvc.app.model;

import java.io.Serializable;

public class User implements Serializable {
    private String username;
    private String email;

    public User(String username, String email) {
        this.username = username;
        this.email = email;
    }

    public String getUsername() {
        return username;
    }

    public String getEmail() {
        return email;
    }

    @Override
    public String toString() {
        return "User [username=" + username + ", email=" + email + "]";
    }
}
```

**Enregistrement de l'objet en session**

```java
User user = new User("joe user", "joe@user.com");

// Création de la session et enregistrement de l'objet
HttpSession session = request.getSession();
session.setAttribute("user", user);
```

**Récupération de l'objet**

```java
// Il faut caster pour que Jakarta sache dans quelle classe dé-sérialiser
User user = (User) session.getAttribute("user");
```

### Gestion de la durée de la session

#### Dans une servlet à la création de la session

```java
// Durée en secondes (ici 30 minutes)
session.setMaxInactiveInterval(30 * 60);
```

#### Dans `web.xml` pour l'ensemble des sessions

```xml
<session-config>
    <session-timeout>30</session-timeout>
</session-config>
```

### Invalidations et destruction de la session

Détruit la session et libère les ressources.

```java
// Invalider la session
session.invalidate();
```

### Vérification de l'état de la session

```java
if (session == null || session.getAttribute("username") == null) {
    response.sendRedirect("login.jsp");
}
```

### Exercices

#### Compteur de visites

- Créer une route et une page qui incrémente et affiche un compteur à chaque fois qu'elle est visitée au sein de la même session.
- Créer un bouton de remise à zéro du compteur.
- Quelles sont les limites de cette solution et quelles solutions alternatives pourraient être mises en place ?

#### Gestion de panier

- Créer une route et une page qui présente un formulaire offrant la possibilité de saisir le nom d'un produit et sa quantité.
- Stocker cette information dans une liste de map enregistrée en session
- Afficher le contenu du panier dans une autre page
- Mettre en place un lien qui permet de vider le panier

## Les ressources externes

Une application web contient généralement des ressources externes telles que des images des fichiers, CSS ou JavaScript.
À la compilation du fichier `.war`, le contenu du dossier `webapp` est automatiquement ajouté. 
C'est donc là qu'il faut placer les ressources.

Étant donné qu'il est possible d'exécuter un fichier `.war` avec un contexte différent, il sera prudent d'ajouter ce contexte en préfixe des liens avec le code suivant :

```jsp
${pageContext.request.contextPath}
```

**Exemple**

```jsp
<link rel="stylesheet" href="${pageContext.request.contextPath}/assets/css/style.css">
<script src="${pageContext.request.contextPath}/assets/js/script.js"></script>
<img src="${pageContext.request.contextPath}/assets/images/logo.png" alt="Logo">
```

### Gestion des bibliothèques externes

Pour gérer les bibliothèques externes CSS ou JavaScript, il existe plusieurs solutions.

### Utiliser un CDN

La solution la plus simple tant que la connexion à l'Internet est assurée.

**exemple**

```html
<link
  rel="stylesheet"
  href="https://cdn.jsdelivr.net/npm/@picocss/pico@2/css/pico.min.css"
>
```

### Télécharger les fichiers manuellement

Cette solution implique de tracer avec Git des ressources qui ne doivent pas être modifiées 
donc dans la plupart des cas, il faudra s'abstenir sauf si les dépendances sont minimales 
et/ou que le nombre de développeurs est très réduit (un nombre impair inférieur à trois par exemple).

### Utiliser un gestionnaire de dépendances

Les dépendances sont référencée dans un fichier `package.json`, 
un nouveau développeur peut les télécharger automatiquement 
et tout le monde travaille avec les mêmes versions.

Le code des bibliothèques est stocké dans un dossier `node_modules` qui n'est pas inclus dans le fichier `.war`. 
Il faudra donc copier les fichiers requis dans un sous dossier de `webapp`.

```shell
npm init -y
npm install @picocss/pico --save

cp node_modules/@picocss/pico/css/pico.css src/main/webapp/css/pico.css 
```

À chaque mise à jour, il faudra copier à nouveau les fichiers. 
Pour éviter les oublis, il sera prudent de créer un script qui réalise ces copies et de le partager avec les collègues.

### Utiliser un bundler tel que Webpack

L'intérêt ici réside dans la consolidation de toutes les dépendances en un seul fichier.

```shell
npm install webpack webpack-cli --save-dev
npm install @picocss/pico --save
```

**Configuration de Webpack**

```javascript
// webpack.config.js

const path = require('path');

module.exports = {
  entry: './src/main/webapp/assets/js/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'src/main/webapp/assets/js'),
  },
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
  mode: 'production',
};
```

**Le fichier d'entrée**

```javascript
// src/main/webapp/assets/js/app.js

import '@picocss/pico/css/pico.min.css';

document.addEventListener('DOMContentLoaded', () => {
    console.log('Pico CSS chargé avec Webpack !');
});
```

**Modification dans `package.json`**

```json
"scripts": {
  "build": "webpack"
}
```

**Génération du bundle**

```shell
npm run build
```

**Intégration dans un jsp**

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html>
<html lang="fr">
<head>
    <title>Intégration Pico CSS avec Webpack</title>
    <link rel="stylesheet" href="${pageContext.request.contextPath}/assets/js/bundle.js">
</head>
<body>
    <article class="container">
        <h1>Bienvenue sur Jakarta EE avec Pico CSS</h1>
        <p>Pico CSS est chargé avec Webpack.</p>
        <button class="contrast">Cliquez-moi</button>
    </article>

    <script src="${pageContext.request.contextPath}/assets/js/bundle.js"></script>
</body>
</html>
```



