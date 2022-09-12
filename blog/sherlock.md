+++
title = "How Sherlock knows if you are on Facebook"
date = 2022-09-12
+++

[Sherlock](https://github.com/sherlock-project/sherlock) is a CLI tool that can be used to find usernames across many social networks. It works by using a large json file as the source which contains attributes of several sites. A sample entry from the file looks like this:

```json
"Reddit": {
    "errorMsg": "Sorry, nobody on Reddit goes by that name.",
    "errorType": "message",
    "headers": {
      "accept-language": "en-US,en;q=0.9"
    },
    "url": "https://www.reddit.com/user/{}",
    "urlMain": "https://www.reddit.com/",
    "username_claimed": "blue",
    "username_unclaimed": "noonewouldeverusethis7"
  }
```

On a high level, Sherlock sends a request to the URL defined in the `url` or the `urlProbe` field and evaluates the response depending on the type of error defined in the `errorType` field. The `errorType` field could be one of `status_code` - HTTP Response Error Codes, `message` - HTTP Response Body that says there is no user by that username or `response_url` - Redirecting to a different page if the username is not found.

Sherlock was using the `status_code` error type to figure out if the username existed on Facebook.

```json
"Facebook": {
    "errorType": "status_code",
    "regexCheck": "^[a-zA-Z0-9\\.]{3,49}(?<!\\.com|\\.org|\\.net)$",
    "url": "https://www.facebook.com/{}",
    "urlMain": "https://www.facebook.com/",
    "username_claimed": "blue",
    "username_unclaimed": "noonewouldeverusethis7"
  }
```

However this proved to be not working as expected when a user raised a [GitHub issue](https://github.com/sherlock-project/sherlock/issues/725) saying his username was on Facebook but Sherlock could not find his username. I remembered Sherlock being able to find my username on Facebook (I used to be on Facebook then. :\)) when I used the tool. 

Probing into this anomaly, I discovered there was a privacy setting on Facebook that disallowed search engines outside of Facebook to link to one's profile.

!(image)[https://imgur.com/a/84y0Oac]