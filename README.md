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

Dans ce chapitre, nous étudions le code d'exemple fourni.

En inspectant le fichier `war/WEB-INF/lib/web.xml" nous constatons la présence
de plusieurs points d'entrée.

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

Nous comptabilisons 4 Servlets. Ces Servlets sont destinés à être employés pour
manipuler les données de l'application. Nous résumons les actions dans le
tableau suivant:

| Servlet         | Chemin d'accès | Action(s)              |
| --------------- | -------------- | ---------------------- |
| ProductServlet  | /product       | GET, PUT, DELETE, POST |
| ItemServlet     | /item          | GET, PUT, DELETE, POST |
| CustomerServlet | /customer      | GET, PUT, DELETE, POST |
| OrderServlet    | /order         | GET, PUT, DELETE, POST |

En regardant le code source, nous constatons la présence de deux Servlets
supplémentaires: **BaseServlet** et **ContactServlet**.

| Servlet        | Chemin d'accès | Action(s)              |
| -------------- | -------------- | ---------------------- |
| BaseServlet    | n/a            | n/a                    |
| ContactServlet | n/a            | GET, PUT, DELETE, POST |

**BaseServlet** est utilisé comme une super-classe de tout les autres Servlets.
Seule la méthode **doGet()** est implantée. Il a pour fonction, de paramètrer
les entêtes HTTP.

**ContactServlet** est utilisé pour gérer le système de "contacts" de
l'application. Aucun point d'entré (chemin d'accès) ne lui a été assigné.

À présent, nous nous concentrons sur la manière dont a été implanté la gestion
de la clientèle. Plus particulièrement, au moment où il y a des écritures dans
le Datastore. Pour ce faire nous portons notre attentions sur la méthode
**doPut()**.

```java
/**
 * Insert the new customer
 */
protected void doPut(HttpServletRequest req, HttpServletResponse resp)
		throws ServletException, IOException {
	logger.log(Level.INFO, "Creating Customer");
	String nameKey = req.getParameter("name");
	String firstName = req.getParameter("firstName");
	String lastName = req.getParameter("lastName");
	String phone = req.getParameter("phone");
	String address = req.getParameter("address");
	String city = req.getParameter("city");
	String state = req.getParameter("state");
	String zip = req.getParameter("zip");
	String email = req.getParameter("eMail");
	Customer.createOrUpdateCustomer(nameKey, firstName, lastName, phone,
			address, city, state, zip, email);
}
```

Nous remarquons qu'à l'épilogue de la fonction, la méthode
**createOrUpdateCustomer()** de la classe **Customer** est appelée. Nous
regardons comment elle a été implanté.

```java
/**
 * Checks if the entity is existing and if it is not, it creates the entity
 * else it updates the entity
 * 
 * @param nickName
 *          : username for the customer
 * @param firstName
 *          : first name of the customer
 * @param lastName
 *          : last name of the customer
 * @param address
 *          : address of customer
 * @param phone
 *          : contact phone number
 * @param email
 *          : email id of customer
 */
public static void createOrUpdateCustomer(String nickName, String firstName,
		String lastName, String phone, String address, String city, String state, String zip, String email) {
	Entity customer = getSingleCustomer(nickName);
	if (customer == null) {
		customer = new Entity("Customer", nickName);
		customer.setProperty("name", nickName);
		customer.setProperty("firstName", firstName);
		customer.setProperty("lastName", lastName);
		customer.setProperty("phone", phone);
		customer.setProperty("address", address);
		customer.setProperty("city", city);
		customer.setProperty("state", state);
		customer.setProperty("zip", zip);
		customer.setProperty("eMail", email);
	} else {
		if (firstName != null && !"".equals(firstName)) {
			customer.setProperty("firstName", firstName);
		}
		if (lastName != null && !"".equals(lastName)) {
			customer.setProperty("lastName", lastName);
		}
		if (phone != null && !"".equals(phone)) {
			customer.setProperty("phone", phone);
		}
		if (address != null && !"".equals(address)) {
			customer.setProperty("address", address);
		}
		if (city != null && !"".equals(city)) {
			customer.setProperty("city", city);
		}
		if (state != null && !"".equals(state)) {
			customer.setProperty("state", state);
		}
		if (zip != null && !"".equals(zip)) {
			customer.setProperty("zip", zip);
		}
		if (email != null && !"".equals(email)) {
			customer.setProperty("eMail", email);
		}
	}
	Util.persistEntity(customer);
}
```

Cette méthode vérifie l'existence d'un client et selon la réponse, l'ajoute ou
le modifie. Une fois encore, une nouvelle méthode est appelée,
**persistEntity()** de la classe **Util** pour mettre à jours l'entitée.

```java
/**
 * Add the entity to cache and also to the datastore
 * @param entity
 *          : entity to be persisted
 */
public static void persistEntity(Entity entity) {
	logger.log(Level.INFO, "Saving entity");
	Key key = entity.getKey();
	// TODO: add a transaction
	datastore.put(entity);
	addToCache(key, entity);
}
```

Nous constatons que c'est dans cette méthode que l'algorithme d'écriture des
données dans le Datastore, est implanté.

## TÂCHE 3: COMPLÉTION DU CODE D'EXEMPLE AVEC DES TRANSACTIONS

## CONCLUSION
