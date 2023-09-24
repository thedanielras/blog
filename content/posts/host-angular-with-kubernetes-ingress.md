---
title: "Hosting an Agular app with Kubernetes Ingress at any URL prefix"
date: 2023-09-24T16:30:41+03:00
categories:
    - Deployment strategies
tags:
    - kubernetes
    - angular
    - ingress
---

## Problem/Motivation

Before Kubernetes, we used to deploy our apps in docker swarm, and generally, each ui app had it's own port which you could specify in the URL to get to the needed app. 

Then we had the requirement to deploy our apps in Kubernetes with ingress with **url prefixes**, which meant no more ports besides HTTP (80) and HTTP (443), and the UI apps should now be accessible only by url of type

* https://www.domain.com/app_1_prefix/
* https://www.domain.com/app_2_prefix/

where */app_1_prefix/* and */app_2_prefix/* are the URL prefixes that serve as the base path of app1 and app2 respectively.

These prefixes are set up in the ingress configuration, which basically acts as a reverse proxy rule that routes the request to the container app hosting the ui server with content files.

The problem is that if we would not take care of this by adapting the code, only the index.html would load, and all the other resources would fail with a 404 status code because their content would be loaded not from *domain.com/app_1_prefix/main.js* for example but from *domain.com/main.js*.

## Short solution summary

The solution is to build the angular application using a template URL argument for the parameters --base-href, --deploy-url, which later at the runtime will be substituted by the real base href URL from an environment variable.

## Solution

We have the base tag which we could use as a way to tell the browser to prefix all relative URL sources within our code with the value of the href attribute in the base tag.
```html
<base href="https://www.domain.com/app_1_prefix/"> <!-- Absolute to domain -->
<base href="/app_1_prefix/"> <!-- Relative to domain -->
```

The same with tweaks can be achieved using Angular ```--base-href``` and ```--deploy-url``` build arguments.

If we would set the prefix manually in the index.html or use the angular build command attributes we would not be able to use the app with another prefix unless we would update the value and rebuild the app.

The best way we could approach this is to build the app using some template string as the base href value, and then substitute it with the real value taken from an environment variable at the runtime before the app starts.

The following is a step-by-step guide on how to achieve this.

## Step-by-step guide on how to adapt an angular app to have runtime configurable base href

1. Use the angular build command with the following attributes, which will translate to the base tag content.

```sh
ng build --prod --base-href=template-base-href-string --deploy-url=template-base-href-string
```

2. Add the following variables to the ```environments.ts```, this way you will be able to use it in the code in custom cases when the base href is not supported (icon URLs).

```
baseHrefUrl: '${UI_BASE_HREF_URL}'

# If your UI app uses an OAuth server for authentication you could also make it's address configurable
oauthIssuerUrl: '${UI_OAUTH_ISSUER_URL}',
```

3. Update your dockerfile to install the required dependencies

```
# Needed for envsubst command in run.sh
RUN apt update
RUN apt install gettext-base
```

and set your entry point to run the following shell lines which will do the substitution of the template value used for the deploy-url and base-href arguments

```sh
#!/usr/bin/env sh

# find files with template string value
variableFiles=$(grep -lrE '\$\{UI_BASE_HREF_URL\}|\$\{UI_OAUTH_ISSUER_URL\}' wwwroot)

# replace variables in files
for f in ${variableFiles}; do
envsubst '${UI_BASE_HREF_URL},${UI_OAUTH_ISSUER_URL}' < "$f" > "${f}.tmp" && mv "${f}.tmp" "$f"
done

```

4. Define the environment variables in Kubernetes deployment and voila, you have an ui app with base url prefix configurable by an environment variable.
