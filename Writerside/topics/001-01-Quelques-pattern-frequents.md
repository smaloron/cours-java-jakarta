# Quelques patterns fréquents

## Traitement d'un formulaire avec la méthode POST

**Structure de l'application**

```
myapp/
├── src/main/java/fr/mvc/app/
│   └── controller/
│       └── NameServlet.java
├── src/main/webapp/
│   ├── name-form.jsp
│   ├── result.jsp
│   └── WEB-INF/
│       └── web.xml
└── pom.xml

```
**La servlet**

```java
package fr.mvc.app.controller;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/name")
public class NameServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    // Affichage du formulaire
    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws ServletException, IOException {
        // Affiche la page du formulaire
        request.getRequestDispatcher("name-form.jsp")
                .forward(request, response);
    }

    // Traitement du formulaire
    @Override
    protected void doPost(HttpServletRequest request,
                          HttpServletResponse response)
            throws ServletException, IOException {
        // Récupération du nom envoyé via le formulaire
        String name = request.getParameter("name");

        // Envoi du nom à la page de résultat
        request.setAttribute("userName", name);

        // Redirection vers la page de résultat
        request.getRequestDispatcher("result.jsp")
                .forward(request, response);
    }
}

```

**Le formulaire**

```jsp
<!-- name-form.jsp -->

<!DOCTYPE html>
<html>
<head>
    <title>Formulaire de nom</title>
</head>
<body>
    <h2>Entrez votre nom</h2>
    <form action="name" method="post">
        <label for="name">Nom :</label>
        <input type="text" id="name" name="name" required>
        <button type="submit">Envoyer</button>
    </form>
</body>
</html>
```

**Le résultat**

```jsp
<!-- result.jsp -->

<!DOCTYPE html>
<html>
<head>
    <title>Résultat</title>
</head>
<body>
    <h2>Bonjour, 
        <%= request.getAttribute("userName") != null ? 
            request.getAttribute("userName") : "inconnu" %>!
    </h2>
</body>
</html>
```

### Exercice

Quelques exercices pour pratiquer les points suivants :


- Utilisation des méthodes doGet et doPost dans les servlets.

- Manipulation des données reçues depuis les formulaires.

- Redirection vers des pages JSP pour afficher les résultats.

- Vérification et validation des données saisies.

- Organisation de l'architecture MVC avec Jakarta Servlets et JSP.

#### Formulaire de calcul de l'âge

**Objectif :** Calculer l'âge à partir de l'année de naissance.

**Détails :**

- Page JSP avec un formulaire demandant l'année de naissance.

- Servlet qui calcule l'âge à partir de l'année saisie et de l'année courante.

- Page de résultat affichant l'âge calculé.

#### Formulaire de conversion de devises

**Objectif** : Convertir un montant d'une devise à une autre.

**Détails** :

- Page JSP avec un formulaire pour saisir le montant, la devise source et la devise cible.

- Servlet qui utilise un taux de conversion fixe pour calculer le montant converti.

- Page de résultat affichant la somme convertie.

## Gestion des sessions et authentification minimaliste

**Structure de l'application**

```
myapp/
├── src/main/java/fr/mvc/app/controller/
│   ├── LoginServlet.java
│   ├── WelcomeServlet.java
│   └── LogoutServlet.java
├── src/main/webapp/
│   ├── auth/
│   │    ├── login.jsp
│   │    ├── welcome.jsp
│   │    └── logout.jsp
│   └── WEB-INF/
│       └── web.xml
└── pom.xml
```
### Les servlets

**LoginServlet**

```java
package fr.mvc.app.controller;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;
import java.io.IOException;

@WebServlet("/login")
public class LoginServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    @Override
    protected void doGet(HttpServletRequest request, 
                         HttpServletResponse response) 
                         throws ServletException, IOException {
        // Redirection vers la page de connexion
        request.getRequestDispatcher("/auth/login.jsp")
               .forward(request, response);
    }

    @Override
    protected void doPost(HttpServletRequest request, 
                          HttpServletResponse response) 
                          throws ServletException, IOException {
        // Récupération du nom d'utilisateur
        String username = request.getParameter("username");

        // Création de la session
        HttpSession session = request.getSession();
        session.setAttribute("userName", username);

        // Redirection vers la page de bienvenue
        response.sendRedirect("welcome");
    }
}
```

**WelcomeServlet**

```java
package fr.mvc.app.controller;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;
import java.io.IOException;

@WebServlet("/welcome")
public class WelcomeServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    @Override
    protected void doGet(HttpServletRequest request, 
                         HttpServletResponse response) 
                         throws ServletException, IOException {
        // Vérification de la session
        HttpSession session = request.getSession(false);
        if (session == null || session.getAttribute("userName") == null) {
            response.sendRedirect("login");
            return;
        }

        // Redirection vers la page de bienvenue
        request.getRequestDispatcher("/auth/welcome.jsp")
               .forward(request, response);
    }
}
```

**LogoutServlet**

```java
package fr.mvc.app.controller;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;
import java.io.IOException;

@WebServlet("/logout")
public class LogoutServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    @Override
    protected void doGet(HttpServletRequest request, 
                         HttpServletResponse response) 
                         throws ServletException, IOException {
        // Invalidation de la session
        HttpSession session = request.getSession(false);
        if (session != null) {
            session.invalidate();
        }

        // Redirection vers la page de déconnexion
        request.getRequestDispatcher("/auth/logout.jsp")
               .forward(request, response);
    }
}
```

