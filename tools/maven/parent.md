

## What does the parent tag in Maven pom represent?

Yes, maven reads the parent POM from your local repository (or proxies like nexus) and creates an 'effective POM' by merging the information from parent and module POM.

See also Introduction to the POM

One reason to use a parent is that you have a central place to store information about versions of artifacts, compiler-settings etc. that should be used in all modules.

To see the combined result use the following mvn command:

```
mvn help:effective-pom  

```
