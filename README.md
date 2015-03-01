# github-thrift-client-server

This project currently consists of 3 clojure projects:
 - api     
 - feeder
 - server

And 1 java spark project built with maven:
 - web 

The api project builds the library jar which contains classes generated by thrift. This is shared by both the client and the server. The feeder application pulls information about "push" events from github and then sends them via thrift to a server. The test server implementation in server/testserve.clj just stores the received push events in memory.  

The feed traversal has been implemented as a lazy sequence of single events which transparently loads additional pages of events as required. 

## Installation

### API

First, build the api project:

1.) Download [thrift](http://thrift.apache.org/) tarball:
   * `./configure` ensuring that java support is enabled (depends on java 1.7+ and ant).
   * `make && make install`

2.) Ensure the `thrift` executable is available on the path.

3.) run `lein jar` to generate the class files from thrift definitions.

4.) run `lein install` to install the library in the local maven repo.

This will make the library available to other projects.

### Server

To build the server jar:

```
cd server
lein uberjar
```

To run the server: `java -jar server-0.1.0-SNAPSHOT-standalone.jar`

This will launch a server that will respond to thrift requests by querying mongodb.

|ENV Variables:    | Defaults  |
|------------------|-----------| 
| APP_MONGO_HOST   | "mongodb" |
| APP_MONGO_PORT   | 27017     |
| APP_SERVER_PORT  | 8080      |
| APP_BIND_HOST    | "0.0.0.0" |

These can be overriden:

`APP_BIND_HOST=127.0.0.1 APP_SERVER_PORT=9000 java -jar server-0.1.0-SNAPSHOT-standalone.jar`

The .deb can be built by running `release.sh` script in the release directory.

### Feeder

To build the feeder jar 

```
cd feeder
lein uberjar
```

Get an api token from [github](https://github.com/blog/1509-personal-api-tokens)

To run the feeder: `GITHUB_TOKEN="<TOKEN>" java -jar feeder-0.1.0-SNAPSHOT-standalone.jar`

This will attempt to poll for latest events and insert them into mongo.

|ENV Variables:    | Defaults           |
|------------------|--------------------| 
| APP_MONGO_HOST   | "mongodb"          |
| APP_MONGO_PORT   | 27017              |
| APP_MAX_EVENTS   | 10000              |
| APP_GITHUB_TOKEN | &lt;no default&gt; |

These can be overriden:

`APP_MONGO_HOST=127.0.0.1 APP_MONGO_PORT=9000 APP_GITHUB_TOKEN=blah-blah-blah java -jar feeder-0.1.0-SNAPSHOT-standalone.jar`

The .deb can be built by running `release.sh` script in the release directory.

### Web

To build the web jar

```
cd web
mvn clean install
```
To run the "web" service: `java -jar web-0.0.1-SNAPSHOT.jar`

This will server the web frontend.

|ENV Variables:    | Defaults           |
|------------------|--------------------|
| APP_SERVER_PORT   | "8080"            |

These can be overriden:

`APP_SERVER_PORT=8222 java -jar web-0.0.1-SNAPSHOT.jar`

##Run with Docker

This obviously requires docker to be installed. There a 3 containers, 1 for each of the following service:
 * mongodb
 * feeder
 * server 

There is a script added for convenience in `docker/docker.sh`.

As a one of step the images need to be built for each container:
```
  cd docker/
  sudo ./docker.sh build
```

After the images have been built, containers can be built and started with the following command:
```  
GITHUB_TOKEN="${TOKEN}" ./docker.sh start
```

The containers can safely be stopped and disposed of with the following command:
```
./docker.sh stop
```

It takes a bit of time for mongodb to start up once its container has run, meaning the feeder may fail the first time it is scheduled to run.

##TODO
* Add web dockerfile and alter script and README.md
* Update readme
* Use logback or some clj logging framework 
* Proper error handling
* Prevent duplicate events from being added to mongo
* Vagrant?
* Add commits to frontend/pagination
* Add serverside pagination

## License

Copyright © 2015 AL

Distributed under the Eclipse Public License either version 1.0 or (at
your option) any later version.
