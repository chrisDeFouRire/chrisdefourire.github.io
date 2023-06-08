---
title: "Drawning in Digital Ocean's serverless functions"
date: 2023-06-07T21:55:06+07:00
draft: false # Set 'false' to publish
description: "Drawning in Digital Ocean's serverless functions"
categories:
- devops
tags:
- cloud

---

## Why serverless functions

blah sslping blah

## The Golang API issue

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
