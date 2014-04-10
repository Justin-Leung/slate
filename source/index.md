---
title: Travis CI - API Reference

language_tabs:
- http
- shell
- ruby
---

# Overview

``` http
GET / HTTP/1.1
User-Agent: MyClient/1.0.0
Accept: application/vnd.travis-ci.2+json
Host: api.travis-ci.org
```

``` http
GET / HTTP/1.1
User-Agent: MyClient/1.0.0
Accept: application/vnd.travis-ci.2+json
Authorization: token "YOUR TRAVIS ACCESS TOKEN"
Host: api.travis-ci.com
```

``` http
GET /api HTTP/1.1
User-Agent: MyClient/1.0.0
Accept: application/vnd.travis-ci.2+json
Authorization: token "YOUR TRAVIS ACCESS TOKEN"
Host: travis.example.com
```

``` shell
$ travis raw /
{"hello"=>"world"}

$ travis raw / --pro
{"hello"=>"world"}

$ travis raw / --api-endpoint https://travis.example.com/api
{"hello"=>"world"}
```

``` ruby
require 'travis'

# talk to travis-ci.org via client object
client = Travis::Client.new
repository = client.repo('travis-ci/travis.rb')

# talk to travis-ci.org via Travis namespace
repository = Travis::Repository.find('travis-ci/travis.rb')

# talk to travis-ci.com via client object
client = Travis::Client.new('https://api.travis-ci.com')
client.access_token = 'YOUR TRAVIS ACCESS TOKEN'
repository = client.repo('travis-pro/billing')

# talk to travis-ci.com via Travis::Pro namespace
Travis::Pro.access_token = 'YOUR TRAVIS ACCESS TOKEN'
repository = Travis::Pro::Repository.find('travis-pro/billing')

# talk to travis.example.com via client object
client = Travis::Client.new('https://travis.example.com/api')
client.access_token = 'YOUR TRAVIS ACCESS TOKEN'
repository = client.repo('my/repo')

# talk to travis.example.com via custom namespace
My = Travis::Client::Namespaces.new('https://travis.example.com/api')
My.access_token = 'YOUR TRAVIS ACCESS TOKEN'
My::Repository.find('my/repo')
```

Welcome to the Travis CI API documentation. This is the API used by the official Travis CI web interface, so everything the web ui is able to do can also be accomplished via the API.

The first thing you will have to find out is the correct API endpoint to use.

