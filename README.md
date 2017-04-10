# LABORATOIRE 4 - Storage as a Service & NoSQL

Dans le [laboratoire précédent](https://github.com/crabone/HEIGVD-CLD-2017-Labo-PaaS),
nous nous sommes initiés aux infrastructure cloud de type Platform as a Service
(PaaS) avec le service Google App Engine (GAE). Le présent laboratoire s'inscrit
dans la continuité du précédent. Nous allons, déployer une application de
e-commerce dans GAE, implémenter les mécanisme de transaction pour l'écriture de
données dans le Datastore et, examiner le comportement de l'application dans le
cas d'accès concurrents.

## ÉTUDIANTS

* FRANCHINI Fabien
* DONGMO NGOUMNAÏ Annie Sandra

## TABLE DES MATIÈRES

1. [Tâche 1: Téléchargement et importation du code d'exemple](#t%C3%82che-1-t%C3%89l%C3%89chargement-et-importation-du-code-dexemple)
2. [Tâche 2: Familliarisation avec le code d'exemple](#t%C3%82che-2-familliarisation-avec-le-code-dexemple)
3. [Tâche 3: Complétion du code d'exemple avec des transactions](#t%C3%82che-3-compl%C3%89tion-du-code-dexemple-avec-des-transactions)
4. [Conclusion](#conclusion)

## TÂCHE 1: TÉLÉCHARGEMENT ET IMPORTATION DU CODE D'EXEMPLE

Dans ce chapitre, nous installons le [code d'exemple](http://heigvd-cld.s3-website-eu-west-1.amazonaws.com/labs/CLD Lab04 DBaaS.d/OrderManagement.zip)
fourni. Ensuite, nous parametrons Eclipse pour acceuillir ce nouveau projet.

L'importation du code source c'est déroulé comme prévu. Notons tout de même, que
nous configurons déjà le serveur distant de GAE.

## TÂCHE 2: FAMILLIARISATION AVEC LE CODE D'EXEMPLE

```xml
<?xml version="1.0" encoding="utf-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns="http://java.sun.com/xml/ns/javaee"
xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" version="2.5">
	<servlet>
		<servlet-name>ProductServlet</servlet-name>
		<servlet-class>com.google.appengine.codelab.ProductServlet</servlet-class>
	</servlet>
	<servlet-mapping>
		<servlet-name>ProductServlet</servlet-name>
		<url-pattern>/product</url-pattern>
	</servlet-mapping>
	<servlet>
		<servlet-name>ItemServlet</servlet-name>
		<servlet-class>com.google.appengine.codelab.ItemServlet</servlet-class>
	</servlet>
	<servlet-mapping>
		<servlet-name>ItemServlet</servlet-name>
		<url-pattern>/item</url-pattern>
	</servlet-mapping>
	<servlet>
		<servlet-name>CustomerServlet</servlet-name>
		<servlet-class>com.google.appengine.codelab.CustomerServlet</servlet-class>
	</servlet>
	<servlet-mapping>
		<servlet-name>CustomerServlet</servlet-name>
		<url-pattern>/customer</url-pattern>
	</servlet-mapping>
	<servlet>
		<servlet-name>OrderServlet</servlet-name>
		<servlet-class>com.google.appengine.codelab.OrderServlet</servlet-class>
	</servlet>
	<servlet-mapping>
		<servlet-name>OrderServlet</servlet-name>
		<url-pattern>/order</url-pattern>
	</servlet-mapping>
	<welcome-file-list>
		<welcome-file>index.html</welcome-file>
	</welcome-file-list>
</web-app>
```

## TÂCHE 3: COMPLÉTION DU CODE D'EXEMPLE AVEC DES TRANSACTIONS

## CONCLUSION
