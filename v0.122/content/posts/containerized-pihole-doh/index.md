---
title: "Containerized Pi-Hole + DoH on a Raspberry Pi"
date: 2020-02-22
description: "Block ads, trackers, and malware from any local device without having to use an ad-blocker; while securing your DNS traffic at the same time - sounds good!"
cover: banner.png
tags: [raspberry pi, homelab, cloudflare, dns, notes]
comments: true
draft: true
toc: true

---



# Containerized Pi-Hole DNS with Cloudflare DoH (on a Raspberry Pi)

I've been using a Raspberry Pi 3 to run Pi-Hole for my home DNS for a little over 12 months now (with Cloudflare as the upstream DoH provider) and it's been working well. A small issue I have is with the deployment itself: It was done directly on a Raspberry Pi 3. If I want to move this installation to another Pi or re-deploy onto a different device (or maybe within a VM), I need to install and configure it again manually.

While something like Ansible could help here, I'd love to get to the stage where I can deploy my homelab services as an immutable overlay irrespective of the hardware or device underneath. This is where containers come in to play.

In this post, I'll be working through a deployment of Pi-Hole and Cloudflare DoH (cloudflared) using containers; specifically Docker, on my Raspberry Pi 3.

# Deja Vu?

I experimented with Docker on the Pi 3 ~4 years ago, and while it worked, there weren't many container images back then that were ARM compatible. In one instance for a side project I was working on for my employer at the time, I had to follow a chain of 4 Docker images and re-build all of them to be compatible with ARM; just so we could get the original image working on a Pi.

This seems to have changed over the past few years - especially now that Docker has introduced Multi-Arch support

# 1. Install Docker

The following will install the required pre-requisites, configure the Docker Repo, and install Docker CE on a Raspberry Pi.