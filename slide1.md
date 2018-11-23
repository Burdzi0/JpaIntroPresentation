# JPA

## Quick intro

---

## What is JPA?

The Java Persistence API is a specification for:
* accessing <!-- .element: class="fragment" data-fragment-index="0" -->
* persisting <!-- .element: class="fragment" data-fragment-index="1" -->
* and managing <!-- .element: class="fragment" data-fragment-index="3" -->

data between Java objects / classes and a relational database <!-- .element: class="fragment" data-fragment-index="4" -->

---

## Why JPA?

JPA is now considered the **standard** industry approach for Object to Relational Mapping (ORM) in the Java Industry.

---

## But wait...

JPA itself is just a specification, not a product.
It **cannot** perform persistence or anything else by itself.
JPA is just a set of interfaces and **requires** an implementation

---

## How am I going to use it?

A provider is needed.
There are several:
* Hibernate <!-- .element: class="fragment" data-fragment-index="0" -->
* EclipseLink <!-- .element: class="fragment" data-fragment-index="1" -->
* OpenJPA <!-- .element: class="fragment" data-fragment-index="2" -->

Hibernate is used in more than 70% of projects <!-- .element: class="fragment" data-fragment-index="3" -->

Use Gradle or Maven, it's that simple <!-- .element: class="fragment" data-fragment-index="4" -->
---

## All we need is a single file

![persistence.xml](persistenceFile.png)

assuming you already have your database set up... <!-- .element: class="fragment" data-fragment-index="0" -->

---

## Persistence.xml

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<persistence ... version="2.1">
  <persistence-unit name="h2" transaction-type="RESOURCE_LOCAL">
    <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
      <properties>
        <property name="javax.persistence.jdbc.driver"
        value="org.h2.Driver"/>
        <property name="javax.persistence.jdbc.url"
        value="jdbc:h2:mem:testdb"/>
        <property name="hibernate.dialect"
        value="org.hibernate.dialect.H2Dialect"/>
        <property name="hibernate.hbm2ddl.auto" 
        value="update"/>
      </properties>
  </persistence-unit>
</persistence>
```

---

## POJO
```java
public class Author {

	private String firstName;
	private String lastName;

	// Constructors, getters and setters..
```

---

## Entity

```java
@Entity 
public class Author {

	@Id 
	@GeneratedValue(strategy = GenerationType.AUTO)
	private Long id;

	private String firstName;
	private String lastName;

	public Author() {
	}
	// Constructors, getters and setters..
```

---

## Persistence classes

```java
EntityManagerFactory factory = 
	Persistence.createEntityManagerFactory("h2");
```
<!-- .element: class="fragment" data-fragment-index="0" -->
```java
EntityManager manager = factory.createEntityManager();
```
<!-- .element: class="fragment" data-fragment-index="1" -->
```java
EntityTransaction transaction = manager.getTransaction();
```
<!-- .element: class="fragment" data-fragment-index="2" -->

---

## Persist entity

```java
Author author = new Author("Tom", "Clancy");
transaction.begin();
manager.persist(author);
transaction.commit();
```

---

## Find entity

```java
Long authorId = 1L;
Author author = manager.find(Author.class, authorId);
```
but something's wrong... <!-- .element: class="fragment" data-fragment-index="0" -->

author can be null! <!-- .element: class="fragment" data-fragment-index="1" -->

---

## Update entity using merge

```java
transaction.begin();
Author author = new Author("Tom", "Clancy");
manager.persist(author);

Author authorToBeFound = new Author();
authorToBeFound.setId(1L);
authorToBeFound.setFirstName("Arnold");
authorToBeFound.setLastName("Schwarzenegger");

transaction.begin();
Author found = manager.merge(authorToBeFound);
transaction.commit();
```

---

## Update entity using merge - generated SQL

```java
Hibernate: 
insert into Author (id, first_name, last_name)
	values (null, ?, ?)

Author{id=1, firstName='Tom', lastName='Clancy'}

Hibernate:
update Author 
	set 
		first_name=?,
		last_name=?
where id=?

Author{id=1, firstName='Arnold', lastName='Schwarzenegger'}
```

---

## Update entity using update

```java
Long authorId = 1L;
author = manager.find(Author.class, authorId);

transaction.begin();
author.setId(1L);
author.setFirstName("Arnold");
author.setLastName("Schwarzenegger");
transaction.commit();
```

---

## Update entity using update - generated SQL

### Exaclty the same!

---

## Remove entity 

```java
Optional.ofNullable(manager.find(Author.class, authorId))
	.ifPresent(found -> {
	transaction.begin();
	manager.remove(found);
	transaction.commit();
});

```

---

## Remove entity using merge

```java
Author toBeRemoved = new Author();
toBeRemoved.setId(1L);
transaction.begin();
manager.remove(manager.merge(toBeRemoved));
transaction.commit();
```

---
