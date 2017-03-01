Kicking the tires of [IronFunctions](http://open.iron.io/), following
their [Quickstart guide](https://github.com/iron-io/functions#quickstart).

I decided to make a Python function instead of Go.

I put the command to run their docker image in the [main.sh](main.sh) script.

```
./main.sh
```

Get the CLI:

```
curl -LSs http://get.iron.io/fn | sh
```

I'm on a Mac with [Homebrew](https://brew.sh/), so I change the owner
to me instead of root:

```
sudo chown $USER:admin /usr/local/bin/fn
```

I wrote a Python function instead of a Go function, and called it
`hello.py`. I hit a bump here:

```
$ fn init $USER/hello
no supported files found to guess runtime, please set runtime explicitly with --runtime flag
```

Okay, waht are the available runtimes?


```
$ fn init --help
NAME:
   fn init - create a local func.yaml file

USAGE:
   fn init [command options] <DOCKERHUB_USERNAME/FUNCTION_NAME>

DESCRIPTION:
   Creates a func.yaml file in the current directory.  

OPTIONS:
   --force, -f              overwrite existing func.yaml
   --runtime value          choose an existing runtime - .cs, .fs, .go, .js, .rb, .py, .rs
   --entrypoint value       entrypoint is the command to run to start this function - equivalent to Dockerfile ENTRYPOINT.
   --format value           hot function IO format - json or http
   --max-concurrency value  maximum concurrency for hot function (default: 1)
```

Cool, I'll use `.py`:

```
$ fn init $USER/hello --runtime .py
init does not support the .py runtime, you'll have to create your own Dockerfile for this function
```

Denied!

Wait though, how does the Go example know to use the Go runtime? Maybe
the `func` prefix of the filename is important. Moved my `hello.py` to
[func.py](func.py), and it worked like a charm:

```
$ fn init $USER/hello
assuming python runtime
func.yaml created.
$ fn build
Building image mboldt/hello:0.0.1
Sending build context to Docker daemon 145.4 kB
Step 1/4 : FROM iron/python:2
 ---> 4c72ca003fce
Step 2/4 : WORKDIR /function
 ---> Using cache
 ---> dc49814d1377
Step 3/4 : ADD . /function/
 ---> 1810ac67caf7
Removing intermediate container 151e0532f517
Step 4/4 : ENTRYPOINT python2 func.py
 ---> Running in 11ba4beb8735
 ---> 2e655565b446
Removing intermediate container 11ba4beb8735
Successfully built 2e655565b446
Function mboldt/hello:0.0.1 built successfully.
$ fn run
Hello, world!
$ fn push
The push refers to a repository [docker.io/mboldt/hello]
4e4018d897cf: Pushed 
b959df2cef7e: Pushed 
e67f7ef625c5: Mounted from iron/python 
321db514ef85: Mounted from iron/python 
6102f0d2ad33: Mounted from iron/python 
0.0.1: digest: sha256:911150032b4e6bbd5dc2565f3b05f73ba499f9976b04da6c92b22e400c372f01 size: 1365
Function mboldt/hello:0.0.1 pushed successfully to Docker Hub.
$ fn routes create helloapp /hello mboldt/hello
/hello created with mboldt/hello:0.0.1
$ curl http://localhost:8080/r/helloapp/hello
Hello, world!
```
