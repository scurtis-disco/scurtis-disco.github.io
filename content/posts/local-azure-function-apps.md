+++
title = 'Azure Function Apps Authentication - Local Edition'
date = 2025-10-24
draft = false
+++

## The Scenario

In my position that I started in September of 2024, I have been working in the Azure space. It has been a great learning
experience given that I had been pretty much AWS-only before. And one area that I have explored quite a bit is Azure Function Apps.
They are analogous to AWS Lambdas with their own Microsoft-related quirks.

One of the nice things about them is that Microsoft has released official Docker images for building and running them locally, so you can test out your code changes with a minimum of effort, and they come in a number of languages. Mostly I write them in Python because it is
quick and easy to get rolling with and Python has so many mature modules for doing the boilerplate stuff. 

## The Issue

One thing I ran across was trying to authenticate against my locally running function app in Docker - this [SO question](https://stackoverflow.com/questions/77096900/where-do-i-find-the-function-key-for-a-locally-deployed-azure-function) sums it up nicely.[^1] 
Function Apps default to function-level authentication if not specified when creating the app + you can set it per function.
When deployed to Azure, you can go to the portal in Azure and pull the keys for the deployed app and then pass the appropriate key in an HTTP
header called `x-functions-key` when invoking the function app. If you really want to be fine-grained, each function has its own key, too,
in addition to the `master` and `default` keys defined at the app level. And you can annotate a route with admin-only authentication which would require the `master` key.


```python
# Authorization required with this decorator = the master key is the only one that provides this level of access
@app.route(function_name="GetStuff", auth.HttpAuthenticationLevel=ADMIN)
def get_thing_admin()
    ...

# Authorization required with this decorator = function level key; the default key or a function-specific key work here 
@app.route(function_name="GetStuff", auth.HttpAuthenticationLevel=FUNCTION)
def get_thing()
    ...

# Authorization required with this decorator = NONE, this is really not that useful
# unless you're hosting a public endpoint for something non-critical or you're using it
# for a healthcheck and/or it's required to be open to all
@app.route(function_name="GetStuff", auth.HttpAuthenticationLevel=ANONYMOUS)
def get_thing()
    ...
```

But what about if you are running the app in a local Docker container?

## Ideas 
The shortcut suggestions are things like:
* [Set it to anonymous auth](https://stackoverflow.com/questions/77096900/where-do-i-find-the-function-key-for-a-locally-deployed-azure-function/79051047#79051047) which you have to remember to reset before deploying
* [Answers that only apply](https://stackoverflow.com/a/77104022/4616451) to the deployed function in Azure
* [A technology-specific answer](https://stackoverflow.com/a/78795603/4616451) - but what if you're not using Testcontainers?

Unlike some SO answers, none of them are totally useless in that they could pertain to a scenario that applies to the reader. If you're throwing together
a quick POC that you're tearing down afterwards and there's no sensitive data involved, you can use ANONYMOUS all day long. Or maybe your org uses Testcontainers
extensively. But none of those satisfied what I needed. And I have a personal loathing for situations where local is different than a "real" environment.
To me, local should only differ in configuration and capacity and should give you a reasonable facsimile of the experience when the code is deployed.

So before I discovered this answer, I plugged away for hours at this problem, trying a number of things with my Dockerfile that I found online or that AI suggested, all to no avail.
I would put them here as warnings but I blocked them out of my memory - it was quite a frustrating day. And unfortunately, the Microsoft documentation on
function apps is no help at all on this topic.

## The Answer
 The basic solution requires 3 parts:
* Setting an environment variable `AzureWebJobsSecretStorageType=files` (I did it directly in the Dockerfile but you can pass it as an env variable at docker run, too)
* Setting an environment variable `AzureFunctionsJobHost__SecretsPath=/azure-functions-host/Secrets`
* Mounting a volume at that path containing a file called `host.json`

The contents of `host.json` should look something like this:

```json
{
  "masterKey": {
    "name": "_master",
    "value": "123masterkey4567890",
    "encrypted": false
  },
  "functionKeys": [
    {
      "name": "default",
      "value": "abc123functionkey4567890",
      "encrypted": false
    }
  ],
  "systemKeys": []
}
```

[My expansion](https://stackoverflow.com/a/79656424) on the accepted answer adds some detail but the long and short of it is that this setup will allow you to
get a local funtion app up and running and you can test it with the tool of your choice. My personal favorite is [Postman](https://www.postman.com/). Enjoy, and I hope this helps someone.

# Random Thought of the Day
I was exchanging texts with one of my stepsons a few months ago on Father's Day, and it went a little something like this:

[him]: Happy Father's Day

[me]: Thank you, same to you. Got any plans today?

[him]: I'm working 

[me]: I am pulling weeds. Wish I was working.

This got me to thinking. No matter what are you doing and whether or not you enjoy it, somewhere in the world is someone else who wishes they were doing what you are doing.
Imagine you live in a desert and your *dream* is to live somewhere that has so much green, you sometimes have to get rid of some of it!

[^1]: spoiler alert: this one is [me](https://stackoverflow.com/a/79656424/4616451) and is the reason for this post.