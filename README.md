

## How Does It Work ##
FunctionShield uses a proprietary behavioral-based runtime protection engine, which enforces the behavior that you define. FunctionShield lives in and around the serverless language runtime. All you have to do is import the library into your code. No function wrapping required. FunctionShield doesn't wrap your code, or perform any kind of monkey-patching.

Below is a sample code snippet, demonstrating how to use FunctionShield:

### Node/JavaScript (AWS) ###

```javascript
var AWS = require('aws-sdk');
const FunctionShield = require('@puresec/function-shield');
FunctionShield.configure({
    policy: {
        // 'block' mode => active blocking
        // 'alert' mode => log only
        // 'allow' mode => allowed, implicitly occurs if key does not exist
        outbound_connectivity: "block",
        read_write_tmp: "block", 
        create_child_process: "block",
        read_handler: "block" },
    token: process.env.FUNCTION_SHIELD_TOKEN });

exports.hello = async (event) => {
    // ... // your code
};
```

### Python (AWS) ###

```python
import boto3
import function_shield

function_shield.configure({
    "policy": {
        # 'block' mode => active blocking
        # 'alert' mode => log only
        # 'allow' mode => allowed, implicitly occurs if key does not exist
        "outbound_connectivity": "block",
        "read_write_tmp": "block",
        "create_child_process": "block",
        "read_handler": "block"
    },
    "token": os.environ['FUNCTION_SHIELD_TOKEN']
})

def handler(event, context):
    # Your Code Here #
```

### Java (AWS) ###

```java
package com.serverless;

import java.util.Map;
import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;
import org.json.JSONObject;
import io.puresec.FunctionShield;

public class Handler implements RequestHandler <Map <String, Object> , String >
{
    static
    {
        /* 'block' mode => active blocking */
        /* 'alert' mode => log only */
        /* 'allow' mode => allowed, implicitly occurs if key does not exist */
        FunctionShield.configure(new JSONObject()
            .put("policy", new JSONObject()
                .put("read_write_tmp", "alert")
                .put("create_child_process", "alert")
                .put("outbound_connectivity", "block")
                .put("read_handler", "block"))
            .put("token", System.getenv("FUNCTION_SHIELD_TOKEN")));
    }

    @Override
    public String handleRequest(Map <String, Object> input, Context context)
    {
        /* TODO: Your function code here */
        return("Done!");
    }

}
```

### Node/JavaScript (GCF) ###

```javascript
'use strict';
const FunctionShield = require('@puresec/function-shield');
FunctionShield.configure(
    {
        policy: {
            // 'block' mode => active blocking
            // 'alert' mode => log only
            // 'allow' mode => allowed, implicitly occurs if key does not exist
            read_write_tmp: "alert",
            create_child_process: "alert",
            outbound_connectivity: "alert",
            read_handler: "alert"
        },
        disable_analytics: false,
        token: process.env.FUNCTION_SHIELD_TOKEN
    });

function handler(request, response) {
    // ... your code ... //
}
```

### Python (GCF) ###

```python
import function_shield

function_shield.configure({
    "policy": {
        # 'block' mode => active blocking
        # 'alert' mode => log only
        # 'allow' mode => allowed, implicitly occurs if key does not exist
        "read_write_tmp": "block",
        "create_child_process": "block",
        "outbound_connectivity": "block",
        "read_handler": "block"
    },
    "disable_analytics": False,
    "token": os.environ['FUNCTION_SHIELD_TOKEN']
})


def python37_handler(request):
    # ... your code here ... #
```

## Installation Instructions ##
1. [Sign up](https://www.puresec.io/function-shield-token-form) to receive your free FunctionShield token
2. Install the library:
    * Node.js: Run 'npm i @puresec/function-shield'
    * Python: Run 'pip3 install function-shield'
    * Java: See instructions for the Java build tools in this LINK
3. Import the library into your code (see code snippets)
4. Configure your desired security policy
5. Embed the token you received by email into the code
6. Rinse, repeat for all of your functions

At the moment, FunctionShield is offered with the following runtimes support:
* AWS Lambda security: Node.js, Python and Java support
* Google Cloud Functions security: Node.js and Python


FunctionShield is also provided as a middleware for the (MIDDY.JS)[https://github.com/middyjs/middy/blob/master/docs/middlewares.md#functionshield] engine. 

## Notes ##

* When using the 'outbound-connectivity' restriction, traffic to the cloud enviroment (AWS cloud or Google cloud) resources will still be enabled, in order to avoid disruption to normal function operation
* Logging from FunctionShield is done asynchronously to CloudWatch or Google cloud logs, and does not add latency
* Overall latency of FunctionShield is extremely low, and is expected to be under 1 millisecond.
* Load time for cold-starts was measured around 20ms
* FunctionShield is a completely self-contained technology, no sensitive data is sent to PureSec. Periodically, during cold starts FunctionShield will send information on its configuration and release version. This behavior can be disabled by using the configuration key "disable_analytics: true"
* Alert mode - ('alert') only logs security events. Block mode - ('block') , will halt function execution when a violation of the policy occurs
* You can call the configure() method as many times as needed inside your function. This means that you can temporarily disable or modify FunctionShield's behavior, depending on your needs. For example, in order to allow outbound connectivity only in a certain portion of your code

## Logging & Security Visibility ##

FunctionShield logs are sent directly to your function's AWS CloudWatch log group or Google cloud logs. Here are a few sample logs, demonstrating the log format you should expect:

```javascript
// Log example #1:
{
    "function_shield": true,
    "policy": "outbound_connectivity",
    "details": {
        "host": "google.com"
    },
    "mode": "alert"
}

// Log example #2:
{
    "function_shield": true,
    "policy": "read_write_tmp",
    "details": {
        "path": "/tmp/node-alert"
    },
    "mode": "alert"
}

// Log example #3:
{
    "function_shield": true,
    "policy": "create_child_process",
    "details": {
        "path": "/bin/sh"
    },
    "mode": "block"
}

// Log example #4:
{
   "function_shield": true,
   "policy": "read_handler",
   "details": {
       "path": "/var/task/handler.js"
   },
   "mode": "alert"
}
```
