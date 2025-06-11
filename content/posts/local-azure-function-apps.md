+++
title = 'Azure Function Apps Authentication - Local Edition'
date = 2025-06-11
draft = true
+++

## The Scenario

In my position that I started in September of 2024, I have been working in the Azure space. It has been a great learning
experience given that I had been pretty much AWS-only before. And one area that I have explored quite a bit is Azure Function Apps.
They are analogous to AWS Lambdas with their own Microsoft-related quirks.

One of the nice things about them is that Microsoft has released official Docker images for building and running them locally, so you can test out your code changes with a minimum of effort, and they come in a number of languages. Mostly I write them in Python because it is
quick and easy to get rolling with and Python has so many mature modules for doing the boilerplate stuff. 

## The Issue

One thing I ran across was trying to authenticate against my locally running function app in Docker - this [SO question](https://stackoverflow.com/questions/77096900/where-do-i-find-the-function-key-for-a-locally-deployed-azure-function) sums it up nicely. 
Function Apps default to function-level authentication if not specified when creating the app + you can set it per function.
When deployed to Azure, you can go to the portal in Azure and pull the keys for the deployed app and then pass the appropriate key in an HTTP
header called `x-functions-key` when invoking the function app. If you really want to be fine-grained, each function has its own key, too,
in addition to the `master` and `default` keys defined at the app level. And you can annotate a route with admin-only authentication which would require the `master` key.

But what about if you are running the app in a local Docker container?

## TODO Add some code samples here

```python
@app.route(function_name="GetStuff", auth.HttpAuthenticationLevel=FUNCTION)
def get_thing()
    ...
```

## Ideas 
The shortcut suggestions
are things like:
* [Set it to anonymous auth](https://stackoverflow.com/questions/77096900/where-do-i-find-the-function-key-for-a-locally-deployed-azure-function/79051047#79051047) which you have to remember to reset before deploying
* Answers that only apply to the deployed function in Azure
* 