### Les JSP

**login.jsp**

```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Connexion</title>
</head>
<body>
    <h2>Connexion</h2>
    <form action="login" method="post">
        <label for="username">Nom d'utilisateur :</label>
        <input type="text" id="username" name="username" required>
        <button type="submit">Se connecter</button>
    </form>
</body>
</html>
```

**welcome.jsp**

```jsp
<% 
    String username = (String) session.getAttribute("userName");
%>
<!DOCTYPE html>
<html>
<head>
    <title>Bienvenue</title>
</head>
<body>
    <h2>Bienvenue, <%= username %> !</h2>
    <p>Vous êtes connecté.</p>
    <a href="logout">Se déconnecter</a>
</body>
</html>
```

**logout.jsp**

```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Déconnexion</title>
</head>
<body>
    <h2>Vous êtes déconnecté.</h2>
    <a href="login">Retour à la page de connexion</a>
</body>
</html>
```

## Gestion de l'upload



```java
package fr.mvc.app.controller;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.MultipartConfig;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.Part;

import java.io.File;
import java.io.IOException;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.UUID;

@WebServlet("/upload")
@MultipartConfig(fileSizeThreshold = 1024 * 1024 * 2,
        maxFileSize = 1024 * 1024 * 10,
        maxRequestSize = 1024 * 1024 * 50)
public class UploadServlet extends HttpServlet {

    private static final String UPLOAD_DIR = "uploads";

    @Override
    protected void doPost(
            HttpServletRequest request, 
            HttpServletResponse response
    ) throws ServletException, IOException 
    {
        // Récupérer le chemin de sauvegarde
        String applicationPath = getServletContext().getRealPath("");
        String uploadPath = applicationPath + File.separator + UPLOAD_DIR;

        // Créer le dossier si nécessaire
        File uploadDir = new File(uploadPath);
        if (!uploadDir.exists()) uploadDir.mkdirs();

        for (Part part : request.getParts()) {
            String originalFileName = Paths.get(part.getSubmittedFileName())
                                           .getFileName()
                                           .toString();
            
            String fileExtension = getFileExtension(originalFileName);
            String uniqueFileName = UUID.randomUUID().toString() 
                                    + fileExtension;

            // Chemin complet du fichier à enregistrer
            String filePath = uploadPath 
                              + File.separator 
                              + uniqueFileName;
            
            // Enregistrement du fichier
            part.write(filePath);

            response.getWriter().println(
                    "Fichier uploadé avec succès : " + uniqueFileName
            );
        }
    }

    // Méthode pour extraire l'extension du fichier
    private String getFileExtension(String fileName) {
        Path path = Paths.get(fileName);
        String name = path.getFileName().toString();
        int dotIndex = name.lastIndexOf(".");
        return (dotIndex == -1) ? "" : name.substring(dotIndex);
    }


}

```

### Les options de `@MultipartConfig`

`fileSizeThreshold`
: Taille limite en octets avant que le fichier ne soit écrit sur le disque. Si la taille est inférieure à ce seuil le fichier sera stocké en mémoire vive.

`maxFileSize`
: Taille maximale d'un seul fichier uploadé.

`maxRequestSize`
: Taille maximale pour l'ensemble de la requête, c'est-à-dire tous les fichiers et autres paramètres du formulaire combinés.

### Inférer le mime type

La méthode `probeContentType` de `nio` permet de connaître le mime type d'un fichier sans passer par l'extension.
Voici une classe utilitaire qui réalise la conversion.

```java
import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.HashMap;
import java.util.Map;

public class MimeTypeUtils {

    private static final Map<String, String> mimeToExtensionMap = new HashMap<>();

    static {
        mimeToExtensionMap.put("image/jpeg", ".jpg");
        mimeToExtensionMap.put("image/png", ".png");
        mimeToExtensionMap.put("image/gif", ".gif");
        mimeToExtensionMap.put("image/bmp", ".bmp");
        mimeToExtensionMap.put("image/webp", ".webp");
        mimeToExtensionMap.put("text/plain", ".txt");
        mimeToExtensionMap.put("text/html", ".html");
        mimeToExtensionMap.put("text/csv", ".csv");
        mimeToExtensionMap.put("text/xml", ".xml");
        mimeToExtensionMap.put("application/pdf", ".pdf");
        mimeToExtensionMap.put("application/zip", ".zip");
        mimeToExtensionMap.put("application/json", ".json");
        mimeToExtensionMap.put("application/msword", ".doc");
        mimeToExtensionMap.put("application/vnd.openxmlformats-officedocument.wordprocessingml.document", ".docx");
        mimeToExtensionMap.put("application/vnd.ms-excel", ".xls");
        mimeToExtensionMap.put("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet", ".xlsx");
        mimeToExtensionMap.put("application/vnd.ms-powerpoint", ".ppt");
        mimeToExtensionMap.put("application/vnd.openxmlformats-officedocument.presentationml.presentation", ".pptx");
    }

    public static String getFileExtension(String filePath) {
        try {
            Path path = Paths.get(filePath);
            String mimeType = Files.probeContentType(path);
            if (mimeType == null) {
                return "unknown";
            }
            return mimeToExtensionMap.getOrDefault(mimeType, "unknown");
        } catch (IOException e) {
            return "error";
        }
    }
}

```

Intégrer cette classe dans le code de l'upload