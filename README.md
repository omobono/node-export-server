# Highcharts Node.js Export Server

Convert Highcharts.JS charts to static image files.

## What & Why

This is a node.js application/service that converts [Highcharts.JS](http://highcharts.com) charts to static image files. 
It supports PNG, JPEG, SVG, and PDF output; and the input can be either SVG, or JSON-formatted chart options.

The application can be used either as a CLI (Command Line Interface), as an HTTP server, or as a node.js module.

### Use Cases

The main use case for the export server is situations where headless conversion of charts are required.
Common use cases include automatic report generation, static caching, and for including charts in e.g.
presentations, or other documents.

In addition, the HTTP mode can be used to run your own export server for your users,
rather than relying on the public export.highcharts.com server which is rate limited.

The HTTP server can either be ran stand-alone and integrate with your other applications and services,
or it can be ran in such a way that the export buttons on your charts route to your own server.

To do latter, add:
    
    {
      exporting: {
        url: "<IP to the self-hosted export server>"
      }
    }

to the chart options when creating your charts.

For systems that generate automatic reports, using the export server as a node.js module
is a great fit - especially if your report generator is also written in node.
See [here](https://github.com/highcharts/node-export-server#using-as-a-nodejs-module) for examples.

## Install
    
    npm install highcharts-export-server -g

OR:
    
    git clone https://github.com/highcharts/node-export-server
    npm install
    npm link


## Running
    
    highcharts-export-server <arguments>

## Command Line Arguments

**General options**

  * `--infile`: Specify the input file.
  * `--instr`: Specify the input as a string.
  * `--options`: Alias for `--instr`
  * `--outfile`: Specify the output filename.
  * `--allowFileResources`: Allow injecting resources from the filesystem. Has no effect when running as a server. Defaults to `true`.
  * `--type`: The type of the exported file. Valid options are `jpg png pdf svg`.
  * `--scale`: The scale of the chart.
  * `--width`: Scale the chart to fit the width supplied - overrides `--scale`.
  * `--constr`: The constructor to use. Either `Chart` or `StockChart`.
  * `--callback`: File containing JavaScript to call in the constructor of Highcharts.
  * `--resources`: Stringified JSON.
  * `--batch "input.json=output.png;input2.json=output2.png;.."`: Batch convert  
  * `--logDest <path>`: Set path for log files, and enable file logging
  * `--logFile <filename>`: Set the name of the log file (without the path). Defaults to `highcharts-export-server.log`. Note that `--logDest` also needs to be set to enable logging.
  * `--logLevel <0..4>`: Set the log level. 0 = off, 1 = errors, 2 = warn, 3 = notice, 4 = verbose
  * `--fromFile "options.json"`: Read CLI options from JSON file
  * `--tmpdir`: The path to temporary output files.
  * `--workers`: Number of workers to spawn

**Server related options**

  * `--enableServer <1|0>`: Enable the server (done also when supplying --host)
  * `--host`: The hostname to run the server on.
  * `--port`: The port to listen for incoming requests on.
  * `--sslPath`: The path to the SSL key/certificate. Indirectly enables SSL support.
  * `--rateLimit`: Argument is the max requests allowed in one minute. Disabled by default.

*`-` and `--` can be used interchangeably when using the CLI.*

## Setup: Injecting the Highcharts dependency

In order to use the export server, Highcharts.js needs to be injected
into the export template.

This is largely an automatic process. When running `npm install` you will
be prompted to accept the license terms of Highcharts.js. Answering `yes` will
pull the version of your choosing from the Highcharts CDN and put them where they need to be.

However, if you need to do this manually you can run `node build.js`.

### Using In Automated Deployments

If you're deploying an application/service that depend on the export server 
as a node module, you can set the environment variable `ACCEPT_HIGHCHARTS_LICENSE` to `YES`
on your server, and it will automatically agree to the licensing terms when running
`npm install`. You can also use `HIGHCHARTS_VERSION` and `HIGHCHARTS_USE_STYLED`
to bake with a specific Highcharts version, and to enable styled mode (requires
a Highcharts 5 license).

## Note About Resources and the CLI

If `--resources` is not set, and a file `resources.json` exist in the folder
from which the cli tool was ran, it will use the `resources.json` file.

## HTTP Server

The server accepts the following arguments:

  * `infile`: A string containing JSON or SVG for the chart 
  * `options`: Alias for `infile`
  * `svg`: A string containing SVG to render
  * `type`: The format: `png`, `jpeg`, `pdf`, `svg`. Mimetypes can also be used.
  * `scale`: The scale factor
  * `width`: The chart width (overrides scale)
  * `callback`: Javascript to execute in the highcharts constructor.
  * `resources`: Additional resources.
  * `constr`: The constructor to use. Either `Chart` or `Stock`.
  * `b64`: Bool, set to true to get base64 back instead of binary.
  * `async`: Get a download link instead of the file data
  * `noDownload`: Bool, set to true to not send attachment headers on the response.
  * `asyncRendering`: Wait for the included scripts to call `highexp.done()` before rendering the chart.
  * `globalOptions`: A JSON object with options to be passed to `Highcharts.setOptions`.
  * `dataOptions`: Passed to `Highcharts.data(..)`
  * `customCode`: When `dataOptions` is supplied, this is a function to be called with the after applying the data options. Its only argument is the complete options object which will be passed to the Highcharts constructor on return.

Note that the `b64` option overrides the `async` option.

It responds to `application/json`, `multipart/form-data`, and URL encoded requests.

CORS is enabled for the server.

It's recommended to run the server using [forever](https://github.com/foreverjs/forever) unless running in a managed environment such as AWS Elastic Beanstalk.

### Running in Forever

The easiest way to run in forever is to clone the node export server repo, and run `forever start ./bin/cli.js --enableServer 1` in the project folder.

Remember to install forever first: `sudo npm install -g forever`.

Please see the forever documentation for additional options (such as log destination).

### SSL

To enable ssl support, drop your `server.key` and `server.crt` in the ssl folder,
or add `--sslPath <path to key/crt>` when running the server.

## Server Test

Run the below in a terminal after running `highcharts-export-server --enableServer 1`.
    
    # Generate a chart and save it to mychart.png    
    curl -H "Content-Type: application/json" -X POST -d '{"infile":{"title": {"text": "Steep Chart"}, "xAxis": {"categories": ["Jan", "Feb", "Mar"]}, "series": [{"data": [29.9, 71.5, 106.4]}]}}' 127.0.0.1:7801 -o mychart.png

## Using as a Node.js Module

The export server can also be used as a node module to simplify integrations:
    
    //Include the exporter module
    const exporter = require('highcharts-export-server');

    //Export settings 
    var exportSettings = {
        type: 'png',
        options: {
            title: {
                text: 'My Chart'
            },
            xAxis: {
                categories: ["Jan", "Feb", "Mar", "Apr", "Mar", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"]
            },
            series: [
                {
                    type: 'line',
                    data: [1, 3, 2, 4]
                },
                {
                    type: 'line',
                    data: [5, 3, 4, 2]
                }
            ]
        }
    };

    //Set up a pool of PhantomJS workers
    exporter.initPool();

    //Perform an export
    /*
        Export settings corresponds to the available CLI arguments described
        above.
    */
    exporter.export(exportSettings, function (err, res) {
        //The export result is now in res.
        //If the output is not PDF or SVG, it will be base64 encoded (res.data).
        //If the output is a PDF or SVG, it will contain a filename (res.filename).

        //Kill the pool when we're done with it, and exit the application
        exporter.killPool();
        process.exit(1);
    });

### Node.js API Reference

**highcharts-export-server module**

**Functions**

  * `log(level, ...)`: log something. Level is a number from 1-4. Args are joined by whitespace to form the message.
  * `logLevel(level)`: set the current log level: `0`: disabled, `1`: errors, `2`: warnings, `3`: notices, `4`: verbose
  * `enableFileLogging(path, name)`: enable logging to file. `path` is the path to log to, `name` is the filename to log to
  * `export(exportOptions, fn)`: do an export. `exportOptions` uses the same attribute names as the CLI switch names. `fn` is called when the export is completed, with an object as the second argument containing the the filename attribute.
  * `startServer(port, sslPort, sslPath)`: start an http server on the given port. `sslPath` is the path to the server key/certificate (must be named server.key/server.crt)
  * `server` - the server instance 
    * `enableRateLimiting(options)` - enable rate limiting on the POST path
      * `max` - the maximum amount of requests before rate limiting kicks in
      * `window` - the time window in minutes for rate limiting. Example: setting `window` to `1` and `max` to `30` will allow a maximum of 30 requests within one minute.
      * `delay` - the amount to delay each successive request before hitting the max
      * `trustProxy` - set this to true if behind a load balancer
      * `skipKey`/`skipToken` - key/token pair that allows bypassing the rate limiter. On requests, these should be sent as such: `?key=<key>&access_token=<token>`.
    * `app()` - returns the express app
    * `express()` - return the express module instance
    * `useFilter(when, fn)` - attach a filter to the POST route. Returning false in the callback will terminate the request.
      * `when` - either `beforeRequest` or `afterRequest`
      * `fn` - the function to call 
        * `req` - the request object
        * `res` - the result object
        * `data` - the request data
        * `id` - the request ID
        * `uniqueid` - the unique id for the request (used for temporary file names)        
  * `initPool(config)`: init the phantom pool - must be done prior to exporting. `config` is an object as such:
    * `maxWorkers` (default 25) - max count of worker processes
    * `initialWorkers` (default 5) - initial worker process count
    * `workLimit` (default 50) - how many task can be performed by a worker process before it's automatically restarted
  * `killPool()`: kill the phantom processes

## Using Ajax in Injected Resources

If you need to perform Ajax requests inside one of the resource scripts,
set `asyncRendering` to true, and call `highexp.done()` in the Ajax return to process the chart.

Example:
    
    {
      asyncRendering: true,
      resources: {
        files: 'myAjaxScript.js'    
      }
    }

myAjaxScript.js:
    
    jQuery.ajax({
      url: 'example.com',
      success: function (data) {
        ...
        highexp.done();
      },
      error: function () {
        highexp.done();
      }
    });

If the Ajax call doesn't call `highexp.done()` within 60 seconds, the 
rendering will time out.

## Performance Notice

In cases of batch exports, it's faster to use the HTTP server than the CLI.
This is due to the overhead of starting PhantomJS for each job when using the CLI. 

As a concrete example, running the CLI with [testcharts/basic.json](testcharts/basic.json) 
as the input and converting to PNG averages about 449ms. 
Posting the same configuration to the HTTP server averages less than 100ms.

So it's better to write a bash script that starts the server and then
performs a set of POSTS to it through e.g. curl if not wanting to host the
export server as a service.

Alternatively, you can use the `--batch` switch if the output format is the same
for each of the input files to process:
    
    highcharts-export-server --batch "infile1.json=outfile1.png;infile2.json=outfile2.png;.."

Other switches can be combined with this switch.

## License

[MIT](LICENSE).