* **Travis CI for open source:** For open source projects tested on [travis-ci.org](https://travis-ci.org), use **[https://api.travis-ci.org](https://api.travis-ci.org)**.
* **Travis Pro:** For private projects tested on [travis-ci.com](https://travis-ci.com), use **[https://api.travis-ci.com](https://api.travis-ci.com)**.
* **Travis Enterprise:** For projects running on a custom setup, use **[https://travis.example.com/api]()** (where you replace travis.example.com with the domain Travis CI is running on).

Note that both Pro and Enterprise will require almost all API calls to be [authenticated](#authentication).

# Making Requests

``` http
GET / HTTP/1.1
User-Agent: MyClient/1.0.0
Accept: application/vnd.travis-ci.2+json
Host: api.travis-ci.org
```

``` http
HTTP/1.1 200 OK
Content-Type: application/json

{"hello":"world"}
```

``` shell
$ travis raw /
{"hello":"world"}
```

``` ruby
require 'travis'

# You usually don't want to fire API requests manually
client = Travis::Client.new
client.get_raw('/') # => {"hello"=>"world"}

client.get('/repos/sinatra/sinatra')
# => {"repo"=>#<Travis::Client::Repository: sinatra/sinatra>}
```

<aside class="warning">
  If you do not set the **Accept** header, you might retrieve our old API formats. These are deprecated and will be removed soon.
</aside>

When you write your own Travis CI client, please keep the following in mind:

* Always set the **User-Agent** header. This header is not required right now, but will be in the near future. Assuming your client is called "My Client", and it's current version is 1.0.0, a good value would be `MyClient/1.0.0`. For our command line client running on OS X 10.9 on Ruby 2.1.1, it might look like this: `Travis/1.6.8 (Mac OS X 10.9.2 like Darwin; Ruby 2.1.1; RubyGems 2.0.14) Faraday/0.8.9 Typhoeus/0.6.7`.
* Always set the **Accept** header to `application/vnd.travis-ci.2+json`.

Any existing client library should take care of these for you.

# External APIs

``` http
GET /config HTTP/1.1
User-Agent: MyClient/1.0.0
Accept: application/vnd.travis-ci.2+json
Host: api.travis-ci.org
```


``` http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "config": {
    "host": "travis-ci.org",
    "pusher": { "key": "5df8ac576dcccf4fd076" },
    "github": {
      "api_url": "https://api.github.com",
      "scopes": [
        "read:org", "user:email", "repo_deployment",
        "repo:status", "write:repo_hook"
      ]
    }
  }
}
```

``` shell
$ travis raw /config
{"config"=>
  {"host"=>"travis-ci.org",
   "pusher"=>{"key"=>"5df8ac576dcccf4fd076"},
   "github"=>
    {"api_url"=>"https://api.github.com",
     "scopes"=>
      ["read:org",
       "user:email",
       "repo_deployment",
       "repo:status",
       "write:repo_hook"]}}}
```

``` ruby
require 'travis'
client = Travis::Client.new
client.config
# => {"host"=>"travis-ci.org",
#     "pusher"=>{"key"=>"5df8ac576dcccf4fd076"},
#     "github"=>
#      {"api_url"=>"https://api.github.com",
#       "scopes"=>
#        ["read:org",
#         "user:email",
#         "repo_deployment",
#         "repo:status",
#         "write:repo_hook"]}}
```

Travis CI integrates with external services, some of which your client library might want to interface with directly.
Most prominently, it uses GitHub as source for users, organizations and repositories, and Pusher for live streaming logs.

You can ask the API for the connection details to these services by loading the `config` endpoint.

This includes, amongst other things:

* The GitHub API endpoint (this might be a GitHub enterprise endpoint) and the GitHub scopes currently required by Travis CI. These are the scopes you should use when [generating a temporary GitHub token](#creating-a-temporary-github-token) for authentication.
* The [Pusher](http://pusher.com/) application key. If the setup is running behind a firewall and uses [Slanger](https://github.com/stevegraham/slanger), it will also include the Slanger URL.

# Authentication

``` http
GET /users HTTP/1.1
User-Agent: MyClient/1.0.0
Accept: application/vnd.travis-ci.2+json
Host: api.travis-ci.org
Authorization: token "YOUR TRAVIS ACCESS TOKEN"
```

``` shell
$ travis whoami -t "YOUR TRAVIS ACCESS TOKEN"
```

``` ruby
require 'travis'

# against the Travis namespace
Travis.access_token = 'YOUR TRAVIS ACCESS TOKEN'
Travis::User.current

# against a client object
client = Travis::Client.new(access_token: 'YOUR TRAVIS ACCESS TOKEN')
client.user
````

<aside class="notice">
Do not confuse the <i>access token</i> with the token found on your profile page.
</aside>

To authenticate against Travis CI, you need an API access token.

You can retrieve a token by using a GitHub token to prove who you are. In the future, we are planning to add a proper OAuth handshake for third party applications.

## With a GitHub token

``` http
POST /auth/github HTTP/1.1
User-Agent: MyClient/1.0.0
Accept: application/vnd.travis-ci.2+json
Host: api.travis-ci.org
Content-Type: application/json
Content-Length: 37

{"github_token":"YOUR GITHUB TOKEN"}
```

``` http
HTTP/1.1 200 OK
Content-Type: application/json

{"access_token":"YOUR TRAVIS ACCESS TOKEN"}
```

``` shell
$ travis login --github-token "YOUR GITHUB TOKEN"
Successfully logged in!

$ travis token
Your access token is YOUR TRAVIS ACCESS TOKEN
```

``` ruby
require 'travis'

# against the Travis namespace
Travis.github_auth(token)

# against a client object
client = Travis::Client.new
client.github_auth(token)
```

If you have a GitHub token, you can use the GitHub authentication endpoint to exchange it for an access token.
The Travis API server will not store the token or use it for anything else then verifying the user account.

Note that therefore the API cannot be used to create new user accounts. The user will have to log in at least once via the web interface before interfacing with the API by other means.

## Creating a temporary GitHub token

``` http
POST /authorizations HTTP/1.1
Host: api.github.com
Content-Type: application/json
Authorization: Basic ...

{
  "scopes": [
    "read:org", "user:email", "repo_deployment",
    "repo:status", "write:repo_hook"
  ],
  "note": "temporary token to auth against travis"
}
```

``` http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 1,
  "url": "https://api.github.com/authorizations/1",
  "scopes": [
    "read:org", "user:email", "repo_deployment",
    "repo:status", "write:repo_hook"
  ],
  "token": "YOUR GITHUB TOKEN",
  "note": "temporary token to auth against travis"
}
```

``` http
POST /auth/github HTTP/1.1
User-Agent: MyClient/1.0.0
Accept: application/vnd.travis-ci.2+json
Host: api.travis-ci.org
Content-Type: application/json
Content-Length: 37

{"github_token":"YOUR GITHUB TOKEN"}
```

``` http
HTTP/1.1 200 OK
Content-Type: application/json

{"access_token":"YOUR TRAVIS ACCESS TOKEN"}
```

``` http
DELETE /authorizations/1 HTTP/1.1
Host: api.github.com
Authorization: Basic ...
```

``` shell
$ travis login --auto
Successfully logged in!

$ travis token
Your access token is YOUR TRAVIS ACCESS TOKEN
```

``` ruby
require 'travis'
require 'travis/tools/github'

# drop_token will make the token a temporary one
github = Travis::Tools::Github.new(drop_token: true) do |g|
  g.ask_login    = -> { print("GitHub login:     "); gets }
  g.ask_password = -> { print("Password:         "); gets }
  g.ask_otp      = -> { print("Two-factor token: "); gets }
end

github.with_token do |token|
  Travis.github_auth(token)
end

Travis.access_token # => "YOUR TRAVIS ACCESS TOKEN"
```

Since Travis CI will not store the GitHub token handed to it for authentication, it is possible to generate a temporary GitHub token and remove it again after the authentication handshake.

To create and delete the GitHub token, you can either use the [GitHub web interface](https://github.com/settings/applications) or automate it via the [GitHub API](https://developer.github.com/v3/oauth_authorizations/#create-a-new-authorization).

Make sure your GitHub token has the scopes [required](#external-apis) by Travis CI.

If you automate this process, authentication will become a handshake consisting of three requests:

* Create a GitHub token via the GitHub API, store GitHub token and URL.
* Use the `/auth/github` endpoint to exchange it for an access token. Store the access token.
* Delete the GitHub token via the GitHub API.

Some client libraries will automate this handshake for you.

## GitHub OAuth handshake

> Triggering an OAuth handshake between GitHub and Travis CI:

``` html
<a href="https://api.travis-ci.com/auth/handshake">log in</a>
```

> Running the OAuth handshake in an iframe:

``` html
<script>
  window.addEventListener("message", function(event) {
    console.log("received token: " + event.data.token);
  });

  var iframe = $('<iframe />').hide();
  iframe.appendTo('body');
  iframe.attr('src', "https://api.travis-ci.org/auth/post_message");
</script>
```

You can also trigger a full OAuth handshake between Travis CI and GitHub by opening `/auth/handshake` in a web browser. The endpoint takes an optional `redirect_to` query parameter, which takes a URL the web browser will end up on if the handshake is successful.

There is an alternative version of this that will try to run the handshake in a hidden iframe and using `window.postMessage` to hand the token to the website embedding the iframe. **This endpoint will only work for whitelisted websites.**

# Entities

## Accounts

``` http
GET /accounts HTTP/1.1
User-Agent: MyClient/1.0.0
Accept: application/vnd.travis-ci.2+json
Host: api.travis-ci.org
Authorization: token "YOUR TRAVIS ACCESS TOKEN"
```

``` http
HTTP/1.1 200 OK
Content-Type: application/json

{
   "accounts" : [
      {
         "repos_count" : 167,
         "name" : "Konstantin Haase",
         "type" : "user",
         "id" : 267,
         "login" : "rkh"
      },
      {
         "repos_count" : 70,
         "name" : "Travis CI",
         "type" : "organization",
         "id" : 87,
         "login" : "travis-ci"
      }
   ]
}
```

``` shell
$ travis accounts
rkh (Konstantin Haase): subscribed, 167 repositories
travis-ci (Travis CI): subscribed, 70 repositories
```

``` ruby
require 'travis'
Travis.access_token = 'YOUR TRAVIS ACCESS TOKEN'

Travis.accounts.each do |account|
  puts "#{account.login} has #{account.repos_count}"
end
```

A user might have access to multiple accounts. This is usually the account corresponding to the user directly and one account per GitHub organization.

### Attributes

Attribute   | Description
----------- | -----------
id          | user or organization id
name        | account name on GitHub
login       | account login on GitHub
type        | `user` or `organization`
repos_count | number of repositories
subscribed  | whether or not the account has a valid subscription

The `subscribed` attribute is only available on Travis Pro.

### List accounts

`GET /accounts`

Parameter | Default | Description
--------- | ------- | -----------
all       | false   | whether or not to include accounts the user does not have admin access to

This request always needs to be authenticated.

## Annotations

``` http
GET /jobs/42/annotations HTTP/1.1
User-Agent: MyClient/1.0.0
Accept: application/vnd.travis-ci.2+json
Host: api.travis-ci.org
```

``` http
HTTP/1.1 200 OK
Content-Type: application/json

{
   "annotations" : [
      {
        "id"          : 1,
        "job_id"      : 42,
        "description" : "foo bar",
        "url"         : "http://example.com/job-42",
        "status"      : "passed"
      }
   ]
}
```

``` shell
$ travis show # job info will include annotations
```

``` ruby
require 'travis'

job = Travis::Job.find(42)
job.annotations.each do |annotation|
  puts annotation.description
end
```

<aside class="warning">
  Annotation support is experimental.
</aside>

### Attributes

Attribute   | Description
----------- | -----------
id          | annotation id
job_id      | job id the annotation is for
description | textual description of the annotation
url         | url with more information
status      | annotation status

### List Annotations

`GET /jobs/{job.id}/annotations`

### Create Annotation

`POST /jobs/{job.id}/annotations`

Parameter   | Default | Description
----------- | ------- | -----------
username    |         | user name for provider authentication
key         |         | secret key for provider authentication
description |         | textual description of the annotation
url         |         | url with more information
status      |         | annotation status

## Branches

``` http
GET /repos/rails/rails/branches HTTP/1.1
User-Agent: MyClient/1.0.0
Accept: application/vnd.travis-ci.2+json
Host: api.travis-ci.org
```

``` http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "branches": [
    {
      "id": 22554234,
      "repository_id": 891,
      "commit_id": 6534402,
      "number": "15184",
      "config": {},
      "state": "created",
      "started_at": "2014-04-08T00:17:34Z",
      "finished_at": "2014-04-08T00:48:59Z",
      "duration": 30546,
      "job_ids": [],
      "pull_request": false
    }
  ]
}
```

``` shell
$ travis branches -r rails/rails
v4.1.0:         #15184 created  Dont encourage aliases now that …
master:         #15183 created  Dont abbreviate that which needs…
4-1-0:          #15183 created  Dont encourage aliases now that …
4-1-stable:     #15185 created  Merge branch '4-1-0' into 4-1-st…
adequaterecord: #15158 passed   wrap the literal value before ha…
```

``` ruby
require 'travis'

repository = Travis::Repository.find('rails/rails')
repository.branches
# => {"v4.1.0"          => #<Travis::Client::Build: rails/rails#15184>,
#      "master"         => #<Travis::Client::Build: rails/rails#15183>,
#      "4-1-0"          => #<Travis::Client::Build: rails/rails#15183>,
#      "4-1-stable"     => #<Travis::Client::Build: rails/rails#15185>,
#      "adequaterecord" => #<Travis::Client::Build: rails/rails#15158>,
#      "4-0-stable"     => #<Travis::Client::Build: rails/rails#15143>}
```

The branches API can be used to get information about the latest build on a given branch.

### Attributes

See [builds](#builds).

### List Branches

This will list the latest 25 branches.

`GET /repos/{repository.id}/branches`

`GET /repos/{repository.owner_name}/{repository.name}/branches`

### Show Branch

`GET /repos/{repository.id}/branches/{branch}`

`GET /repos/{repository.owner_name}/{repository.name}/branches/{branch}`

## Broadcasts

``` http
GET /broadcasts HTTP/1.1
User-Agent: MyClient/1.0.0
Accept: application/vnd.travis-ci.2+json
Authorization: token YOUR ACCESS TOKEN
Host: api.travis-ci.org
```

``` http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "broadcasts": [
    {
      "id": 42,
      "message": "We're switching our build infrastructure on April 29."
    }
  ]
}
```

``` shell
$ travis raw /broadcasts
{"broadcasts"=>[{"id"=>451, "message"=>"This is a broadcast!"}]}
```

``` ruby
require 'travis'

Travis::Broadcast.current.each do |broadcast|
  puts broadcast.message
end
```

### Attributes

Attribute   | Description
----------- | -----------
id          | broadcast id
message     | broadcast message

### List Broadcasts

`GET /broadcasts`

This request always needs to be authenticated.

## Builds

``` http
GET /repos/sinatra/sinatra/builds HTTP/1.1
User-Agent: MyClient/1.0.0
Accept: application/vnd.travis-ci.2+json
Host: api.travis-ci.org
```

``` http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "builds": [
    {
      "commit_id": 6534711,
      "config": { },
      "duration": 2648,
      "finished_at": "2014-04-08T19:52:56Z",
      "id": 22555277,
      "job_ids": [22555278, 22555279, 22555280, 22555281],
      "number": "784",
      "pull_request": true,
      "pull_request_number": "1912",
      "pull_request_title": "Example PR",
      "repository_id": 82,
      "started_at": "2014-04-08T19:37:44Z",
      "state": "failed"
    }
  ],
  "jobs": [ ],
  "commits": [ ]
}
```

``` shell
$ travis history
...
$ travis show 15 # show build #15
...
$ travis restart 15
...
$ travis cancel 15
...
```

``` ruby
require 'travis'
Travis.access_token = 'YOUR ACCESS TOKEN'

repository = Travis::Repository.find('my/repo')
repository.each_build do |build|
  # restart all the builds
  build.restart
end
```

### Attributes

Attribute           | Description
------------------- | -----------
id                  | build id
repository_id       | repository id
commit_id           | commit id
number              | build number
pull_request        | true or false
pull_request_title  | PR title if pull_request is true
pull_request_number | PR number if  pull_request is true
config              | build config (secure values and ssh key removed)
state               | build state
started_at          | time the build was started
finished_at         | time the build finished
duration            | build duration
job_ids             | list of job ids in the build matrix

Note that `duration` might not correspond to `finished_at - started_at` if the build was restarted at a later point.

### List Builds

`GET /builds`

Parameter     | Default | Description
------------- | ------- | -----------
ids           |         | list of build ids to fetch
repository_id |         | repository id the build belongs to
slug          |         | repository slug the build belongs to
number        |         | filter by build number, requires slug or repository_id
after_number  |         | list build after a given build number (use for pagination), requires slug or repository_id
event_type    |         | limit build to given event type (`push` or `pull_request`)

You have to supply either `ids`, `repository_id` or `slug`.

`GET /repos/{repository.id}/builds`

Parameter     | Default | Description
------------- | ------- | -----------
number        |         | filter by build number
after_number  |         | list build after a given build number (use for pagination)
event_type    |         | limit build to given event type (`push` or `pull_request`)

`GET /repos/{repository.owner_name}/{repository.name}/builds`

Parameter     | Default | Description
------------- | ------- | -----------
number        |         | filter by build number
after_number  |         | list build after a given build number (use for pagination)
event_type    |         | limit build to given event type (`push` or `pull_request`)

### Show Build

`GET /builds/{build.id}`

`GET /repos/{repository.id}/builds/{build.id}`

`GET /repos/{repository.owner_name}/{repository.name}/builds/{build.id}`

### Cancel Build

`POST /builds/{build.id}/cancel`

This request always needs to be authenticated.

### Restart Build

`POST /builds/{build.id}/restart`

This request always needs to be authenticated.

## Caches

TODO

## Commits

TODO

## Hooks

TODO

## Jobs

TODO

## Logs

TODO

## Permissions

TODO

## Repository Keys

TODO

## Repositories

TODO

## Requests

TODO

## Settings

TODO

## Users

# API Clients

There are a few API clients out there you can use for interacting with the Travis API, rather than manually triggering HTTP requests and parsing the responses.

## Official

The following clients are maintained by the Travis CI team:

* **[travis.rb](https://github.com/travis-ci/travis.rb)**: Command line client and Ruby library
* **[travis-web](https://github.com/travis-ci/travis-web)**: Web interface and JavaScript library, using Ember.js
* **[travis-sso](https://github.com/travis-ci/travis-sso)**: Single-Sign-On Rack middleware for Travis CI applications

## Web Browsers

> API requests using CORS:

``` html
<script>
  // using XMLHttpRequest or XDomainRequest to send an API request
  var req = window.XDomainRequest ? new XDomainRequest() : new XMLHttpRequest();

  if(req) {
    req.open("GET", "https://api.travis-ci.org/", true);
    req.onreadystatechange = function() { alert("it worked!") };
    req.setRequestHeader("Accept", "application/vnd.travis-ci.2+json");
    req.send();
  }
</script>
```

> With jQuery:

``` html
<script>
$.ajax({
  url: "https://api.travis-ci.org/",
  headers: { Accept: "application/vnd.travis-ci.2+json" },
  success: function() { alert("it worked!") }
});
</script>
```

> JSONP

``` html
<script>
  function jsonpCallback() { alert("it worked!") };
</script>
<script src="https://api.travis-ci.org/?callback=jsonpCallback"></script>
```

When writing an in-browser client, you have to circumvent the browser's
[same origin policy](http://en.wikipedia.org/wiki/Same_origin_policy).
Generally, we offer two different approaches for this:
[Cross-Origin Resource Sharing](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) (aka CORS)
and [JSONP](http://en.wikipedia.org/wiki/JSONP). If you don't have any good
reason for using JSONP, we recommend you use CORS.

### Cross-Origin Resource Sharing

All API resources set appropriate headers to allow Cross-Origin requests. Be
aware that on Internet Explorer you might have to use a different interface to
send these requests.

In contrast to JSONP, CORS does not lead to any execution of untrusted code.

Most JavaScript frameworks, like [jQuery](http://jquery.com), take care of CORS
requests for you under the hood, so you can just do a normal *ajax* request.

Our current setup allows the headers `Content-Type`, `Authorization`, `Accept` and the HTTP methods `HEAD`, `GET`, `POST`, `PATCH`, `PUT`, `DELETE`.

### JSONP

You can disable the same origin policy by treating the response as JavaScript.
Supply a `callback` parameter to use this.

This has the potential of code injection, use with caution.


## Third Party

Besides the official clients, there is a range of third party client available, amongst these:

* **[PHP Travis Client](https://github.com/l3l0/php-travis-client)**: PHP client library
* **[Travis Node.js](https://github.com/pwmckenna/node-travis-ci)**: Node.js client library
* **[travis-api-wrapper](https://github.com/cmaujean/travis-api-wrapper)**: Asynchronous Node.js wrapper
* **[travis-ci](https://github.com/mmalecki/node-travis-ci)**: Thin Node.js wrapper
* **[TravisMiner](https://github.com/smcintosh/travisminer)**: Ruby library for mining the Travis API