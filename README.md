Wigwam
======

Wigwam is a simple PHP web framework. This is an example application using it.

Quick Start
===========

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

Then point your browser to your newly installed application. You can open the
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

    /**
     * Main function to encapsulate any local variables.
     */
    function main() {
      // Config files are processed in order. Settings in files processed later
      // will overwrite settings from files processed earlier.
      $config_files = array('config/config.yaml', 'config/config.local.yaml');

      // Create new HTTP wrapper instance.
      $http = new HTTP($config_files);

      // Load the application API into the HTTP wrapper.
      $http->addApi(new App());

      // Add a specific route handler separate from the API.
      $http->get('/foo', function() use ($http) {
        return $http->render(array('foo', 'bar', 'baz'));
      });

      // Delegate to the HTTP wrapper.
      $http->run();
    }

    // Doit.
    main();
```
