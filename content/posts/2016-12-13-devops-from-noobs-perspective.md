+++
title = "DevOps from a noob's perspective"
date = 2016-12-13T00:00:00+00:00
+++

I am involved in some official project where I got a chance to set up web
infrastructure. This was the first time I did any kind of system
administration work so it has been an adventure.

Here are some lessons I learned as I did and continue to do this work.

## Read the fine manual (RTFM)
Be ready to invest lots of time reading up documentation. For web applications:

* Read up on your hosting provider and how network security works.
* Read up on servers (web/proxy/database) you are using and how
  they handle requests.
* Read up on storage components you are administering like database
  servers.

Having this knowledge handy can make recovery easier and less stressful.
For example, data errors are especially costly and a bug might make you
desperate as you are looking for some magical band-aid on StackOverflow
in the middle of an outage.

Common thinking goes something like "We don't have time to read documentation,
let's release our product first". And this is fine if you want to throw away your
code and your company in a couple of years. If you are in for some long-term project though,
the errors you commit here pile on and on. Think of it as a high leverage investment.
Reading up about this stuff will help to deal with emergencies much easier
because you have the knowledge of internal workings on your fingertips and you already
know a few failure modes of the system.

Scenario: I just ran a `DELETE` SQL query without qualifying `WHERE` what to do?!
Oh, just stop the database, do a Point-In-Time-Recovery (PITR) and restart it
again. This will cause downtime but still better than losing data.

## Use the most boring tools possible
What I mean by that is to use the most solid, middle of the road technologies
possible. This point has some nuances. The risk you can afford depends on the
component you are talking about. If we are talking about a tool that measures
system resources like CPU, RAM, and NIC traffic, you can probably experiment a
bit. But for something like your primary data store, you absolutely can't
afford to take risks because if your data gets corrupted/deleted the recovery
might be impossible. The idea here is the less the cost of your failures,
the more risk you can afford.

While I was doing infra, some people suggested Docker. And Docker looks good,
judging by its hype. I didn't use it because I don't know how docker replaces
processes and how security works in Docker (Docker might not be replacing
processes at all).

## Knowledge of operating systems comes in handy
Knowing how kernel operates lets you know

* Why running more than one process on a box might improve its utilization.
* Why threads don't suck and event-driven programming is not a panacea.
* Why threads in python suck if you have some user-level thread library that has preemptive scheduling.
* Why reading from disk is the same as reading from network and event loops in Linux
  might not help there.
* Why using HTTP client libraries for internal calls make no sense.

Another less concrete benefit is it can make you spend less money on your infra.
This is particularly important for companies that hope to make a profit anytime
soon. And contrary to many people's opinion, saving money on infra is not
time-consuming. It comes a lot from intuitive judgment as you get more and more
familiar with this subject.

## Invest in infrastructure automation
Having your infrastructure automated using tools like Chef, Puppet, or
Ansible brings these benefits.

* It helps in having identical machines/software and thus identical failure
  modes.
* The configurations can be reviewed by someone else.

If you have your infra automated, the only hard work you will do is whenever you
are setting up some component that is different from your existing infra.
Another way to say that is, going from `0` to `1` is difficult but going from
`1` to `n` is smooth as silk.

## Have logging and monitoring in place
This is obvious to anyone who has written a `Hello, World` program. But it's
still pretty important to have a comprehensive logging system and it brings
in some benefits.

* Error analysis and postmortem of outages.
* How requests are distributed across endpoints and where to focus your
  optimization efforts.
* Replaying requests to a new system before you make it online.
* Ability to do live experiments.

The more verbose your logs are, the better. The ideal set up is having logs sent to
a different centralized server with a beefy disk, and have all logs time
ordered. Just writing this made my mouth water (yeah, I haven't done it yet).

## Never do infra stuff when you are tired
As a culture, we like to burn our midnight oil. Here's the deadline, few more
steps and you will be done. Or maybe your boss has forced you in lockdown mode.
Or maybe the company will sink if you can't get this done in the next couple of
hours.

It might still be fine if you are debugging something. However, doing the
infra stuff while you are tired can be catastrophic.

Even if you are doing all the nice things like automating and versioning your
infra, it is hard to review. In my experience, reviewing infra code is harder
then application code. And infra bugs are harder to find in running systems
and are much more costly.

In my opinion, doing it while your mind is functioning at an optimal level
shows your respect towards infra work.

## Still learning
I still have a long way to go I will definitely think about these things
differently as I get more experienced. But having my thoughts persisted
somewhere will help in retrospection later.
