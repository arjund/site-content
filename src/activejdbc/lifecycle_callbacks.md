Lifecycle callbacks | <a href="/activejdbc">ActiveJDBC</a>,Lifecycle callbacks

# Lifecycle callbacks

<div id="toc"></div>

# Lifecycle callbacks

Like ActiveRecord, ActiveJDBC has lifecycle callbacks. These are methods that can be implemented on a Model subclass to get notified of a special life cycle event performed on a model. These callbacks are captured in an interface that is implemented by a `Model` class:

## Callback interface

~~~~ {.java}
public interface  CallbackListener {
    void beforeSave(Model m);
    void afterSave(Model m);

    void beforeCreate(Model m);
    void afterCreate(Model m);

    void beforeDelete(Model m);
    void afterDelete(Model m);

    void beforeValidation(Model m);
    void afterValidation(Model m);
}
~~~~

See: [CallbackListener](http://javalite.github.io/activejdbc/org/javalite/activejdbc/CallbackListener.html)

There are total of eight calls that a subclass can override to get notified of a specific event.

## Registration of external listeners

You can implement the [CallbackListener](http://javalite.github.io/activejdbc/org/javalite/activejdbc/CallbackListener.html)
interface external to any model and then register it:

~~~~ {.java}
Registry.instance().addListener(Role.class, myListener);
~~~~

or like this:

~~~~ {.java}
Role.addListener(myListener1, myListener2,...);
~~~~

this assuming that Role is a model. You can implement either the [CallbackListener](http://javalite.github.io/activejdbc/org/javalite/activejdbc/CallbackListener.html) interface or extend
[CallbackAdapter](http://javalite.github.io/activejdbc/org/javalite/activejdbc/CallbackAdapter.html) (where all methods
are implemented with blank bodies) and only override the ones you need.

## Override Model callback methods

The Model class already extends a class [CallbackAdapter](http://javalite.github.io/activejdbc/org/javalite/activejdbc/CallbackAdapter.html),
which provides empty implementations of these eight methods. All a developer needs to do is to override one or more
methods to perform a task at a certain time.

## Usage

Let's say we have a model `User`:

~~~~ {.java}
public class User extends Model{}
~~~~

and a user has a password that needs to be stored in a DB in encrypted form. Using callbacks is useful in this case,
since all you have to do is to override a `beforeSave()` method and provide some encryption routine to make the password secure:

~~~~ {.java}
public class User extends Model{
   public void beforeSave(){
      set("password" encryptPassword());
   }   
   private String encryptPassword(){
      //do what it takes
   }
}
~~~~

The framework will call `beforeSave()` within a context of `save()` or `saveIt()` when appropriate, and your
code will encrypt the password for storage.
