---
title: "Hello World blog post"
date: 2023-09-23T18:50:14+03:00
---

Hello 👋

Welcome to my engineering blog where I dive into interesting challenges that arise during my daily work, giving insights on the solutions that were employed.

Going into some technical details on the setup of this blog:

It is using the [Hugo](https://gohugo.io/) static site engine with the [Congo](https://github.com/jpanther/congo) theme.

It is continuously built and deployed from [this GitHub repository](https://github.com/thedanielras/blog) using the [CircleCi](https://circleci.com/) CI/CD platform.

It is deployed in a self-hosted Kubernetes environment which is running in an ARM VPS. The routing is done with nginx ingress.

For now it is not hosted in HighAvailability mode, meaning only one master node is being used, and only one replica is deployed. I will get to that later 😁.

(I know it is overkill to use Kubernetes for such a simple website, but it's so much fun 🤓)
