# Why Quick?

## Coming from &lt;cfquery&gt; or queryExecute?

You might be thinking, I don't need an ORM engine.  I don't even know what ORM means!  I know SQL backwards and forwards so there's nothing an ORM can offer me.  Maybe you've had experience with other ORM engines, whether CFML-based or not, and the experience was less that ideal.  Why should you consider Quick?

Quick's ORM philosophy comes down to three main points:

1. Give relevant names to important collections of SQL code. \([scopes](guide-1/getting-started/query-scopes-and-subselects.md#what-are-scopes), [relationships](guide-1/relationships/), etc.\)
2. Make queries easy to compose at runtime to get the exact data you want in the most efficient way \([subselects](guide-1/getting-started/query-scopes-and-subselects.md#subselects), [eager loading](guide-1/relationships/eager-loading.md), etc.\)
3. Get out of your way when you need or want to write barebones SQL.

## Coming from Hibernate?

Quick was built out of lessons learned and persistent challenges in developing complex RDBMS applications using built-in Hibernate ORM in CFML.

* Hibernate ORM error messages often obfuscate the actual cause of the error

  because they are provided directly by the Java classes.

* Complex CFML Hibernate ORM applications can consume significant memory and

  processing resources, making them cost-prohibitive and inefficient when used

  in microservices architecture.

* Hibernate ORM is tied to the engine releases. This means that updates come

  infrequently and may be costly for non-OSS engine users.

* Hibernate ORM is built in Java. This limits contributions from CFML

  developers who don't know Java or don't feel comfortable contributing to a

  Java project.

* Hibernate ORM doesn't take advantage of a lot of dynamic- and

  meta-programming available in CFML. \(Tools like CBORM have helped to bridge

  this gap.\)

We can do better.



