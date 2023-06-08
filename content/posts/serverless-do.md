---
title: "Drawning in Digital Ocean's serverless functions"
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


## Why even use serverless functions

Because I'm in the process of re-creating __SSLping__! 

I had been _"Mr. SSLping.com"_ for 7 years before shutting it down, and I'm missing the sense of purpose it gave me to be offering a free product to the community. I had to shut it down because of the cost of running it, and because of the technical debt that will inevitably hurt any software project. Time passes, and tech choices that once were modern become a liability and make any changes hard, very hard, or _impossible_.

__So why serverless?__ to cut the cost mostly. 

SSLping is implemented as multiple APIs that test different SSL misconfigurations or vulnerabilities, and each one takes a few seconds to probe a website. I'm thinking of using functions to implement all of these so they can scale to zero when not used.

I also want to integrate APIs written in different languages. SSLPing started as a node.js project, but it also relied on docker containers running APIs written in Go. This diversity is actually required, because you need obsolete (!) environments to run obsolete SSL versions (like `SSLv3` which has been long deprecated everywhere). I've even rewritten some of my node.js code in Go to make it smarter, but I still need some (old) node.js!

Functions allow mixing implementation details easily, more easily than a monolith obviously.

## Why Digital Ocean

It's cheap, it's simple, I've used it a lot in the past. Usually, it suits solo founders' needs perfectly. In this precise case, as you'll read below, I was disappointed. 

However you can use [this Link](https://m.do.co/c/813f527beae9) to get $200 in credit valid for 60 days! I'm still using DO and it's still a great option to host your projects!

## The Golang functions API issue

blah too

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

## The logging issue

blah blah
