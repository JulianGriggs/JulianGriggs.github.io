---
title: Mock External Dependencies In Local Development
layout: post
---
At LightStep, we can run our entire application on our laptops when doing local development (well, most of it anyhow). At some point this will no longer be feasible but I’m hoping to ride the wave for as long as possible. I’ve found that running the app locally is a great way to build intimacy with the product we’re building. It has also helped me develop a deep understanding of how the various services fit together to form our complete architecture. However, running a complex app like ours locally comes with challenges.

One challenge is figuring out how to handle integrations with the SaaS providers we rely on for things like database management, product analytics, and email. These SaaS solutions usually come with limitations on the number of seats available and the number of data sources. Additionally, they often require managing secret API keys. These services are usually designed with a customer’s production environment in mind — where an account allows data to be segmented across a few “environments” (staging, production). They are rarely designed to allow each developer on your team to have their own isolated environment. 

Even if they make it possible, it might not be worth doing from a cost or complexity perspective. Product analytics is one example where this often applies. Does every click in the local app really need to be sent through to an externally hosted data sink? Probably not. All told, we are left with a conundrum: we want the locally running app to resemble production as closely as possible, but there are certain limitations we need to contend with. What do we do?

One solution that can work well is to mock out the external dependency in local development. Let’s say you use a third-party email provider for sending emails in your app. For illustration purposes, let’s assume your app solely consists of a main function that sends a single email. It looks something like this:
```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"net/http"
	"os"
)

// ThirdPartyEmailProvider actually sends emails to people.
type ThirdPartyEmailProvider struct {
	APIKey string
}

// Send issues a post request to the ThirdPartyEmailProvider server
// informing it of an email that needs to be sent on our behalf.
func (t ThirdPartyEmailProvider) Send(email *Email) error {
	b, err := json.Marshal(email)
	if err != nil {
		return err
	}
	_, err = http.Post(
		"https://thirdpartyemailprovider.com/api/send-email",
		"application/json",
		bytes.NewBuffer(b),
	)
	return err
}

// Email represents all of the data that goes into a logical email
type Email struct {
	To      []string
	From    string
	Subject string
	Body    string
}

func main() {
	apiKey := os.Getenv("THIRD_PARTY_EMAIL_PROVIDER_API_KEY")
	if len(apiKey) == 0 {
		fmt.Println("no api key provided")
		os.Exit(1)
	}
	e := ThirdPartyEmailProvider{APIKey: apiKey}
	err := e.Send(
		&Email{
			To:      []string{"ppotts@starkindustries.com"},
			From:    "tony@starkindustries.com",
			Subject: "URGENT",
			Body:    "When are you coming home for dinner?",
		},
	)
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
	fmt.Println("email successfully sent")
}
```

In your production environment, it would be no problem to set the API key as an environment variable and this code would run perfectly well. However, let’s assume that the security team won’t allow each developer to maintain a copy of the production account’s API key on their local machine.  Without the API key set in an environment variable, running the app triggers the following error.
```bash
~ go run main.go
no api key provided
Process finished with exit code 1
```
To fix this problem, we can introduce an interface that abstracts the `Send()` method provided by the `ThirdPartyEmailProvider` and supply a custom implementation that we only use in local development.

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"math/rand"
	"net/http"
	"os"
	"path/filepath"
	"time"
)

// Emailer abstracts the sending of emails.
type Emailer interface {
	Send(email *Email) error
}

// ThirdPartyEmailProvider is an Emailer that actually sends emails
// to people.
type ThirdPartyEmailProvider struct {
	APIKey string
}

// Send issues a post request to the ThirdPartyEmailProvider server
// informing it of an email that needs to be sent on our behalf.
func (t ThirdPartyEmailProvider) Send(email *Email) error {
	b, err := json.Marshal(email)
	if err != nil {
		return err
	}
	_, err = http.Post(
		"https://thirdpartyemailprovider.com/api/send-email",
		"application/json",
		bytes.NewBuffer(b),
	)
	return err
}

// Email represents all of the data that goes into a logical email.
type Email struct {
	To      []string
	From    string
	Subject string
	Body    string
}

// LocalDevelopmentEmailProvider is an Emailer used for local
// development situations where the ThirdPartyEmailProvider
// cannot be used.
type LocalDevelopmentEmailProvider struct{}

// Send writes the contents of email to a file at
// /tmp/local_emails/{unix timestamp}-{random integer}.
func (l LocalDevelopmentEmailProvider) Send(email *Email) error {
	dir := "/tmp/local_emails"
	err := os.MkdirAll(dir, 0770)
	if err != nil {
		return err
	}

	b, err := json.Marshal(email)
	if err != nil {
		return err
	}
	path := filepath.Join(
		dir,
		fmt.Sprintf("%d-%d", time.Now().Unix(), rand.Intn(10000)),
	)
	return ioutil.WriteFile(path, b, 0770)
}

func main() {
	var e Emailer
	if os.Getenv("ENV") == "development" {
		e = LocalDevelopmentEmailProvider{}
	} else {
		apiKey := os.Getenv("THIRD_PARTY_EMAIL_PROVIDER_API_KEY")
		if len(apiKey) == 0 {
			fmt.Println("no api key provided")
			os.Exit(1)
		}
		e = ThirdPartyEmailProvider{APIKey: apiKey}
	}
	err := e.Send(
		&Email{
			To:      []string{"ppotts@starkindustries.com"},
			From:    "tony@starkindustries.com",
			Subject: "URGENT",
			Body:    "When are you coming home for dinner?",
		},
	)
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
	fmt.Println("email successfully sent")
}
```
Now, if we set an environment variable named `ENV`  to  `development` during local development, our app will use the `LocalDevelopmentEmailProvider` and successfully send the email.
```bash
~ go run main.go
email successfully sent
Process finished with exit code 0
```

Furthermore, if we want to inspect the contents of the email, all we need to do is read a file on disk.
```bash
~ cat /tmp/local_emails/1576132221-8081
{"To":["ppotts@starkindustries.com"],"From":"tony@starkindustries.com","Subject":"URGENT","Body":"When are you coming home for dinner?"}%
```

In this case, we lose out on the ability to see how the actual email contents get rendered in our email viewer, but we gain the ability to run our code without a tricky external integration. Mocking external dependencies is a powerful technique for working around the various limitations that come from integrating with third party providers from your locally running instance. It’s doesn’t work for every use case but I’ve come across a surprising number where it’s a great solution.
