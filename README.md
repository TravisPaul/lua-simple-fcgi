# lua-simple-fcgi

A simple FastCGI interface for writing stateful web applications in Lua.

## Usage

Create a lua script that returns a table with the following 4 functions.
The only required function is ```accept```.

* [accept](#accept)
* [start](#start)
* [restart](#restart)
* [stop](#stop)

### accept

The `accept` function is called each time a request is made.

You must return a string with the correctly formatted headers and response body.

The `Status` header is particularly important, it is needed by the web server
to return the correct response code and header.

You can access the REQUEST_URI, REQUEST_METHOD and other environment variables
using the `os.getenv` function.

You can never call `print` in the accept function, the FastCGI library owns
standard input, standard output, and standard error at this point. Calling
`print` will cause standard out to close and your application will be unable to
respond to requests.

```lua
    function accept()
        return "Status: 200 Ok\r\n" ..
            "Content-Type: text/html\r\n\r\n" ..
            "<h1>Hello!</h1>"
    end
```

### start

The `start` function is called before the process starts accepting requests.
This is a good place to open database connections, parse config files or 
perform any other initialization tasks needed by your application.

Any non-zero return value will be considered a failure and the process will exit
with the return code.

```lua
    function run()
        -- do initialization stuff here...
        return 0
    end
```

### restart

The `restart` function is called any time a SIGHUP signal is sent to the spawned
FastCGI process. You can use this function to reload configuration files,
reconnect to databases or simply clear the state of the application.

```lua
    function restart()
        -- Do stuff on restart here
    end
```

### stop

The `stop` function is called any time a SIGTERM signal is sent to the spawned 
FastCGI process. It's a good time to flush any cached data or cleanly tear down
connections or open files.

If a non-zero return value is returned the process will not terminate.
<sup>Feature not yet implemented</sup>


```lua
    function stop()
        -- goodbye ...
        return 0
    end
```

## Building

```shell
  git submodule update --init --recursive
  mkdir build
  cd build
  cmake ..
  make
```

## Running your app

See the [examples](examples) and [test](test) directories for webserver
configuration and example lua scripts.

```shell
  $ spawn-fcgi -p 9999 -- lua-fcgi app.lua
  spawn-fcgi: child spawned successfully: PID: 17003  
```

For debugging you can run the application on the command line in CGI mode:
```shell
  $ ./lua-fcgi app.lua
  Lua file doesn't provide an "accept" function.
```

Or you can supply an environment variable named ```ERRLOG``` with the path
to a log file:
```shell
  $ ERRLOG=/path/to/my.log spawn-fcgi -p 9999 -- lua-fcgi app.lua
  spawn-fcgi: child spawned successfully: PID: 4025
  $ cat /path/to/my.log
  Error start function returned non-zero value: 101  
```
