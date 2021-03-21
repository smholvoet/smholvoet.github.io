---
published: true
title: "Using REST APIs with curl"
excerpt: "Using REST APIs with curl"
date: 2021-03-21T00:00:00-04:00
show_date: true
tags:
  - curl
  - REST
  - API
---
![curl-logo](/assets/images/curl-logo.svg){: .align-center}

This article digs deeper into how you can use [curl](https://curl.se/) to call REST API's, which you would typically do from within a bash script.
If you're looking for a way to do the same from within PowerShell might be better off using either[`Invoke-WebRequest`](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-webrequest) or [`Invoke-RestMethod`](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-restmethod).

## ➰ curl?

As you might already know, curl is a command-line tool which is used to *transfer data*. Due to its lightweight nature, it's actually broadly used in places you wouldn't necessarily suspect:

> curl is also used in cars, television sets, routers, printers, audio equipment, mobile phones, tablets, settop boxes, media players and is the Internet transfer engine for thousands of software applications in over ten billion installations. curl is used daily by virtually every Internet-using human on the globe.

A very basic curl command could be as simple as:

```bash
curl https://curl.se
```

The above command "connects" (performs a `GET`) to curl's very own website and returns the entire HTTP response, which will be a HTML page.

Making API calls using curl is not *that* different, but obviously requires a few more parameters.

## ☎️ Calling a REST API using curl

The basic sample below uses [GitHub's REST API](https://docs.github.com/en/rest/overview) to fetch information from a GitHub [account](https://docs.github.com/en/rest/reference/users#get-a-user).

```bash
curl \
     -H "Accept: application/vnd.github.v3+json" \
     https://api.github.com/users/smholvoet
```

The only additional parameter which where specifying here is the `-H` (or `--header`) flag. Depending on the endpoint you're sending requests to, specific headers will have to be included in your request. In the example above we're adding the `accept` key with a value of `application/vnd.github.v3+json`. Multiple headers can be passed by simply specifying said parameter multiple times:

```bash
curl \
     -H "header_1: value_1" \
     -H "header_2: value_2" \
     ...
     -H "header_n: value_n" \
     https://your.api.com/endpoint
```

### HTTP verbs

The `-X` (or `--request`) option tells curl which HTTP verb to use. You may have noticed this parameter is nowhere to be found in the initial sample. The request method defaults to `GET`, so leaving it out means curl will send a request using the default `GET` HTTP verb. Specify `-X` followed by `POST`, `PUT`, `DELETE` if you're manipulating data instead of simply fetching something:

```bash
curl \
     -X POST \
     https://your.api.com/endpoint
```

### Verbose

The `-v` (or `--verbose`) flag shows us what's happening under the hood. Useful for debugging purposes, or making sure you don't lose your sanity upon slipping a typo into your endpoint URL 🙊 like  in the example below.

```bash
* Could not resolve host: api.githubxyz.com
* Closing connection 0
curl: (6) Could not resolve host: api.githubxyz.com
```

### Authentication

Most API's will require some form of authentication. curl handles authentication by using the `-u` (or `--user`) option. 
You could specify the username and password:

```bash
curl \
     -u <username>:<password> \
     https://your.api.com/endpoint
```

However, instead of using a password, most modern API's will push you towards the use of a personal access token (PAT) or an access token which you typically request from an authentication provider.

```bash
curl \
     -u <username>:<token> \
     https://your.api.com/endpoint
```

A couple of examples which make use of such tokens are:

- [Azure DevOps (PAT)](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate)
- [GitHub (PAT)](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token)
- [Microsoft Graph](https://docs.microsoft.com/en-us/graph/auth/auth-concepts#access-tokens)

### Request body

Some API endpoints might require a `body`, usually in [JSON](https://www.json.org/json-en.html) format. That is where the `-d` (or `--data`) option comes in.
The below example expects us to:

- ✅ pass data in JSON format 👉 `-H "Content-Type: application/json"`
- ✅ manipulate data instead of fetching data 👉 `-X POST`
- ✅ pass a request body containing a value for 2 attributes: *color* and *animal*

```bash
curl \
     -H "Content-Type: application/json" \
     -X POST \
     -d '{"color":"pink","animal":"panther"}' \
     https://your.api.com/endpoint
```

Instead of passing the request body in your actual curl command, you could also store data in a separate JSON file:

```json
{
   "color":"pink",
   "animal":"panther"
}
```

...and in turn point to said file:

```bash
curl \
     -H "Content-Type: application/json" \
     -X POST \
     -d data.json \
     https://your.api.com/endpoint
```

### Storing output

Instead of outputting the response to your shell, you can easily save it to a file by using the `-o` or `--output` option. The below example would output the return message of a request to a 📑 `output.json` file.

```bash
curl \
     -o output.json \
     https://your.api.com/endpoint
```

I hope the above examples gave you a basic idea on how to utilize curl in order to make REST calls.
The full 📚 man page of curl can be found over at [curl.se](https://curl.se/docs/manpage.html).
