# RedisJSON Jedis Quick Start

Jedis supports [RedisJSON](https://oss.redis.com/redisjson/) and [RediSearch](https://oss.redis.com/redisearch/).

The latest versions of RedisJSON let you store, manipulate, index, and query JSON.
To use these features with Jedis, you'll need to use the `UnifiedJedis` interface
or a sub-class of it.

Let's see how this works.

## Connecting with UnifiedJedis

First, let's create a `UnifiedJedis` instance:

```java
UnifiedJedis client = new JedisPooled(Protocol.DEFAULT_HOST, 6479);
```
Now we can start working with JSON. For these examples, we'll be using [GSON]
(https://github.com/google/gson) to handle the serialization of POJOs to JSON.

## Creating JSON documents

Suppose we're building an online learning platform, and we want to represent
students. Let's create a POJO to represent our students:

```java
private class Student {
    private String firstName;
    private String lastName;

    public Student(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    public String getFirstName() {
      return firstName;
    }

    public String getLastName() {
      return lastName;
    }
}
```

Now we can create some students and store them in Redis as JSON":

```java
Student maya = new Student("Maya", "Jayavant");
client.jsonSet("student:111", maya);

Student oliwia = new Student("Oliwia", "Jagoda");
client.jsonSet("student:112", oliwia);
```

## Querying and indexing JSON

If we want to be able to query this JSON, we'll need to create an index. Let's
create an index on the "firstName" and "lastName" fields.

1. We define which fields to index ("firstName" and "lastName").
2. We set up the index definition to recognize JSON and include only those
documents
whose key starts with "student:".
3. Then we actually create the index, called "student-index", by calling `ftCreate
()`.

```java
Schema schema = new Schema().addTextField("$.firstName", 1.0).addTextField("$" +
            ".lastName", 1.0);
IndexDefinition rule = new IndexDefinition(IndexDefinition.Type.JSON)
        .setPrefixes(new String[]{"student:"});
client.ftCreate("student-index",
            IndexOptions.defaultOptions().setDefinition(rule),
            schema);
```

With an index now defined, we can query our JSON. Let's find all students whose
name begins with "maya":

```java
Query q = new Query("@\\$\\" + ".firstName:maya*");
SearchResult mayaSearch = client.ftSearch("student-index", q);
```

We can then iterate over our search results:

```java
List<Document> docs = mayaSearch.getDocuments();
for (Document doc : docs) {
   System.out.println(doc);
}
```

This example just scratches the surface. You can atomically manipulate JSON documents and query them in a variety of ways. See the [RedisJSON docs](https://oss.redis.com/redisjson/), the [RediSearch](https://oss.redis.com/redisearch/) docs, and our course, ["Querying, Indexing, and Full-text Search in Redis"](https://university.redis.com/courses/ru203/), for a lot more examples.
