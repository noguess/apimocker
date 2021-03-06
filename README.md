# apimocker
This is a node.js module to run a simple http server, which can serve up mock service responses.
Responses can be JSON or XML to simulate REST or SOAP services.
Access-Control HTTP Headers are set by default to allow CORS requests.
Mock services are configured in the config.json file, or on the fly, to allow for easy functional testing.
Using apimocker, you can develop your web or mobile app with no dependency on back end services.
(There are lots of these projects out there, but I wrote this one to support all kinds of responses,
to allow on-the-fly configuration, and to run in node.)

## Installation
		sudo npm install -g apimocker
That will install globally, and allow for easier usage.
(On Windows, you don't need "sudo".)

## Usage
        apimocker [-c, --config \<path\>] [-q, --quiet] [-p \<port\>]

Out of the box, you can just run "apimocker" with no arguments.
(Except on windows, you'll need to edit config.json first.  See below.)

Then you can visit "http://localhost:7878/first" in your browser to see it work.
The quiet and port options can also be set in the config.json file,
and values from config.json will override values from the command line.
After you get up and running, you should put your config.json and mock responses in a better location.
It's not a good idea to keep them under the "node_modules" directory.
Make sure another process is not already using the port you want.
If you want port 80, you may need to use "sudo" on Mac OSX.

### Windows note
After installing from npm, you'll need to edit this file:
        /Users/xxxxx/AppData/Roaming/npm/node_modules/apimocker/config.json
Change the "mockDirectory" to point to this location.
(Or another location where you put the mock responses.)
        mockDirectory: /Users/xxxxx/AppData/Roaming/npm/node_modules/apimocker/samplemocks

### Help
        apimocker -h

## Configuration
On startup, config values are loaded from the config.json file.
During runtime, mock services can be configured on the fly.
See the sample config.json file in this package.

* config.json file format has changed with the 0.1.6 release.  See below for the new format.  (Old config.json file format is deprecated, but still functioning.)
* Content-type for a service response can be set for each service.  If not set, content-type defaults to applicatoin/xml for .xml files, and application/json for .json files.
* Latency (ms) can be set to simulate slow service responses.  Latency can be set for a single service, or globally for all services.
* mockDirectory value should be an absolute path.
* Allowed domains can be set to restrict CORS requests to certain domains.

```js
{
  "note": "This is a sample config file. You should change the mockDirectory to a more reasonable path.",
  "mockDirectory": "/usr/local/lib/node_modules/apimocker/samplemocks/",
  "quiet": false,
  "port": "7878",
  "latency": 50,
  "allowedDomains": ["abc.com"],
  "webServices": {
    "first": {
      "mockFile": "king.json",
      "latency": 20,
      "verbs": ["get"]
    },
    "second": {
      "mockFile": "king.json",
      "contentType": "foobar",
      "verbs": ["post"]
    },
    "nested/ace": {
      "mockFile": "ace.json",
      "verbs": ["post", "get"]
    },
    "var/:id": {
      "mockFile": "xml/queen.xml",
      "verbs": ["all"]
    }
  }
}
```
The most interesting part of the configuration file is the webServices section.
This section contains a JSON object describing each service.  The key for each service object is the service URL (endpoint.)  Inside each service object, the "mockFile" and "verbs" are required.  "latency" and "contentType" are optional.
For instance, a GET request sent to "http://server:port/first" will return the king.json file from the samplemocks directory, with a 20 ms delay.

## Runtime configuration
After starting apimocker, mocks can be configured using a simple http api.
This http api can be called easily from your functional tests, to test your code's handling of different responses.

### /admin/setMock
This allows you to set a different response for a single service at any time by sending an http request.
Request can be a post containing a JSON object in the body:
```js
{
	"verb":"get",
	"serviceUrl":"third",
	"mockFile":"queen.xml",
    "latency": 100,
    "contentType": "anythingyouwant"
}
```

or a get with query string parameters:
localhost:7878/admin/setMock?verb=get&serviceUrl=second&mockFile=ace.json

### /admin/reload
If the config.json file is edited, you can send an http request to /admin/reload to pick up the changes.

## Versions
### 0.1.6
New config file format was introduced, allowing for custom content-types and more fine grained control over services.

## Contributors
Run "grunt watch" in the root "apimocker" directory to start the grunt watch task.  This will run JSHint and mocha tests.

## Acknowledgements
Big thanks to magalhas for his httpd-mock project.  This gave me a great starting point.
Also thanks to clafonta and the Mockey project for inspiration.
