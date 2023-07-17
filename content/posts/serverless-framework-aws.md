---
title: "Finding productivity with the Serverless Framework and AWS Lambda"
date: 2023-07-03T08:31:43+07:00
draft: false # Set 'false' to publish
description: "In this article, I'll switch my Golang function from an OpenWhisk implementation to the Serverless Framework and AWS Lambda"
categories:
- devops
tags:
- cloud

---

In this article:

- [From OpenWhisk to AWS Lambda](#from-openwhisk-to-aws-lambda)
- [What's Devops already?](#whats-devops-already)
- [The Golang Functions API issue](#the-golang-functions-api-issue)
- [The logging issue](#the-logging-issue)


## From OpenWhisk to AWS Lambda

In my [previous article]({{< ref serverless-do.md >}}), I was writing about how I've written a Golang function on Digital-Ocean's OpenWhisk implementation, and all the issues I found along the way. I couldn't keep it that way, I had to move on to a better solution.

Having tried the [Serverless.com platform](https://www.serverless.com) a long time ago, I remember it made it easy and fast to go from `zero` to `deployed`, so that's what I picked for todays article: I'll be migrating the function I wrote from OpenWhisk to AWS Lambda.

## What's Devops already?

When using OpenWhisk on Digital Ocean's platform, there was no mention of devops. I just created and deployed my function with the `doctl` CLI tool.

In the case of AWS, as they say:

>Great AWS power comes with great AWS complexity

As you'll see along this article, AWS Lambda is much more structured and organised. One consequence is that deploying a Lambda is not as easy as typing a single command... or is it?

That's where devops plays a role: automating cloud infrastructure provisioning and making hard things simple. I hesitated between various options for this article, and Serverless.com wasn't the only option. I could have used Pulumi, or CDK, or Terraform... but Serverless.com was probably the easiest to try first.
I'm very tempted to try Pulumi next as it can be implemented in Go.

In our case, Serverless.com will make it easy to write our Lambda and get it deployed, with API Gateway, Logging, and all the AWS niceties. It will abstract away all this complexity and handle CloudFormation configuration for us.

## THE CODE

In the end, it's still the thing to focus on. 

Serverless.com lambdas are still AWS lambdas, so you can count on all documentation and examples you can find online, which is a big plus.

Contrary to OpenWhisk which abused Go's flexibility with a fully untyped API, AWS made more reasonable API choices. The API is typed, well organised and documented.

The handler function type is the following.

{{< highlight go "linenos=false" >}}
func Handler(
	ctx context.Context, 
	event events.APIGatewayProxyRequest,
) (*Response, error) {
{{< / highlight >}}

The actual type of the `event` parameter depends on the type of lambda you're using, which is a surprise to me... I would have thought a lambda could have processed different types of events, knowing they will all have a payload. 

I would have expected separate parameters for the payload and, let's call it that, the invocation context (API Gateway, SQS, etc). Maybe the event type can be type-checked at runtime to know in which context the function is invoked.

Yet even with a type specifi to API Gateway, we're not getting much help in terms of request processing: I end up having to decode the event payload myself.

{{< highlight go "linenos=false" >}}
type Input struct {
	Hostname string `json:"hostname"`
	Ip       string `json:"ip"`
	Port     string `json:"port"`
}

func Handler(ctx context.Context, event events.APIGatewayProxyRequest) (*Response, error) {
	var input Input
	b := []byte(event.Body)
	if event.IsBase64Encoded {
		rawBody, err := base64.StdEncoding.DecodeString(event.Body)
		if err != nil {
			return nil, err
		}
		b = rawBody
	}
	if err := json.Unmarshal(b, &input); err != nil {
		return nil, err
	}
	.../...
}
{{< / highlight >}}

That's a lot of phony boilerplate code, and the base64 decoding part is clearly a leaky abstraction! Also my code is handling a small portion of what proper body handling requires... HTTP frameworks have been handling this for me for years!

## Functions flaws

Serverless platforms feel like they're assuming I'll ever write only one or two functions. 

They take the `cgi-bin` analogy too far I think. These scripts were called as executable programs, with `stdin` and `stdout` holding the request and response, and environemnt variables for parsed HTTP request data.

Serverless functions reproduce this model when they insist on putting a single function in an executable. They're not even invoking the executable once per request, unlike `cgi-bin`, they're more like `FastCGI` in this respect. Since they implement an intermediate socket-based protocol, they could use HTTP instead, or at least include some routing!

The platforms miss the opportunity to group functions in an executable then use one activation to respond to an array of different requests.

The real value of serverless functions lies in scale-to-zero and the billing model.

.../... more to come