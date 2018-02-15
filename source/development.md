# Development

## How to contribute

Coming soon...

## Development environment

PM2 is keeping the backend and frontend running with
    
    NODE_ENV: 'production'

If environment value NODE_ENV is set to **dev** instead of **production**,

    "NODE_ENV": "dev"
     
application changes behaviour on the following aspects:
*   No actual salt calls are made while running the setup or testing components buttons.
* *  This project is still in active development and frontendy people do not want
to wait a millenia while a component actually installs or uninstall itself.
*   Reset button will appear in the menu, which quickly destroys the database 
back to default and kickstarts the web engine.
* * Easier to test registration mechanics.
* Application will switch from PM2 to hot reloading 
* * client side has this feature builtin
* * server side uses nodemon for this, create a config from sample:

    
    /srv/s4a-[central | detector]/server/nodemon.sample.json


---

Another Important environment variable is **DEBUG_LEVEL** 

    "DEBUG_LEVEL":"5"  

which sets the debug level accordingly for logging output, dev mode or not.
Levels are from level 1 ( error ) to 6 ( strace ).

    self.method_map = [false, "error", "warn", "debug", "log", "info", "trace"];
    
    ( /srv/s4a-[central | detector]/server/common/models/helper.js )

Most of the steps in the application workflow are logged from start to stop.

Increase the log level to find out where issue gets stuck.

* PM2 logs are kept in 

    /home/s4a/.pm2/logs
    