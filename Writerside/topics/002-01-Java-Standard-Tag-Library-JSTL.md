# Java Standard Tag Library (JSTL)

La JSTL propose de simplifier et rendre plus lisibles les JSP. 

### Installation
Avec Maven dans le fichier `pom.xml`
```xml
<dependencies>
    <dependency>
        <groupId>jakarta.servlet.jsp.jstl</groupId>
        <artifactId>jakarta.servlet.jsp.jstl-api</artifactId>
        <version>3.0.0</version>
    </dependency>
    <dependency>
        <groupId>org.glassfish.web</groupId>
        <artifactId>jakarta.servlet.jsp.jstl</artifactId>
        <version>3.0.0</version>
    </dependency>
</dependencies>
```

## Principes

JSTL est composée de plusieurs bibliothèques de balises regroupées en cinq catégories principales :

- Core Tags (c): contrôle de flux, importation, gestion des variables, etc.

- Formatting Tags (fmt): formatage de nombres, dates, i18n.

- SQL Tags (sql): exécution de requêtes SQL (à éviter en production).

- XML Tags (x): traitement XML.

- Functions (fn): fonctions utilitaires (ex: manipulation de chaînes).

### Déclaration des bibliothèques

Chaque JSP doit déclarer les bibliothèques qu'elle utilise en début de fichier dans une directive.

```jsp
<%@ taglib uri="http://jakarta.ee/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://jakarta.ee/jsp/jstl/fmt" prefix="fmt" %>
<%@ taglib uri="http://jakarta.ee/jsp/jstl/sql" prefix="sql" %>
<%@ taglib uri="http://jakarta.ee/jsp/jstl/functions" prefix="fn" %>
<%@ taglib uri="http://jakarta.ee/jsp/jstl/xml" prefix="xml" %>
```

## Les balises Core

### Déclaration ou affectation

```jsp
<c:set var="greeting" value="Bonjour" />
<p>${greeting}</p>
```

### Affichage

```jsp
<c:set var="user" value="Joe" />
<c:out value="${user}" default="Anonyme" />
```

### Condition simple

```jsp
<c:if test="${user.admin}">
  <p>Bienvenue, administrateur !</p>
</c:if>
```

### Switch

```jsp
<c:choose>
  <c:when test="${score >= 90}">Excellent</c:when>
  <c:when test="${score >= 70}">Bien</c:when>
  <c:otherwise>À améliorer</c:otherwise>
</c:choose>
```

### Boucle sur un iterable

```jsp
<c:forEach var="prod" items="${productList}" varStatus="loop">
  ${loop.index + 1} - ${prod.name} (${prod.price} €)<br/>
</c:forEach>
```

> varStatus permet d'accéder à .index, .count, .first, .last

### Boucle sur une chaîne
Équivalent à un split suivi d'un foreach.

```jsp
<c:forTokens items="Java,Python,PHP" delims="," var="lang">
  <p>${lang}</p>
</c:forTokens>
```

### Inclusion d'une jsp

Équivalent à l'include ou require de PHP.

```jsp
<c:import url="/includes/footer.jsp" />
```

### Redirection

```jsp
<c:redirect url="home.jsp" />
```

### Création d'url et de paramètres

```jsp
<c:url var="productUrl" value="/productDetails.jsp">
  <c:param name="id" value="${product.id}" />
</c:url>
<a href="${productUrl}">${product.name}</a>
```

## Les balises de formatage

### Attributs communs à tous les tags

| Attribut  | Description                                                                                                                                                  |
|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| var       | Spécifie le nom de la variable dans laquelle la valeur générée sera stockée.                                                                                 |
| scope     | Définit la portée de la variable définie par `var`. Les valeurs possibles sont `page`, `request`, `session`, `application`. La portée par défaut est `page`. |
| value     | La valeur ou l'expression EL à traiter ou à afficher.                                                                                                        |
| default   | Utilisé pour spécifier une valeur par défaut si la valeur principale est `null` ou absente.                                                                  |
| escapeXml | Si défini sur `true`, les caractères spéciaux XML (comme `<`, `>`, `&`) sont échappés. Par défaut : `true`.                                                  |
| locale    | Définit la locale pour le formatage des dates, des nombres ou des messages.                                                                                  |
| bundle    | Utilisé pour spécifier le bundle de ressources pour les messages localisés.                                                                                  |
| timeZone  | Spécifie le fuseau horaire pour le formatage des dates et heures.                                                                                            |

>Si l'attribut `var` est omis, la date formatée sera directement affichée.

### Le formatage des nombres

```jsp
<fmt:formatNumber value="${number}" type="number|currency|percent" />
```

**Exemple**

```jsp
<fmt:formatNumber value="${amount}" type="currency" />
```

### Les attributs de formatNumber

