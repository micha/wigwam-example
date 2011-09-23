Wigwam
======

Wigwam is a simple PHP web framework. This is an example application using it.

Installation
============

In a shell do the following commands:
  
```bash
    # Cd to your public html directory or docroot
    cd ~/public_html

    # Clone this repository and install the wigwam framework
    git clone git:github.com/micha/wigwam-example.git
    cd wigwam-example
    git submodule init
    git submodule update

    # Create an appropriate htaccess file
    cp rpc/htaccess.example rpc/.htaccess
    vi rpc/.htaccess
```

Quick Start
===========

Point your browser to your newly installed application. You can open the
JavaScript console and watch the xhr traffic, and experiment with commands like
these:

```javascript
    // Run a remote command in synchronous mode

    with (Wigwam.sync) {
      console.log(Wigwam.Test.App.getDoit());
    }

    // Run a remote command in asynchronous mode

    Wigwam.async(
      Wigwam.Test.App.getDoit(),
      function(data) { console.log(data) },
      function(err) { alert(err) }
    );

    // Exception handling

    with (Wigwam.sync) {
      var App = Wigwam.Test.App;

      console.log("1 + 1 = " + App.getOops(1, 1));

      // This will fail with a Wigwam.NotAllowed exception
      try {
        console.log("200 + 1 = " + App.getOops(200, 1));
      } catch (err) {
        if (err instanceof Wigwam.NotAllowed)
          alert(err);
        else
          alert("unexpected exception type: " + err);
      }
    }
```

What's Going On
===============

There are four components in the application:

1. The _rpc/index.php_ file, which loads and configures the framework.
2. The **Wigwam\HTTP** class, which takes care of the HTTP layer, doing all of
   the content negotiation, session management, etc.
3. The API classes that are inserted into the **Wigwam\HTTP** instance. Their
   public static methods are automatically exported to the JavaScript 
   environment.
4. The _index.html_ file, which loads the wigwam JavaScript runtime and makes
   RPC calls.
   
The server-side code that is being executed is the public static methods of the
API class as configured in _rpc/index.php_:

```php
    <?php

    use Wigwam\HTTP;
    use Wigwam\Test\App;

    //===========================================================================//
    // CLASSLOADER INITIALIZATION                                                //
    //===========================================================================//

    require 'classes/Wigwam/ClassLoader.php';

    //===========================================================================//
    // CONFIGURE APPLICATION/HTTP WRAPPER AND HANDLE REQUEST                     //
    //===========================================================================//

    function main() {
      // Config files are processed in order. Settings in files processed later
      // will overwrite settings from files processed earlier.
      $config_files = array('config/config.yaml', 'config/config.local.yaml');

      // Create new HTTP wrapper instance.
      $http = new HTTP($config_files);

      // Load the application API into the HTTP wrapper.
      $http->addApi(new App());

      // Delegate to the HTTP wrapper.
      $http->run();
    }

    main();
```

The API class _rpc/classes/Wigwam/Test/App.php_ used in this example is a minimal
implementation:

```php
    <?php namespace Wigwam\Test;

    use Wigwam\HTTP\Auth\Roles;

    class App {

      /**
       * This implements the App interface (unofficial). Used by the HTTP wrapper
       * for roles based auth. You'd normally subclass the Roles class to implement
       * your own roles based auth system.
       *
       * @return object The roles subclass object.
       */
      public function getRoles() {
        return new Roles();
      }

      /**
       * Public static method: this method will be part of the public API, access-
       * ible via the HTTP interface. You'd call this method with a GET request to
       * /Wigwam/Test/App/getDoit. The response should be an object with the pro-
       * perty 'did' set to the value 'bar', in the format specified by the request
       * accepts header (i.e., Accepts: application/json -> { "did" : "it" }).
       *
       * In PHP:
       *
       *   \Wigwam\Test\App::getDoit();
       *   => array('did' => 'it')
       *
       * In JavaScript:
       *
       *   with (Wigwam.sync) {  // synchronous
       *     Wigwam.Test.App.getDoit();
       *   }
       *   
       *   Wigwam.async(  // asynchronous
       *     Wigwam.Test.App.getDoit(),
       *     function(data) { console.log(data) },
       *     function(err) { alert(err) }
       *   );
       *
       * Here we use the wigwam base roles class, so any role (including deny) 
       * will pass. Normally you'd subclass it to have a functioning roles-based 
       * authorization and authentication system.
       *
       * @role deny
       *
       * @return array What it did.
       */
      public static function getDoit() {
        return array('did' => 'it');
      }

      /**
       * This method will throw an exception that can be caught in JS.
       *
       * @param foo number The foo.
       * @param bar number The bar.
       * @return number The sum of the foo and the bar.
       * @throws \Wigwam\NotAllowed
       */
      public static function getOops($foo, $bar) {
        if ($foo > 100)
          throw new \Wigwam\NotAllowed("too much foo", array('bar' => 'baz'));
        return $foo + $bar;
      }
    }
```
