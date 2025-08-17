---
title: "Real World Runs on Graphs!!📈 : A Hands-On Intro to Neo4j (Part 1)"
date: 2025-07-17T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["Database","Spring boot","graph"]
categories: ["Programming"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
# hidemeta: false
# comments: false
# description: "Desc Text."
# canonicalURL: "https://canonical.url/to/page"
# disableHLJS: true # to disable highlightjs
# disableShare: false
# disableHLJS: false
# hideSummary: false
# searchHidden: true
# ShowReadingTime: true
# ShowBreadCrumbs: true
# ShowPostNavLinks: true
# ShowWordCount: true
# ShowRssButtonInSectionTermList: true
UseHugoToc: true
# cover:
#     image: "<image path/url>" # image path/url
#     alt: "<alt text>" # alt text
#     caption: "<text>" # display caption under cover
#     relative: false # when using page bundles set this to true
#     hidden: true # only hide on current single page
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
---

## Graph database what and how?

Recently at work, I had the opportunity to dive into a new product that deals with highly complex, hierarchical data schemas—deeply interconnected across multiple levels of business domains. To put it simply: imagine an ETL process pulling structured data from physical documents and presenting that data differently for different teams—each with their own use cases and logic.

Now, the fun (and frustration) really began when we start thinking about how to handle these transformations. Some were done using custom functions, others using Java reflection (yes..yes, you read it right, we use java reflection🙃).

We realized that once the volume of data gets bigger and bigger, It will be hard to model it in relational database and the normalization of this data will start feeling like playing Jenga on a rollercoaster. That’s when the team made the call to use a high perfomance graph database offered by a top cloud provider. And let me tell you—we never looked back. 

In this blog series, we’ll be exploring ***Neo4j***—a powerful and developer-friendly graph database that’s trusted by organizations all over the world, including ***NASA***. You can check out how NASA uses Neo4j for their critical data knowledge graph [**here**](https://neo4j.com/blog/knowledge-graph/nasa-critical-data-knowledge-graph/) . But don’t worry—even though NASA uses it, learning Neo4j isn’t rocket science. It’s intuitive, elegant, and surprisingly easy to get started with. We will build a simple yet futuristic social media using spring boot and Neo4j. This will help us practically understand how the graph database work.

`!! I assume that you are a software developer who is familiar with development fundamentals and have prior experince building java services` 

So fasten your seatbelts, fire up the thrusters, and let’s launch into the world of ***Neo4j***! 🚀
![Rocket Launch](https://media4.giphy.com/media/v1.Y2lkPTc5MGI3NjExZXplNTdidmU0eTlyeTkxNHQ2c2R1c2Y5eGZsa3M3c3hqcnI2cTd5YSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/sGBMzyeEzKpySD74qv/giphy.gif)


## Lets build `InstaGrat`📷

We will be building the backend for a new fictional futuristic social media platform called **InstaGrat**, where people can create accounts, follow other users, post content, and also like and comment on other posts (very futuristic, right? 😅). In upcoming blog posts, we will dive into AI concepts like [**RAG (Retrieval-Augmented Generation)**](https://aws.amazon.com/what-is/retrieval-augmented-generation/) and [**MCP (Model Context Protocol)**](https://modelcontextprotocol.io/introduction), and explore how to leverage them to make the best out of our service!
In this blog, we will do the following by starting small:
- Install the database , create spring boot project
- Connect backend to database
- Create Users

### Install the database , create spring boot project

#### ✅ Step 1: Neo4j Installation 📁
- Install **Neo4j Desktop** from [here](https://neo4j.com/download/)  
  OR  
- Use Docker compose:
```yml
version: '3.8'

services:
  neo4j:
    image: neo4j:5.16
    container_name: instagratdb
    ports:
      - "7474:7474"  
      - "7687:7687"  
    environment:
      - NEO4J_AUTH=yourusername/yourpassword          #more than or equals 8 characters
    volumes:
      - neo4j_data:/data

volumes:
  neo4j_data:
```

#### ✅ Step 2: Run docker container 🐳

you can now run the docker compose script. Once the db container starts, access the port `http://0.0.0.0:7474/browser/`, provide the above username password, and login. If everything goes right you will navigate to the neo4j dashboard UI.

If you click at the database icon in the top left of the left panel, we could see there is a default database which is neo4j. We will use this database for this tutorial. But feel free to play with creating new database and changing db configs that suits you. 

**congrats!!** we have the database up and running!! Now lest create a spring boot project from [**spring initializer**](https://start.spring.io/). We will use the following dependencies to create this backend.
- Spring Reactive Web
- Spring Data Neo4j
- Lombok

Once we have ths project imported into an IDE of your choice lets connect the backend to the database.

### Connect backend to database

#### ✅ Step 1: cofigure backend to connect local database 💿

Similar to how we connect spring to any database, we need to configure application.yml file as the following:

```yml
spring:
  neo4j:
    uri: bolt://localhost:7687
    authentication:
      username: yourusername
      password: yourpassword  
```
#### ✅ Step 2: Create User Node and business logic to persist in db

We will not run the backend yet. We will need to create Entities and business logic to propogate data to the database. So lets create simple user entity.
```java
@Node
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    @Id @GeneratedValue
    private Long id;
    private String name;

    @Relationship(type = "FOLLOWS", direction = Relationship.Direction.OUTGOING)
    private List<User> following = new ArrayList<>();
}
```

The class above is quite similar to a typical entity class used in JPA/Hibernate. However, you'll notice some different annotations like ***@Node*** and ***@Relationship***. For now, you can think of these as the building blocks of a graph. Nodes represent the data elements in the graph and hold the properties of the entity. Relationships, on the other hand, define the logical connections between nodes. They also store properties such as the source (from) node and the target (to) node.

We’ll dive deeper into this in the next blog post, where we’ll explore how data is represented in Neo4j. We’ll also implement custom queries to retrieve and add complex data. In addition, we’ll introduce the Cypher query language, which is designed specifically for working with graph data. Cypher is similar to SQL but optimized for expressing graph structures and relationships. With Cypher, you can easily create, read, update, and delete nodes and relationships using a simple and human-readable syntax.

Let's define a Service class and a Repository interface to persist the user:
```java
@Repository
public interface UserRepository extends ReactiveNeo4jRepository<User, Long> {
    Mono<User> findByName(String name);
}
```

There’s nothing fancy here — we’re simply extending the ReactiveNeo4jRepository<T, ID> interface from Spring Data Neo4j and defining a reactive, non-blocking CRUD operation findByName() that returns a Mono of User.

Some of you might be new to reactive programming. I initially thought of covering the basics here, but to be honest, it's a vast topic on its own. It might take some time to get familiar with it at first, but once you get the hang of it, it can do wonders for your backend.

I highly recommend checking out this excellent [**tutorial**](https://medium.com/simform-engineering/deep-dive-into-reactive-programming-with-spring-boot-d62cae63bb03) on Medium for a deeper dive.


As we have created the repository class for the User entity, lets create a service class to add business logic for saving and retreiving the user

```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    public Mono<AppUser> createUser(String name) {
        AppUser user = new AppUser();
        user.setName(name);
        return userRepository.save(user);
    }

    public Flux<AppUser> getAllUsers() {
        return userRepository.findAll();
    }
}
```

So far, we’ve added two methods in our service class — one to create a user and another to fetch all the users from the database. In the next post, we’ll look at how to get a single user along with their connected nodes. We’ll also explore how users can follow or unfollow others, how to like, comment, and create posts. Plus, we’ll dive into how to customize relationships to build better connections between nodes.

Alright, let’s go ahead and create a controller class to expose our API endpoints!

```java
@RestController
@RequestMapping("/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @PostMapping
    public Mono<User> createUser(@RequestParam String name) {
        return userService.createUser(name);
    }

    @GetMapping
    public Flux<User> getAllUsers() {
        return userService.getAllUsers();
    }
}
```

### Lets test calling our backend API ☎️

Now we have the endpoints, lets test it using any http tool, creating a user and also retreiving it.

so lets CURL to create a user node specifying name Gokul.
>`curl -X POST "http://localhost:8080/users?name=Gokul"`

We see the response from backend like this:
```json
{"id":2,"name":"Gokul","following":[]}
```

We could also check this visiting the local Neo4j dashboard at `http://0.0.0.0:7474/browser/`.

Which means that the backend has created a user with name Gokul and with an Id 2. Currently the followers list is empty. But As I said earlier in the next blog we will be going to write custom Cypher query for advanced operations and refactor the code to allign with latest development principles. 

## Conclusion

🎉🥳 Congratulations!! We’ve officially laid the foundation for our social media app! 🚀 It’s just the beginning, and we’ve got a long journey ahead before we can build anything truly exciting and practical. But hey, every big adventure starts with a small step, right? 😉

So, I’ll wrap up this blog here. We’ve covered quite a bit today! You can find all the code for this part here 👉 [**github**](https://github.com/gokintosh/InstaGrat/tree/part1).

The next part of this series will be available as soon as it’s ready. Stay tuned — it’s coming soon!

Until then, may the code be with you. ✨👨‍💻
![yodha](https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExaGx5ZWdvaW5rMDc5bTgwb2dqZTlldnU4dm9vaHp6d3phdGhzZDMxZiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/3o7qDK5J5Uerg3atJ6/giphy.gif)











