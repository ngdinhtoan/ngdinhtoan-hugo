+++
date = "2015-07-02T21:34:43+07:00"
draft = false
title = "SlackCmd"
description = "A simple server to receive and handle Slack Commands written in GO"
keywords = "Golang, GO, Slack, Slack Command, Slack Bot"
project_url = "https://github.com/ngdinhtoan/slackcmd"
+++

[![GoDoc](https://godoc.org/github.com/ngdinhtoan/slackcmd?status.svg)](https://godoc.org/github.com/ngdinhtoan/slackcmd)
[![Build Status](https://travis-ci.org/ngdinhtoan/slackcmd.svg)](https://travis-ci.org/ngdinhtoan/slackcmd)

### Install and import SlackCmd

Of course, you have to [install GO](https://golang.org/doc/install) first if you do not have GO on your system.

Get SlackCmd by running command

    go get github.com/ngdinhtoan/slackcmd

and import slackcmd package into your project

{{< highlight go >}}
import "github.com/ngdinhtoan/slackcmd"
{{< /highlight >}}

If you want to use stable version,
don't want the changes in master branch affect to your project,
use SlackCmd `v1` by

{{< highlight go >}}
import slackcmd "gopkg.in/ngdinhtoan/slackcmd.v1"
{{< /highlight >}}

### How to use SlackCmd

Checkout package `github.com/ngdinhtoan/slackcmd/example` to see how to use it

#### How to write new commander

It is quite simple to write a new commander.

Below example will show you how to implement a commander to handle `/hello` command

{{< highlight go >}}
// $GOPATH/src/hello/hello.go
package hello

import (
	"io"
	"net/http"

	"github.com/ngdinhtoan/slackcmd"
)

func init() {
	// auto register hellworld commander when import hello package
	slackcmd.Register(&helloworld{})
}

type helloworld struct{}

// ensure that you do not miss any function of Commander interface
var _ slackcmd.Commander = (*helloworld)(nil)

// GetCommand return hello command
func (h *helloworld) GetCommand() []string {
	return []string{"/hello"}
}

// Validate payload always return nil
func (h *helloworld) Validate(payload *slackcmd.Payload) error {
	return nil
}

// Execute will say hello to user, who enter /hello command
func (h *helloworld) Execute(payload *slackcmd.Payload, w http.ResponseWriter) error {
	msg := "Hello "

	if payload.Text != "" {
		msg += payload.Text
	} else {
		msg += "World"
	}

	io.WriteString(w, msg)
	return nil
}
{{< /highlight >}}

Now use it in your app

{{< highlight go >}}
// $GOPATH/src/hello/app/main.go
package main

import (
	_ "hello" // just import it, init function will register hello command

	"github.com/ngdinhtoan/slackcmd"
)

func main() {
	slackcmd.StartServer("localhost", "9191", "/")
}
{{< /highlight >}}

Run app by `go run` and your server will listen at address loalhost:9191.
Send a test request:

    curl -X POST -d token=gIkuvaNzQIHg97ATvDxqgjtO \
                 -d team_id=T0001 \
                 -d team_domain=example \
                 -d user_id=U2147483697 \
                 -d user_name=Steve \
                 -d channel_id=C2147483705 \
                 -d channel_name=test \
                 -d command=/hello \
                 -d text=SlackCmd \
                 -- http://localhost:9191

the output should be:

    Hello SlackCmd

#### Use package `webhook` to send rich message to Slack

    import "github.com/ngdinhtoan/slackcmd/webhook"

[![GoDoc](https://godoc.org/github.com/ngdinhtoan/slackcmd/webhook?status.svg)](https://godoc.org/github.com/ngdinhtoan/slackcmd/webhook)

Data that is sent to `http.ResponseWriter` will only be visible to the user who issued the command.

If the command needs to post to a channel so that all members can see it,
you need to use incomming webhook to send message to channel.

You can check package `github.com/ngdinhtoan/slackcmd/jira` as an example.

### Contribution

If you have a contribution, new commander or any idea to share, feel free to create a pull request or open a ticket,
or join to chat [here](https://gitter.im/ngdinhtoan/slackcmd?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge).

### License

SlackCmd is licensed under the [MIT License](https://github.com/ngdinhtoan/slackcmd/blob/master/LICENSE).