| Attribut          | Description                                                                                               |
|-------------------|-----------------------------------------------------------------------------------------------------------|
| value             | La valeur numérique à formater. Peut être un objet `Number`, une chaîne ou une expression EL.             |
| type              | Le type de formatage (`number`, `currency`, `percent`). Par défaut : `number`.                            |
| pattern           | Motif personnalisé pour le formatage du nombre. Remplace les autres attributs si défini.                  |
| maxIntegerDigits  | Nombre maximal de chiffres pour la partie entière.                                                        |
| minIntegerDigits  | Nombre minimal de chiffres pour la partie entière.                                                        |
| maxFractionDigits | Nombre maximal de chiffres pour la partie fractionnaire.                                                  |
| minFractionDigits | Nombre minimal de chiffres pour la partie fractionnaire.                                                  |
| groupingUsed      | Indique si le regroupement (séparateur de milliers) est utilisé (`true` ou `false`). Par défaut : `true`. |
| var               | Nom de la variable pour stocker le résultat formaté.                                                      |
| scope             | Portée de la variable (par défaut : `page`).                                                              |


### Le formatage des dates

```jsp
<fmt:formatDate value="${date}" pattern="dd-MM-yyyy" />
```

**Exemple**

```jsp
<fmt:formatDate value="${currentDate}" pattern="dd/MM/yyyy" />
```

#### Les attributs de formatDate

| Attribut  | Description                                                                |
|-----------|----------------------------------------------------------------------------|
| value     | La date à formater. Peut être un objet `Date` ou une chaîne de date.       |
| type      | Le type de formatage (`date`, `time`, `both`). Par défaut, `date`.         |
| pattern   | Le motif (format) de la date. Si défini, il remplace le format par défaut. |
| dateStyle | Style pour la date (`default`, `short`, `medium`, `long`, `full`).         |
| timeStyle | Style pour l'heure (`default`, `short`, `medium`, `long`, `full`).         |
| var       | Nom de la variable pour stocker le résultat formaté.                       |
| scope     | Portée de la variable (par défaut : `page`).                               |




#### Le pattern de formatDate

| Symbole | Signification                  | Exemple            |
|---------|--------------------------------|--------------------|
| y       | Année (4 chiffres)             | yyyy -> 2025       |
| yy      | Année (2 chiffres)             | yy -> 25           |
| M       | Mois (1 ou 2 chiffres)         | M -> 4, MM -> 04   |
| MMM     | Mois (abrégé)                  | MMM -> Avr         |
| MMMM    | Mois (complet)                 | MMMM -> Avril      |
| d       | Jour du mois (1 ou 2 chiffres) | d -> 6, dd -> 06   |
| E       | Jour de la semaine (abrégé)    | E -> Dim           |
| EEEE    | Jour de la semaine (complet)   | EEEE -> Dimanche   |
| H       | Heure (0-23)                   | H -> 9, HH -> 09   |
| h       | Heure (1-12, format AM/PM)     | h -> 9, hh -> 09   |
| m       | Minute                         | m -> 3, mm -> 03   |
| s       | Seconde                        | s -> 7, ss -> 07   |
| S       | Millisecondes                  | S -> 1, SSS -> 001 |
| a       | Indicateur AM/PM               | a -> AM            |
| z       | Fuseau horaire abrégé          | z -> CET           |
| Z       | Décalage GMT                   | Z -> +0100         |

**Exemples de patterns courants**

| Pattern                      | Résultat Exemple               |
|------------------------------|--------------------------------|
| yyyy-MM-dd                   | 2025-04-06                     |
| dd/MM/yyyy                   | 06/04/2025                     |
| EEEE, d MMMM yyyy            | Dimanche, 6 avril 2025         |
| yyyy.MM.dd G 'at' HH:mm:ss z | 2025.04.06 AD at 09:30:00 CET  |
| h:mm a                       | 9:30 AM                        |
| hh 'heures' a                | 09 heures AM                   |
| HH:mm:ss.SSS                 | 09:30:00.123                   |
| yyyy-MM-dd'T'HH:mm:ssZ       | 2025-04-06T09:30:00+0100       |
| EEE, d MMM yyyy HH:mm:ss Z   | Dim, 6 avr 2025 09:30:00 +0100 |


#### Gestion de date localisées

```jsp
<fmt:setLocale value="fr_FR" />
<fmt:formatDate value="${now}" pattern="EEEE, d MMMM yyyy" />
```
### La conversion de chaînes en date

Ce tag possède les mêmes attributs que `formatDate`.

```jsp
<c:set var="dateString" value="2025-04-06"/>
<fmt:parseDate value="${dateString}" 
               var="parsedDate" 
               pattern="yyyy-MM-dd"/>

<p>Date : ${parsedDate}</p>
```


