Wigwam
======

Wigwam is a simple PHP web framework. This is an example application using it.

Quick Start
===========

In a shell do the following commands:
  
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

Then point your browser to your newly installed application. You can open the
JavaScript console and watch the xhr traffic, and experiment with commands like
these:

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
