---
title: "Drawning in OpenWhisk serverless nonsense at Digital Ocean"
date: 2023-06-07T21:55:06+07:00
draft: false # Set 'false' to publish
description: "This article sums up my short and bad experience with DO's serverless functions"
categories:
- devops
tags:
- cloud

---

In this article:

- [Why even use serverless functions](#why-even-use-serverless-functions)
- [Why Digital Ocean](#why-digital-ocean)
- [The Golang Functions API issue](#the-golang-functions-api-issue)
- [The logging issue](#the-logging-issue)


## Why even use serverless functions?

Because I'm in the process of re-creating __SSLping__! 

I had been _"Mr. SSLping.com"_ for 7 years before I had to shut it down, and I'm missing the sense of purpose it gave me to be offering a free product to the community. I had to shut it down because of the cost of running the infrastructure, and because of the technical debt that will inevitably hurt any software project. Time passes, and tech choices that once were modern become a liability and make any changes hard, very hard, or _impossible_.

__So why serverless?__ to cut the cost.

SSLping is implemented as multiple APIs that test different SSL misconfigurations or vulnerabilities, and each one takes a few seconds to probe am HTTPS server. I want to use functions to implement all of these so they can scale to zero when not used.

I also want to integrate APIs written in different languages. SSLPing started as a node.js project, but it also relied on docker containers running checks written in Go. This diversity is actually required, because you need obsolete (!) environments to run obsolete SSL versions (like `SSLv3` which has been long deprecated everywhere). I've rewritten some of my node.js code in Go to make it smarter and more modern, but I still need some (old) node.js too! 

__TL;DR__

- new architecture
- low cost
- scale to zero
- mix languages

## Why Digital Ocean?

It's cheap, it's easy to use, and I've used it a lot in the past. Usually, it suits solo founders' needs perfectly.

__Usually__ is the key word.

Still you can use [this Link](https://m.do.co/c/813f527beae9) to get $200 in credit valid for 60 days! I'm still using DO and it's still a great option to host many projects!

## The Golang functions API issue

I quickly got a function running with DO's Serverless... kind of.

I'm using Go, and OpenWhisk (which DO serverless is based on) has made super weird developer experience choices: it's using an untyped generic API, and a function is not even producing a compilable program!

A Go OpenWhisk function is basically a function with this definition in a `main` package:

{{< highlight go "linenos=false" >}}
func WhateverName(interface{}) interface{} {
	...
}
{{< / highlight >}}

First of all, the compiler will not tell you if your function definition is correct. Because that's the signature of the least typed function I can think of, and that's all there is to the API. The rest you must figure out with the documentation and answers from support.

__How on earth?__

Basically, calling the function will pass a JSON document to the function, and the function will return a JSON document. And OpenWhisk didn't think of any better API than `interface{}`.

The problem is that it can __kind of__ work even if your understanding of the actual API is very wrong. For instance, at first I was returning my reply struct directly. I was able to invoke my function through the web console, and get the correct result! but in reality OpenWhisk _mandates_ that you return the body of the response in a `body` field for the response, otherwise invoking your function on the web will never work.

Here's the structure I ended up using as my return type (instead of `interface{}`).

{{< highlight go "linenos=false" >}}
type Response struct {
	Body       Output            `json:"body"`
	StatusCode string            `json:"statusCode"`
	Headers    map[string]string `json:"headers"`
}
{{< / highlight >}}

By the way, notice how the status code and headers can be specified for the reply. The API doesn't get you this, only docs.

Why did OpenWhisk model it that way? I have no idea: you can be a fan of dynamicly typed languages, but Go isn't one. 

__Use types, don't make me guess!__

For the input type of the function, I had a similar problem. Invocation parameters (JSON or form encoded) will be placed at the root of the input `interface{}`, but an `http` key will hold the method, headers and path used to invoke the function. Again, it's dumb to hide this typed information.

Here's the struct I used to receive parameters and HTTP params.

{{< highlight go "linenos=false" >}}
type Event struct {
	Hostname string `json:"hostname"`
	Ip       string `json:"ip"`
	Port     string `json:"port"`

	Http struct {
		Method  string            `json:"method"`
		Headers map[string]string `json:"headers"`
		Path    string            `json:"path"`
	} `json:"http"`
}
{{< / highlight >}}

And you'll have to repeat this code, more or less, for each and every one of your functions.

Then there's the function signature itself, revisited. Not only did the input and output structures have to be redefined to include the fields specific to OpenWhisk, but the function name and arity of the function are variable too!

I ended up using the following function definition: 

{{< highlight go "linenos=false" >}}
func Check(ctx context.Context, event Event) Response {
	...
}
{{< / highlight >}}

__I can't get worse still, right?__

Sorry, you'll have to add to this that each function is defined in its own directory, which must hold a `main` package. 

So if you need modules (which I do), you'll have to have many modules in the same repository, which no tool likes because it's not standard. How did OpenWhisk reach the conclusion that is was a good idea? Did they learn Go after it was too late?

```
./packages/sslping/check:
total 12
-rw-rw-r-- 1 chris chris 2673 Jun  4 03:28 check.go
-rw-rw-r-- 1 chris chris  204 Jun  2 08:09 go.mod
-rw-rw-r-- 1 chris chris  796 Jun  2 08:09 go.sum
```

When you want to add another function to the sslping package, you'll have to add another go.mod, another main package, and hell will break lose in VSCode.

__TL;DR: This was the worst Golang developer experience ever.__

## The logging issue

Well, you think, __you still got it working__! 

I can invoke the function on the web, it's fast, easy to deploy and all... Functions are marvelous.

__But where are my logs?__ I remember I had logs in the web console when I invoked my function before!
It must be a bug, let's ask DO's support.

Support was efficient, they helped me find the answer: OpenWhisk logs function calls only for one kind of invocation, and it's not web invocation. The function must be invoked through the API (which requires passing a private DO token, which is not an option for most apps) and it must be async invocation, or maybe it's sync invocation: I don't know since I have no idea what the difference could be! and I couldn't find any information for it in the docs.

__So yeah, no logs__.

Who wants logs for their API calls? Who wants logs for their backend? __I do__.

And it is the moment I thought: _"Okay, I must write about this nonsense in my blog, and switch over to another functions platform ASAP"_

__TL;DR: I'm switching to AWS Lambda__
