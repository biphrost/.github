# Biphrost

Biphrost is a next-generation app hosting platform, built by humans, for the benefit of other humans.


## Introduction

### Feed your technolust

Biphrost uses **LXC** containers, managed by a **[literate documentation](https://en.wikipedia.org/wiki/Literate_programming)** system, with a simple API, and an interactive, animated, colorful web interface inspired by the sci-fi and cyberpunk aesthetics of cinema.

### Ethos and mission statements

Biphrost is an app hosting platform for [some of the best self-hostable software available](https://github.com/awesome-selfhosted/awesome-selfhosted).

But it is also a rejection of so much of the current fashion in business.

Biphrost's goal is to empower users to affordably and easily explore more of the wonderful world of free software that's available to them, to encourage them to leave exploitative, dystopian megacompanies like Facebook and Google, and to have fun and be part of a community while doing it.

In service of this goal:
* Biphrost charges just enough to fund continued development and maintenance;
* Nearly all of Biphrost is open source software, allowing other people to use its components to build similar platforms of their own;
* There is a strong focus on end-user experience;
* The platform is designed to be managed by experienced systems administrators, but is also highly automated for long-term stability.

### Vibe

Biphrost is also a love letter to the style of the [Mirrorshades](https://en.wikipedia.org/wiki/Mirrorshades) era of cyberpunk, along with inspirations from retrocomputing and the halcyon days of the late 90s BBS scene. Biphrost comes from a universe where Atari remained a dominant gaming platform, where more people reacted to the cancerous growth of megacorps by building fun hacker communities and making their own tools, where the future still looked dystopian but also less *boring*.

It's not for everyone.

It's not meant to be.


## Architecture

### LXC

Biphrost uses [LXC](https://linuxcontainers.org/lxc/introduction/) instead of Docker for containerization (scroll down if your first question is "why?"). LXC is configured here to run *unprivileged* containers, to further reduce the risks associated with a compromised container.

Each container gets its own unprivileged user account on the host server.

#### Incus

The first versions of Biphrost were built before Incus existed, but also before there was good support for LXD in Debian. As a result, some tooling was built around managing the LXC containers. Incus looks really good, and I want to start testing it soon. See the roadmap, below.

### Shell

The Biphrost [shell](https://github.com/biphrost/shell) is a collection of systems administration tools that are used by system administrators ("sysops") to easily deploy, reconfigure, and decommission LXC containers.

The shell is a collection of *literate programming* documents, each written in Markdown. Another tool, [Golem](https://github.com/robsheldon/golem), translates these documents into executable shell scripts.

Each shell command is designed to be *idempotent*: a sysop can rerun any of the commands on a specified container as needed, and it will only make changes to the container on the first run. This allows containers to be easily updated with newer configurations, and makes the shell commands overall much safer to run.

The shell is *meant* to be run by human systems administrators. Biphrost has an API, and container mamagement will almost always be done through the API. But, occasionally, a sysop might need to hand-hold a deployment or troubleshoot a broken configuration; they should have advanced tooling at their disposal. For this reason, the API acts as a thin wrapper around the Biphrost shell.

The Biphrost shell requires root on the host, and assumes that the operator is authorized to make changes to any container on the host.

### API

An API is being developed, using Laravel. More to come.

### UI

A web-based UI is in progress.

Meeting Biphrost's stylistic aspirations quickly collided with limitations in CSS. CSS has become an amazing system for building interactive web-based user interfaces, and many of its early problems have been satisfiably addressed. Still, the Biphrost UI is a step into something more advanced than the typical web application interface.

SVG solves the challenges presented by a CSS-only UI.

So, the web-based UI is a thin wrapper around the API, and is responsible for rendering API responses in fun, stylish, and animated ways.


## Roadmap

* Conclude a period of importing a number of deployments from another service (in progress, until about mid-March)
* Active defense
    * AI bot crawlers are becoming a severe nuisance
        * Add [Nepenthes](https://zadzmo.org/code/nepenthes/) to the current configuration (March)
    * Automatically detect, deny, and frustrate brute-force login attacks against common web applications
    * Use log monitoring to automatically flag IPs and networks that are scanning for common vulnerabilities
    * WordPress-specific:
        * Build out enough security at the edge that Wordfence, Akismet, and similar plugins are unnecessary for hosted sites
* Tune nginx SSL configuration
    * Should be able to shave a few ms off each request
* API first release (April/May)
* UI build-out
* Speed up deployments
    * `biphrost deploy base lxc --apache --php --mysql` should finish in under a minute
    * May need to implement local mirrors of some packages
* Centralize logging
    * rsyslog from containers to host
    * neutralize journald
    * mysql slow query logging on by default
* Deployment commands for the first batch of applications
* Invite codes for selected first adopters
* IRC server set up, chat with a sysop via web interface
* Monitoring, observability, analytics integrated into API and web UI

### TODOs and wishlist

- [] The UI is designed to be themeable. Add themes for:
   - [] Basic or bare text, or a TUI-like interface
   - [] A more traditional, bland web-based interface
   - [] A hand-drawn-like theme, using https://roughjs.com/


## Why...?

### Why didn't you use Docker?

I've used Docker while employed at a couple of larger companies. It's... *okay*. Docker really shines best when you have elastic, short-lived, repeatable workloads. For example, you want someone to be able to upload a document and then do a lot of post-upload processing on that document. That's a fine job for Docker.

But, while at those companies, I also saw an enormous amount of engineering effort get wasted trying to troubleshoot Docker-specific issues, usually in the form of some kind of unexpected behavior. It does not work as well for longer-lived containers, and Dockerfiles generally (or completely?) follow a single-process model, which presents a lot of challenges for hosting web applications, which often need some combination of web server, application software, and database.

I explored LXC years ago and liked it immediately. I probably *could* make this work with Docker, and there would have been some advantages (lots of web applications now have a premade Dockerfile), but I didn't want to, because I don't enjoy working with it.

Also, Dockerfiles tend to be developed not so much by experienced systems administrators, but by developers that were often cribbing together a few articles from around the web on "how to" get something-or-other working. I've seen entirely too many Dockerfiles with outdated server software configurations or security or performance problems.

I have a great deal of respect for the skills of experienced Linux systems administrators, and occasionally count myself among their number, and I wanted an architecture that emphasized that respect for a finely tuned server.

### Why didn't you use Ansible/Puppet/Chef/Terraform... anything other than, what is this thing, "Golem"?

Yeah, valid question.

There are lots of ways to do infrastructure-as-code, and most DevOps practitioners would prefer to see any of those, rather than what we do here.

1. I like shell programming, actually.
2. There are inherent limitations in YAML-based instance management systems and their kin.
   * A lot of the Biphrost shell commands do some form of sanity-checking;
   * Sometimes they allow a sysop to make a decision;
   * They are *composable*: a deployment is a process that combines multiple shell commands together, which allows for deduplication in the shell commands, which follows the Don't Repeat Yourself principle in development, and this is hard to do with some of the other approaches;
3. Biphrost's shell commands can be used to reconfigure running containers without taking them down or redeploying them, or sometimes, interrupting them at all.
4. I built Golem kind of as a joke and then I started using it for some things and then I started to like it.

The current approach is working quite well. If it stops working well, I'll switch to something else.

