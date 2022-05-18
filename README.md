# Payara 5 -> Payara 6 Migration

## Background

Payara 6 will be released in Q2 2002, after which Payara 5 Community Edition will not receive security updates. Payara 6 will only support Jakarta EE 10 and Java 11, so we need to upgrade all our components.

## Process

We will do this in 2 phases:
 - Phase 1: move from Java 8 and JavaEE 7 (Payara 5) to Java 11 and JakartaEE 9.1 (Payara 6-Alpha)
 - Phase 2: move from JakartaEE 9.1 (Payara 6-Alpha) to JakartaEE 10 (Payara 6)

This is because JakartaEE 10 and Payara 6 have not yet been releaed.

## Components

Component           | Java 11 | JakartaEE 9.1 | JakartaEE 10 | New version
---                 | ---     | ---           | ---          | ---
icat.utils          | N/A     | N/A           | N/A          | N/A
icat.authentication | Done    | Done          |              | 5.0.0-SNAPSHOT
icat.server         | Done    | Done          |              | 6.0.0-SNAPSHOT
icat.client         | Done    | Done          |              | 6.0.0-SNAPSHOT
icat.lucene         | Done    | Done          |              | 2.0.0-SNAPSHOT
authn.anon          | Done    | Done          |              | 3.0.0-SNAPSHOT
authn.db            | Done    | Done          |              | 3.0.0-SNAPSHOT
authn.ldap          | Done    | Done          |              | 3.0.0-SNAPSHOT
authn.simple        | Done    | Done          |              | 3.0.0-SNAPSHOT
authn.uows_clf      |         |               |              | 
authn.ip            |         |               |              | 
clf-ids             |         |               |              | 
ecat2               |         |               |              | 

## Instructions – Phase 1 

 - First, build the component using Java 8 without making any changes. This ensures you have all the required dependencies including servers running other components needed for WSDLs or for tests.

 - Create a new branch called `payara6`.

 - Increment the major version number in `pom.xml`.

 - Change the java version in the maven-compiler-plugin. You may also need to increase the version of the maven plugin itself.
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.10.1</version>
    <configuration>
        <release>11</release>
    </configuration>
</plugin>
```
Note that we are using `<release>` instead of `<source>` and `<target>`. See here for why: https://maven.apache.org/plugins/maven-compiler-plugin/examples/set-compiler-release.html 

 - Change the javaee-api dependency to:
```
<dependency> 
    <groupId>jakarta.platform</groupId> 
    <artifactId>jakarta.jakartaee-api</artifactId> 
    <version>9.1.0</version> 
    <scope>provided</scope> 
</dependency> 
```
Note: `<scope>provided</scope>` is correct. However, changing the scope may break some execution steps; if so fix them properly – don't remove `<scope>provided</scope>`.

 - Update dependencies for other icat components (they should already have been updated to jakartaee-api 9.1.0).

 - Find any other dependencies from JavaEE and convert them to the appropriate versions. These are typically implementation packages so ensure that they have `<scope>test</scope>` since Payara will supply the implementation. You can find the 'correct' version by looking for the version that depends on the same -api version as the jakartaee-api packages:
   - https://mvnrepository.com/artifact/jakarta.platform/jakarta.jakartaee-api/9.1.0 
   - https://mvnrepository.com/artifact/jakarta.platform/jakarta.jakartaee-web-api/9.1.0 

 - Change imports from `javax` to `jakarta`. Some things are still in the `javax` namespace so change them back after:
```
find . -name '*.java' | xargs sed -i 's/import javax\./import jakarta./' 
find . -name '*.java' | xargs sed -i 's/import jakarta\.naming\./import javax.naming./' 
find . -name '*.java' | xargs sed -i 's/import jakarta\.xml\.datatype\./import javax.xml.datatype./' 
find . -name '*.java' | xargs sed -i 's/import jakarta\.xml\.namespace\./import javax.xml.namespace./' 
```

 - Run `mvn install` to build the component and run the test suite. Fix any minor issues. If there are major issues, create a card on the project board.

 - Test the component in Payara 6-Alpha.

 - Run `mvn deploy` to upload the package to the repo. Ask M.R. for credentials.
