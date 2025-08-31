---
title: "Real World Runs on Graphs!!📈 : In the begining there was BROK!! (Part 3(final))"
date: 2025-08-01T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["Database","Spring boot","Graph"]
categories: ["Programming"]
author: "Code by Me, Spelled by ChatGPT"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: true
# hidemeta: false
# comments: false
description: ""
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
## A quick rewind!!

In the previous post, we implemented the follow functionality between users, comment posts, and also like functionality. Also we learned about cypher language and how to use Pinecone vector storage for indexing data for similarity search for our RAG. In this post, we will look into:
- Indexing the post data into our vector store.
- Will create the application with an LLM.
- create the ASK BROK functionality to know various details about the friends posts.

## Large Language Model🤖

**Large Language Models (LLMs)** like Mistral’s ***Open Mistral 7B*** bring the power of natural language understanding and generation into your applications. Instead of manually writing complex rules for summarization, question answering, or sentiment analysis, you can simply provide a prompt and let the model handle the heavy lifting.
For example, if you want to summarize posts and analyze sentiment, a traditional approach would require separate steps:
- Preprocessing text
- Running sentiment analysis via a trained ML model
- Writing custom summarization logic
With an **LLM**, you just send a natural language prompt such as:
"Summarise the posts and analyze the sentiment."
The model understands both tasks in a single request and returns structured, human-like output. This flexibility is why LLMs are increasingly integrated into modern backends.
Choosing Mistral Open Mistral 7B
Mistral AI provides open, production-ready LLMs, including Open Mistral 7B, a lightweight yet powerful model you can run in the cloud through their API. It’s optimized for speed, cost-efficiency, and can be integrated seamlessly into applications like chatbots, analytics pipelines, or backend services.

> Also Mistral is Home made in European Union😀🇪🇺

## Getting Started with Mistral AI
Before you can start using API, you need two things:
- A Mistral AI Account
- An API Key
Step 1: Create a Mistral AI Account
* Go to Mistral AI Console.
* Sign up using your email or GitHub/Google account.
* Once inside, you’ll have access to the developer dashboard.

Step 2: Generate an API Key
* In the Mistral AI Console, navigate to API Keys.
* Click Create API Key.
Copy the generated key — this is what you’ll use to authenticate requests.

[![temp-Image52-Moha.avif](https://i.postimg.cc/j22dMrqj/temp-Image52-Moha.avif)](https://postimg.cc/jnVbjm7V)

⚠️ Important: Keep your API key private. Never hardcode it into frontend code or share it publicly. Store it securely in environment variables.

## Adding the configs to application.yaml file

```yaml
spring:
  application:
    name: instagrat
  ai:
    mistralai:
      apiKey: ${Your Api Key for Mistral}
      model: open-mistral-7b
    vectorstore:
      pinecone:
        apiKey: ${Your Pinecone API key}
        environment: ${your environment id}
        projectId: ${your project id}
        indexName: ${your index name}

  neo4j:
    uri: bolt://localhost:7687
    authentication:
      username: neo4j
      password: demo

```

Add required dependencies to gradle:

```groovy
dependencyManagement {
	imports {
		mavenBom "org.springframework.ai:spring-ai-bom:$springAiVersion"
	}
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-neo4j'
	implementation 'org.springframework.boot:spring-boot-starter-webflux'
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testImplementation 'io.projectreactor:reactor-test'
	testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
	
	implementation("org.springframework.ai:spring-ai-pinecone-store-spring-boot-starter:1.0.0-M6")
	implementation("org.springframework.ai:spring-ai-autoconfigure-vector-store-pinecone:1.0.0-M8")

	implementation 'org.springframework.ai:spring-ai-mistral-ai-spring-boot-starter'

}
```

once we add the dependencies, we need to add java code for getting the post for a user with userId to store in vector datanbase from the neo4j database:

```java
    @GetMapping("/getposts/{userId}")
    Mono<Boolean> getAllPosts(@PathVariable Long userId){
        return postRepository.getAllPostsForUser(userId)
                .collectList().map(post-> {
            try {
                return Document.builder().id(userId.toString()).text(mapper.writeValueAsString(new UserPostLists(userId,post))).build();

            } catch (JsonProcessingException e) {
                throw new RuntimeException(e);
            }
        }).flatMap(docs->Mono.fromCallable(()->{
            vectorStore.add(List.of(docs));
            return true;
                }).subscribeOn(Schedulers.boundedElastic())

                );


    }
```

After executing this, we will have the posts indexed for the user in the pinecone db.

![temp-Image-Rh-GBh-N.avif](https://imgpx.com/NfxFy3KXCLs2.png)


Now we need to implement and endpoint for prompting BROK for Augment Retreiving the data from vector storage.

```java
 @RequestMapping("/askbrok/{userId}")
    Mono<String> askBrok(@PathVariable String userId, @RequestParam String prompt){


        PromptTemplate pt=new PromptTemplate("""
                {query}.
                Only consider posts from user {userId}.
                Summarise their latest posts clearly.
                """);

        Prompt p=pt.create(
                Map.of("query",prompt,
                        "userId",userId
                )
        );

//        SearchRequest searchRequest = SearchRequest.builder().query(prompt).filterExpression(Map.of("userId", userId)).build();

        return Mono.fromCallable(()->
                this.chatClient.prompt(p)
                        .advisors(new QuestionAnswerAdvisor(
                                store
                        )).call()
                        .content()
                ).subscribeOn(Schedulers.boundedElastic());
    }

```

once we have the functionality set up, if we hit the endpoint with the following prompt, the output we get is:
![temp-Imageb-Tainr.avif](https://screenshot-2025-08-31-at-17-42-17.tiiny.site/Screenshot-2025-08-31-at-17-42-17.png)


## Conclusion

🚦 Wrapping Up (Finally 😅)

Whew! If you’ve made it this far — hats off 🧢 to you. This post is already clocking in at over 10+ minutes of reading, and we’ve covered a lot:
- Built a basic social media backend with Neo4j and Spring WebFlux
- Implemented core features like post creation, comments, and likes
- Connected the dots for vector storage using Pinecone and OpenAI embeddings

I initially planned to introduce [Spring AI](https://spring.io/projects/spring-ai) here too… but that would turn this post into a mini eBook! So, let’s hit pause for now and leave that for the next chapter. If you’re curious, check out the [Spring AI documentation](https://docs.spring.io/spring-ai/reference/) for a sneak peek at what’s coming next.


🚀 What’s Next?

In the next post, we’ll roll up our sleeves and build our own “poor man’s GROK”— we  will name it **BROK** 😄.

----

Thanks for following along! 🚀  
Keep experimenting, building, and sharing your progress. So, I’ll wrap up this blog here. We’ve covered quite a bit today! You can find all the code for this part here 👉 [**github**](https://github.com/gokintosh/InstaGrat/tree/rag-data-seeding).

The next part of this series will be available as soon as it’s ready. Stay tuned — it’s coming soon!

---


![Geek Celebration GIF](https://media.giphy.com/media/l0MYt5jPR6QX5pnqM/giphy.gif)

Happy coding! 🧑‍💻✨