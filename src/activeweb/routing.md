Title: Learn ActiveWeb routing

[Introduction](#Introduction)

[Standard routing](#Standard_routing)

[RESTful routing](#RESTful_routing)

[Routing with packages](#Routing_with_packages)

[Mapping paths to controller names](#Mapping_paths_to_controller_names)

[Custom routing](#Custom_routing)

-   [Custom routing configuration](#Custom_routing_configuration)
-   [Built-in segments](#Built_in_segments)
-   [Static segments](#Static_segments)
-   [User/dynamic segments](#User_dynamic_segments)
-   [Http method - based routing](#Http_method___based_routing)
-   [RouteConfig reloaded](#_RouteConfig_reloaded)

Introduction
------------

Routing in ActiveWeb is an act of matching an incoming request URL to a controller and action. Current implementation supports built-in standard routing, built-in REST - based routing as well as custom routing.

Standard routing
----------------

NOTE: the "context" in all URIs is a web application context, which is usually a WAR file name.

  ------------------------- --------------------------------- ------------ --------
  **path**                  **controller**                    **action**   **id**
  /context/books            app.controllers.BooksController   index        
  /context/books/save       app.controllers.BooksController   save         
  /context/books/save/123   app.controllers.BooksController   save         123
  ------------------------- --------------------------------- ------------ --------

\
 In standard routing, the HTTP method is not considered, but you might get an exception of you send an HTTP method to an action that is configured for a different HTTP method. Routing and action HTTP methods are independent in case of standard routing. For standard routing, there is no need to do anything, it works by default

RESTful routing
---------------

In case of restful routing, the actions are pre-configured. RESTful routing is configured by placing a @RESTfull annotation on a controller. For more informaiton, see: [ControllerExplained\#RESTful\_controllers](ControllerExplained#RESTful_controllers)

  ----------------- ---------------------- --------------------------------- ------------ ---------------------------------------------
  **HTTP method**   **path**               **controller**                    **action**   **used for**
  GET               /books                 app.controllers.BooksController   index        display a list of all books
  GET               /books/new\_form       app.controllers.BooksController   new\_form    return an HTML form for creating a new book
  POST              /books                 app.controllers.BooksController   create       create a new book
  GET               /books/id              app.controllers.BooksController   show         display a specific book
  GET               /books/id/edit\_form   app.controllers.BooksController   edit\_form   return an HTML form for editing a books
  PUT               /books/id              app.controllers.BooksController   update       update a specific book
  DELETE            /books/id              app.controllers.BooksController   destroy      delete a specific book
  ----------------- ---------------------- --------------------------------- ------------ ---------------------------------------------

\

Routing with packages
---------------------

While `app.controllers` is a default package for controllers, you might want to organize them into sub-packages. These sub-packages can only be children of `app.controllers` package though. In case a controller is located in a sub-packages, the path mapping would also include sub-package names:

Standard routing

  ------------------------------------------- --------------------------------------------------- ------------ --------
  **path**                                    **controller**                                      **action**   **id**
  /context/package1/books                     app.controllers.package1.BooksController            index        
  /context/package1/books/save                app.controllers.package1.BooksController            save         
  /context/package1/books/save/123            app.controllers.package1.BooksController            save         123
  /context/package1/package2/books            app.controllers.package1.package2.BooksController   index        
  /context/package1/package2/books/save       app.controllers.package1.package2.BooksController   save         
  /context/package1/package2/books/save/123   app.controllers.package1.package2.BooksController   save         123
  ------------------------------------------- --------------------------------------------------- ------------ --------

\
 RESTful routing supports sub-packaging exactly the same as standard.

Mapping paths to controller names
---------------------------------

When matching a path to a controller class, ActiveWeb converts a name of a controller from underscore or hyphenated format to CamelCase:

  ------------------------- -------------------------------------------------
  **path**                  **controller**
  /context/books            app.controllers.package1.BooksController
  /context/student\_books   app.controllers.package1.StudentBooksController
  /context/student-books    app.controllers.package1.StudentBooksController
  ------------------------- -------------------------------------------------

\

Custom routing
--------------

Besides standard and RESTful, ActiveWeb also offers custom routing. Custom routing provides ability to configure custom URIs to be forwarded to specific controllers and actions.

### Custom routing configuration

As with any other types of configuration, ActiveWeb route configuration is done in code, rather that property or XML files:

~~~~ {.prettyprint}
public class RouteConfig extends AbstractRouteConfig {
    public void init(AppContext appContext) {
        route("/myposts").to(PostsController.class);
        route("/{action}/{controller}/{id}");
        route("/{action}/greeting/{name}").to(HelloController.class);
    }
}
~~~~

Custom routing is based on URI segments, which are chunks of URIs submitted in the request separated by slashes. For example the following URI has three segments:

~~~~ {.prettyprint}
/greeting/show/bob
~~~~

The above URI has three segments: "greeting", "show", "bob".

ActiveWeb defines three types of segments:

-   Built in
-   Static
-   User (or dynamic)

### Built-in segments

ActiveWeb defines three built-in segments:

-   `{controller}`
-   `{action}`
-   `{id}`

Using built-in segments, you can reorder where controller, action and Id appear on the URI:

~~~~ {.prettyprint}
/(action)/{controller}/{id}
~~~~

When such a route is specified, this URI:

~~~~ {.prettyprint}
/show/photo/123
~~~~

Will be routed to: `app.controllers.PhotoController#show` with ID ==123.

### Static segments

Static segments are simply plain text without the braces. The are matched one to one with the incoming request. Example:

~~~~ {.prettyprint}
route("/{action}/greeting/{name}").to(HelloController.class);
~~~~

In the snippet above, "greeting" is a static segment.

### User/dynamic segments

User segments are any text in braces in configuration which are then converted to parameters that can be retrieved inside controllers and filters. Here is an example:

~~~~ {.prettyprint}
route("/{action}/greeting/{name}").to(HelloController.class);
~~~~

where "name" is a placeholder whose value will be available in controller:

URL submitted:

~~~~ {.prettyprint}
/show/greeting/alex
~~~~

will be routed to controller `app.controllers.HelloController#show` and value `name` will be available:

~~~~ {.prettyprint}
public class HelloController extends AppController{
  public void show(){
    String name = param("name");
  }
}
~~~~

### Http method - based routing

You can include an Http method used in the request into the routing rule:

~~~~ {.prettyprint}
route("/{action}/greeting/{name}").to(HelloController.class).get();
~~~~

In this example, this route will only match the incoming request if the Http method of the request is GET. There are four corresponding methods: get(), post(), put() and delete(). They can be used in isolation or in combination. For instance, this route:

~~~~ {.prettyprint}
route("/{action}/greeting").to(HelloController.class).get().post();
~~~~

will match these requests:

GET:

~~~~ {.prettyprint}
/show/greeting
~~~~

POST:

~~~~ {.prettyprint}
/save/greeting
~~~~

Of course the action "save" needs to have a @POST annotation for this to work. Annotations are independent of routing rules.

* * * * *

Default Http method used in routing rules is get().

* * * * *

### RouteConfig reloaded

The class `app.config.RouteConfig` is recompiled and reloaded in development environment in case a system property "active\_reload" is set to true. This makes it easy and fun to play with the routes during development.
