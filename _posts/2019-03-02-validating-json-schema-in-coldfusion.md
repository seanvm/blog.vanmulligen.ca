---
layout: post
title: "Validating JSON in ColdFusion using JSON Schema"
date: 2019-03-03
comments: true
time: 12
---

<span class="dropcap">W</span>hen building any sort of API, proper documentation and consistency are two of the most important aspects. When building JSON based web services, [JSON Schema](https://json-schema.org/){:target="\_blank"} can be a vital component in the developer's toolkit to validate both requests and responses and ensure that the API contract is upheld.

Using JSON Schema in conjunction with a library for validation, you can easily and quickly determine whether a request to your API is missing required fields, or conforms to your API's required format. Similarly, JSON Schema can be very beneficial when running automated tests on your API in a CI framework. If you are not familiar with JSON Schema, I highly suggest you check out ["Understanding JSON Schema"](https://json-schema.org/understanding-json-schema/){:target="\_blank"}. The rest of this post will assume that you have a JSON Schema file ready to validate against.

Recently, I decided to attempt implementing JSON Schema validation within a legacy ColdFusion application I was working on. Unfortunately, ColdFusion has no native libraries or tools available for validating JSON against a JSON Schema file. However, ColdFusion does allow developers to integrate and load Java libraries, which opens up the possibilities of using a Java-based JSON Schema validation library.

There are a few Java libraries available for validating JSON against schema. After some research, I decided [https://github.com/everit-org/json-schema][everit-json-schema-github]{:target="\_blank"} would be the best tool for for this particular implementation. This decision was based partially on how simple the interface for the validator appeared to be, but also due to its purported speed, as well as its support for JSON Schema Draft 6 / 7. Your needs may necessitate using a different library.

Fortunately, ColdFusion makes it quite simple to define a directory where your JAR files live (the classpath). Assuming you have your classpath defined already, you would place the JAR file for the `org.everit.json.schema` package inside your classpath. Because this library has a dependency on the `org.json` and `handy-uri-templates` packages, you would need to place them in your classpath as well.

Downlad links to both packages on Maven:

- [org.everit.json.schema][maven-everit]{:target="\_blank"}
- [org.json][maven-json]{:target="\_blank"}
- [handy-uri-templates][uri-templates]{:target="\_blank"}

_Note_: If adding to the ColdFusion classpath is not an option for you, an alternative is to use Mark Mandel's [JavaLoader library][javaloader]{:target="\_blank"}. However, in my experience, this only works if the library you are loading is packaged in an uber-jar (aka a super-jar) with all of its dependencies. Because `org.everit.json.schema` is not packaged with its dependencies, I decided to create a Java class that handles the implementation of the validator. I have packaged that class along with its dependencies in a JAR so that it can be used with Mark Mandel's JavaLoader library. If adding JAR files into your classpath seems like it might be an issue, you may want to skip ahead to ["Alternative Implementation with JavaLoader"](#alternative-implementation-with-javaloader). Otherwise, go ahead and read the next section.

##### Implementation:

While integrating the `org.everit.json.schema` package into ColdFusion, I discovered a few limitations of using Java classes.

The quickstart guide for the validator seems straightforward:

{% gist ce0d950f4bee29331f7a1269a28c1617 quickstart.java %}

In cfscript, this might roughly look something like this:

{% gist ce0d950f4bee29331f7a1269a28c1617 quickstart.cfm %}

Unfortunately, when loading my schema file as a FileInputStream and passing it to the JSONTokener, I received this error:

`Unable to find a constructor for class org.json.JSONTokener that accepts parameters of type ( java.io.FileInputStream ).`

From what I can tell, even though a `FileInputStream` is an `InputStream`, ColdFusion is unable to cast it as such. This is strange, but not a blocker.

Thankfully we can remove that step altogether and just pass JSON content directly to the JSONObject constructor:

{% gist ce0d950f4bee29331f7a1269a28c1617 rawSchema.cfm %}

At this point, the validation should work. If there are any schema errors `schema.validate()` will throw an exception with details of the validation error, otherwise, it won't return anything.

A component for handling this validation might look something like this:

{% gist ce0d950f4bee29331f7a1269a28c1617 JSONSchema.cfc %}

Then, your implementation might look something like this: 

{% gist ce0d950f4bee29331f7a1269a28c1617 implementation.cfm %}

If there is an error, `results.error` will be a struct containing a list of the errors, in an array. The error messages look like this:

`#/data: required key [name] not found`

If you are validating incoming requests to your API, these error messages can easily be parsed and returned back to the consumer of your API.

##### Alternative Implementation with JavaLoader:

In some scenarios, adding items to the classpath or modifying the classpath might be difficult or require infrastructure changes. In such cases, use the JavaLoader library might be your only option. I have created a JAR file that wraps the `org.everit.json.schema` package. This package handles the implementation of the validator and includes its dependencies in an uber-jar so that it can be used with Mark Mandel's JavaLoader library. That being said, if it is at all possible, I would recommend adding the JARs to your classpath and loading up the Java classes directly as that will provide more control over the output of the validator.

Setup and implementation using this method are quite simple. You will need to download the JavaLoader CFCs as well as the JAR file for the JSON Schema Validation class.

Download links:
* JavaLoader: [https://github.com/markmandel/JavaLoader](https://github.com/markmandel/JavaLoader){:target="\_blank"}
* JSON Schema Validation Package: [https://github.com/seanvm/json-schema-validator](https://github.com/seanvm/json-schema-validator){:target="\_blank"}

Your component might look something like this:

{% gist ce0d950f4bee29331f7a1269a28c1617 JSONSchemaJavaLoader.cfc %}

You can then call the validation function similarly to how we did it above. The main difference being that the `ca.vanmulligen.json.schema.Validator` class returns JSON as it handles the `ValidationException` itself, so there is no need for a `try/catch`.

{% gist ce0d950f4bee29331f7a1269a28c1617 javaLoaderImplementation.cfm %}

The `isValid()` function will return a JSON string with two keys: `valid` and `error`. The `valid` key will be set to a boolean, and `error` will be the output from the `ValidationException` that is being caught. This is the result of calling `toJSON()` on the `ValidationException` as documented here: [https://github.com/everit-org/json-schema#json-report-of-the-failures](https://github.com/everit-org/json-schema#json-report-of-the-failures).

In case you are curious what the wrapper class actually does, I have included the code below:

{% gist ce0d950f4bee29331f7a1269a28c1617 Validator.java %}

You should now be armed with the knowledge you need to implement a JSON Schema validator with your ColdFusion application!

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]: http://jekyllrb.com
[javaloader]: https://github.com/markmandel/JavaLoader
[everit-json-schema-github]: https://github.com/everit-org/json-schema
[maven-everit]: http://central.maven.org/maven2/org/everit/json/org.everit.json.schema/1.5.1/
[uri-templates]: https://mvnrepository.com/artifact/com.damnhandy/handy-uri-templates
[maven-json]: http://central.maven.org/maven2/org/json/json/20180813/
