---
date: 2024-05-07T10:10:58+01:00
title: Getting Started
---

Well, "Hello, World"

I've been wanting to setup a site for me to post musings and share things I've learned recently.

I've spent way too much time working out "how" to do it. Naturally, this setup is over engineered for what it is. Some of that is by design (it is a great way for me to learn/test new things) and some of that is well... I'm an engineer.

So what's the setup?

To "start" I setup (via click ops) an S3 Bucket and Cloudfront Distro. The site itself is built with Hugo ([https://gohugo.io](https://gohugo.io)). I'm using TailwindCSS ([https://tailwindcss.com](https://tailwindcss.com)) and I'm working to build the layouts myself.

Some next steps are:

- Build out a GitHub workflow to publish the site
- Move the AWS resources into Cloudformation and build out GitHub workflows to manage changes
