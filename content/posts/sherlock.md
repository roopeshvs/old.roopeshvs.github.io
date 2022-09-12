+++
title = "How Sherlock knows if you are on Facebook"
date = 2022-09-12
+++

[Sherlock](https://github.com/sherlock-project/sherlock) is a CLI tool that can be used to find usernames across many social networks. Sherlock works by having [a JSON file as the source](https://github.com/sherlock-project/sherlock/blob/master/sherlock/resources/data.json) where a large collection of sites are listed with a few attributes. 

A sample entry from the file looks like this:  

```json
"Reddit": {
    "errorMsg": "Sorry, nobody on Reddit goes by that name.",
    "errorType": "message",
    ...
    "url": "https://www.reddit.com/user/{}",
    "urlMain": "https://www.reddit.com/",
    ...
  }
```

Each site has its own way of handling usernames that do not exist. To know if a particular username is on the site, Sherlock sends a request to the URL defined in the `url` or the `urlProbe` field and evaluates the response depending on the type of error defined in the `errorType` field. The `errorType` field could be one of `status_code` - HTTP Response Error Codes, `message` - HTTP Response Body that says there is no user by that username or `response_url` - Redirecting to a different page if the username is not found, depending on the method appropriate for consistently finding users without false positives. 

Sherlock was using the `status_code` error type to figure out if the username existed on Facebook.

```json
"Facebook": {
    "errorType": "status_code",
    ...
    "url": "https://www.facebook.com/{}",
    "urlMain": "https://www.facebook.com/",
    ...
  }
```

However this proved to be not working consistently when [a user raised a GitHub issue](https://github.com/sherlock-project/sherlock/issues/725) saying Sherlock could not find his username on Facebook. I vivdly remembered Sherlock being able to find my username on Facebook.

Probing into this mystery, I discovered there was a privacy setting on Facebook that disallowed search engines outside of Facebook to link to one's profile.

<p align="center">
<a target='_blank'><img src='https://i.postimg.cc/L4rXh8xg/fb.png' border='0' alt='fb'/></a>
</p>

Only when the search engines were allowed to link to one's facebook profile at `facebook.com/{username}`, the requests to that profile returned `200 OK` otherwise even when a user with that username existed on facebook the response was `404 NOT FOUND`. This is why the status code approach was inconsistent.

One of the trivial ways to bypass this issue would be to login to facebook and then check for the username. However, for Sherlock, authenticating is not an option!

Only if there was a profile path that shows the login page if the username existed not caring about the search engine option, I thought to myself and wondered. With turning on and off the privacy option on my facebook profile, I started testing various paths like `/about`, `/images`, `/photo` and their responses. 

Few minutes into the search I stumbled upon `/videos` and I was able to see my expectations come alive. `facebook.com/{username}/videos` showed the login page when the username existed and only responded with `This page isn't available` when the username was not yet taken on Facebook.

[The site resource file was updated to check](https://github.com/sherlock-project/sherlock/pull/737) for an error message & use the `/videos` path and voila! This now allows Sherlock to check if a username is on facebook without authentication and irrespective of the search engine privacy option they've opted to go with. 

```json
"Facebook": {
    "errorMsg": "This page isn't available",
    "errorType": "message",
    ...
    "urlProbe": "https://www.facebook.com/{}/videos/",
    ...
  }
```

> Elementary

but hey, it works.