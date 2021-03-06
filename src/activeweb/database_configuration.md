<ol class=breadcrumb>
   <li><a href=/>JavaLite</a></li>
   <li><a href=/activeweb>ActiveWeb</a></li>
   <li class=active>Database configuration</li>
</ol>
<div class=page-header>
   <h1>Database configuration <small></small></h1>
</div>



ActiveWeb *does not use* property files, XML, Yaml or any other text files for configuration. Configuration
is done in Java code. There are a few advantages of this approach: auto-suggestions by IDE, documentation at
finger tips, and some level of comfort from Java compiler.

ActiveWeb uses two application classes for configuration: `app.config.AppControllerConfig` and `app.config.DbConfig`.
Both of these classes are initialized from a third application level class called `app.config.AppBootstrap`.

## Database connections' configuration

In order to configure database connection, an application needs to provide a class called `app.config.DbConfig`.
It is used to configure database connections for various **environments and modes**.

## What is an environment?

An ActiveWeb environment is a computer where a project executes. In the process of software development there can be a
number of environments where a project gets executed, such as development, continuous integration, QA, staging,
production and more. The number of environments for ActiveWeb is custom for every project. However, it is typical to
have four: development, continuous integration, staging and production

## How to specify an environment

An environment is specified by an environment variable: `ACTIVE_ENV` Every computer where an ActiveWeb project gets executed, needs to have this variable specified. This value is used to determine which DB connections need to be initialized.

## Default environment

In case an environment variable `ACTIVE_ENV` is not provided, the framework defaults to "development".

## What is a mode?

ActiveWeb defines two modes of operation: "standard", which is also implicit, and "testing". Standard mode is used
during regular run of the program, and testing used during the build when tests are executed. ActiveWeb promotes a style
of development where one database used for testing, but a different one used under normal execution. When tests are
executed, a "test" database is used, and when a project is run in a normal mode, a "development" database is used.
Having sa separate database for testing ensures safety of data in the development database.

## Property file configuration

You can create a file called `database.properties` and place it on classpath or somewhere on the file system. 

The content of the file can be similar to: 

```
development.driver=com.mysql.jdbc.Driver
development.username=root
development.password=pwd1
development.url=jdbc:mysql://localhost/simple_development

test.driver=com.mysql.jdbc.Driver
test.username=root
test.password=pwd2
test.url=jdbc:mysql://localhost/simple_test

production.driver=com.mysql.jdbc.Driver
production.username=root
production.password=pwd2
production.url=jdbc:mysql://localhost/simple_test
```

The configure the class `app.config.DbConfig`: 


```java
public class DbConfig extends AbstractDBConfig {
    public void init(AppContext context) {
        configFile("/database.properties");
        environment("production", true).jndi("jdbc/simple_production");        
    }
}
```

The property file configuration works well with [Database Migration](database_migrations) system, so that 
configuration of database connections is done only in one place. 

The last line in  `app.config.DbConfig` overrides "production" cofiguration from the file. The "production" configuration 
in the file can be used by migration plugin, while the app itself will be using a connection from a pool provided by 
container. 

> Please, see a working example of file based configuration in the [ActiveWeb Simple](https://github.com/javalite/activeweb-simple/) project.

## Java code configuration

~~~~ {.java  }
public class DbConfig extends AbstractDBConfig {
    public void init(AppContext context) {
         environment("development").jndi("jdbc/kitchensink_development");
         environment("development").testing().jdbc("com.mysql.jdbc.Driver", "jdbc:mysql://localhost/kitchensink_development", "root", "****");
         environment("jenkins").testing().jdbc("com.mysql.jdbc.Driver", "jdbc:mysql://172.30.64.31/kitchensink_jenkins", "root", "****");
         environment("production").jndi("java:comp/env/jdbc/myproject_production);
    }
}
~~~~

The code above is borrowed from [Kitchensink](https://github.com/javalite/kitchensink) project.
Lets examine it line by line.

-   **Line 3**: here we provide configuration for a "standard" mode in "development" environment. This DB connection will
be used when the application is running under normal conditions in development environment.
-   **Line 4**: This is a configuration of DB connection for "development" environment, but for "testing" mode. This
connection will be used by unit and integration tests during the build.
-   **Line 5**: This is a configuration of DB connection for "jenkins" environment, but for "testing" mode.
The "jenkins" environment is a computer where this project is built by Jenkins - the continuous integration server.
Since Jenkins computer is fully automated, and this project is not running there in "standard" mode, there is no
standard configuration for jenkins environment, just one for testing.
-   **Line 6**: This is configuration similar to one on line 3, but for "production" environment.

> Configuration of database connections is just that - configuration. This code only configures a connection, but
does not open it. To open a connection, you need to use [DBConnectionFilter](https://github.com/javalite/activeweb/blob/master/activeweb/src/main/java/org/javalite/activeweb/controller_filters/DBConnectionFilter.java#DBConnectionFilter).
 For more, see [Controller filters](controller_filters).