### La conversion de chaînes en nombres

```jsp
<fmt:parseNumber value="${stringValue}" var="number" />
```

### La localisation des textes

La balise `<fmt:setBundle>` est utilisée pour charger un fichier de ressources (de type ResourceBundle) contenant des paires clé/valeur de chaînes de caractères pour la localisation.

```jsp
<fmt:setBundle basename="nom_du_fichier" var="nom_de_la_variable" />
```

Le nom du fichier portera l'extension `.properties` et un suffixe déterminant la langue.
Par exemple `messages_fr.proprerties` pour le français.

Ce qui donnera :

```jsp
<fmt:setLocale value="fr_FR" />
<fmt:setBundle basename="messages" var="bundle" />
```

Le contenu du fichier de localisation fait correspondre des clefs avec des valeurs qui représentent la traduction.

```
greeting = Bonjour!
farewell = Au revoir!
```

#### Affichage des messages localisés

```jsp
<p><fmt:message key="greeting" bundle="${bundle}" /></p>
<p><fmt:message key="farewell" bundle="${bundle}" /></p>
```

#### Messages avec des paramètres

Il est possible de définir des paramètres dans la traduction comme ceci :

```
welcome = Bienvenue, {0}
```

Le paramètre est ensuite passé lors de l'affichage de la traduction

```jsp
<fmt:setBundle basename="messages" var="bundle" />
<fmt:message key="welcome" bundle="${bundle}">
    <fmt:param value="Alice" />
</fmt:message>
```

#### Passage de la locale dans l'URL

Il est fréquent de passer la locale dans l'URL comme ceci : `http://route?lang=fr_FR` ou bien `http://page.jsp?lang=fr_FR`.
Dans ce cas la page jsp doit récupérer ce paramètre pour change la locale.

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>

<!-- Récupérer la langue passée dans l'URL -->
<c:choose>
    <c:when test="${not empty param.lang}">
        <fmt:setLocale value="${param.lang}" />
    </c:when>
    <c:otherwise>
        <fmt:setLocale value="fr_FR" />
    </c:otherwise>
</c:choose>

<!-- Charger le fichier de ressources correspondant -->
<fmt:setBundle basename="messages" var="bundle" />

<h2><fmt:message key="greeting" bundle="${bundle}" /></h2>

<!-- Liens pour changer la langue -->
<a href="?lang=fr_FR">Français</a> |
<a href="?lang=en_US">English</a>
```

## Les balises de fonction

| Fonction                             | Description                                                                    |
|--------------------------------------|--------------------------------------------------------------------------------|
| `fn:contains(str, substr)`           | Vérifie si `str` contient la sous-chaîne `substr` (renvoie `true` ou `false`). |
| `fn:containsIgnoreCase(str, substr)` | Vérifie si `str` contient `substr` sans tenir compte de la casse.              |
| `fn:endsWith(str, suffix)`           | Vérifie si `str` se termine par le suffixe spécifié.                           |
| `fn:startsWith(str, prefix)`         | Vérifie si `str` commence par le préfixe spécifié.                             |
| `fn:escapeXml(str)`                  | Échappe les caractères XML dans `str`.                                         |
| `fn:indexOf(str, substr)`            | Renvoie l'index de la première occurrence de `substr` dans `str`.              |
| `fn:length(obj)`                     | Renvoie la longueur d'une chaîne, d'un tableau ou d'une collection.            |
| `fn:replace(str, old, new)`          | Remplace toutes les occurrences de `old` par `new` dans `str`.                 |
| `fn:substring(str, start, end)`      | Renvoie la sous-chaîne de `str` entre les indices `start` et `end`.            |
| `fn:substringAfter(str, substr)`     | Renvoie la partie de `str` après la première occurrence de `substr`.           |
| `fn:substringBefore(str, substr)`    | Renvoie la partie de `str` avant la première occurrence de `substr`.           |
| `fn:toLowerCase(str)`                | Convertit la chaîne en minuscules.                                             |
| `fn:toUpperCase(str)`                | Convertit la chaîne en majuscules.                                             |
| `fn:trim(str)`                       | Supprime les espaces en début et fin de chaîne.                                |


### Exemples d'utilisation

**Vérification de la présence d'une sous-chaîne**
```jsp
<c:set var="message" value="Bonjour tout le monde!" />

<c:if test="${fn:contains(message, 'monde')}">
    <p>La chaîne contient "monde".</p>
</c:if>
```

**Conversion en majuscules**
```jsp
<c:set var="text" value="java est génial!" />
<p>Texte en majuscules : ${fn:toUpperCase(text)}</p>
```

**Remplacement de sous-chaîne**

```jsp
<c:set var="phrase" value="J'aime le café" />
<p>${fn:replace(phrase, 'café', 'thé')}</p>
```





