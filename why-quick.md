# Why Quick?

## Coming from &lt;cfquery&gt; or queryExecute?

You might be thinking, I don't need an ORM engine.  I don't even know what ORM means!  I know SQL backwards and forwards so there's nothing an ORM can offer me.

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



