# JSON Validation Service

The JSON Validation Service (JVS) is a validator that allows a user to check JSON objects for compliance with the JSON grammar according to [RFC 7159§](https://tools.ietf.org/html/rfc7159#section-9).

## Features

* [Plain API](#usage)
* [Easy to deploy](#deployment) using provided [`Dockerfile`](Dockerfile)
* Supports configurable logging based on *Java Logging API* via [`logging.properties`](logging.properties) file

## Usage

### A Simple Way

```java
JSONValidationService service;

try {
    service = new JSONValidationService(); // use default parameters
    service.start();
} catch (JVSException e) {
    LOG.log(Level.SEVERE, "Service exception", e);
}
```

### Customize Options

You have several ways to build [`JVSOptions`](src/service/JVSOptions.java) and pass it to the [`JSONValidationService`](src/service/JSONValidationService.java) class constructor.

```java
/* use default options */
JVSOptions options1 = new JVSOptions();
/* populate parameters from the configuration file */
JVSOptions options2 = new JVSOptions("config.file");
/* manually build your JVSOptions instance */
JVSOptions options3 = new JVSOptionsBuilder("localhost", 8081)
                      .setPath("/app")
                      .build();
/* or tune parameters from the file at runtime */
JVSOptions options4 = new JVSOptions("config")
                      .newBuilder()
                      .setPort(8080)
                      .setDelay(32)
                      .build()

/* and use like this */
new JSONValidationService(options4);
```

## Configuration File

The configuration file is in JSON format with the following fields:

 * `host` - a hostname or an IP address for the service to bind
 * `port` - a port number that is used by the service
 * `backlog` - a maximum number of incoming TCP connections
 * `path` - a location of the service on a given host
 * `delay` - the maximum time in seconds to wait until exchanges are finished

See [jvs.properties](jvs.properties) file for an example.

## Handling the Responses

1. If a received JSON is valid it is formatted to human-readable form and sent back.

2. If data in a request do not conform to the JSON grammar, the following response is sent:

    ```json
     {
         "errorCode"  : 12,
         "errorMessage" : "plain language description of the problem",
         "errorPlace" : "the point where error has occurred",
         "resource"   : "filename taken from a URI",
         "request-id" : "the request id for easier tracking of errors"
     }
     ```

Below is a list of known errors:

| Code | Meaning                         |
|------|---------------------------------|
| 1    | Unterminated array              |
| 2    | Unterminated object             |
| 3    | Expected name                   |
| 4    | Expected ':'                    |
| 5    | Unexpected value                |
| 6    | Expected value                  |
| 7    | Unterminated string             |
| 8    | Unterminated comment            |
| 9    | Malformed JSON                  |
| 10   | Unterminated escape sequence    |
| 11   | Invalid escape sequence         |
| 12   | JSON forbids NaN and infinities |

Normally, JVS sends `HTTPS OK` responses with `Content-Type: application/json` in either case.

## Deployment

Run and build the Docker image:
```shell
$ cd "the path where the Dockerfile is placed"
$ docker build -t jvs .
$ docker run --rm -d -p 80:80 jvs
```

Or you can fork this project, change it as you want to, and commit the changes. Then you can simply run:

```shell
$ docker run --rm -d -p 80:80 jvs https://example.com/your_repo.git
```

To test that JVS is working properly, make a POST request as shown below:

```shell
$ cat file.json # example of a JSON file
{
    "key" : "value"
}
$ curl -s --data-binary @file.json http://example.com[:port]/file.json
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

## Acknowledgments

* The project is based on [Google Gson](https://github.com/google/gson) library
