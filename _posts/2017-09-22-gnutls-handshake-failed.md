---
layout: post
title: gnutls_handshake() failed. An unexpected TLS packet was received
tags: linux git jenkins bug
comments: True
excerpt_separator: <!--more-->
---

One day I got this error in our job on Jenkins server:

```
plugins.git.GitException: Failed to fetch from <our-repo-server>.git
	at hudson.plugins.git.GitSCM.fetchFrom(GitSCM.java:817)
	at hudson.plugins.git.GitSCM.retrieveChanges(GitSCM.java:1084)
	at hudson.plugins.git.GitSCM.checkout(GitSCM.java:1115)
	at org.jenkinsci.plugins.workflow.steps.scm.SCMStep.checkout(SCMStep.java:113)
	at org.jenkinsci.plugins.workflow.steps.scm.SCMStep$StepExecutionImpl.run(SCMStep.java:85)
	at org.jenkinsci.plugins.workflow.steps.scm.SCMStep$StepExecutionImpl.run(SCMStep.java:75)
	at org.jenkinsci.plugins.workflow.steps.AbstractSynchronousNonBlockingStepExecution$1$1.call(AbstractSynchronousNonBlockingStepExecution.java:47)
	at hudson.security.ACL.impersonate(ACL.java:260)
	at org.jenkinsci.plugins.workflow.steps.AbstractSynchronousNonBlockingStepExecution$1.run(AbstractSynchronousNonBlockingStepExecution.java:44)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
Caused by: hudson.plugins.git.GitException: Command "git fetch --no-tags --progress <our-repo-server>.git +refs/pull-requests/62/from:refs/remotes/origin/PR-62" returned status code 128:
stdout: 
stderr: fatal: unable to access '<our-repo-server>.git/': gnutls_handshake() failed: An unexpected TLS packet was received.

	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.launchCommandIn(CliGitAPIImpl.java:1924)
	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.launchCommandWithCredentials(CliGitAPIImpl.java:1643)
	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.access$300(CliGitAPIImpl.java:71)
	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl$1.execute(CliGitAPIImpl.java:352)
	at hudson.plugins.git.GitSCM.fetchFrom(GitSCM.java:815)
	... 13 more
```

How to fix it?

<!--more-->

After some googling of this problem I found working solution: you need to rebuild `git` with `libcurl4-openssl-dev` lib instead of `libcurl4-gnutls-dev` by default. Some users tell about some problems with `gnutls` when it is located in **deep** proxy (these strange words is connected with the fact, that our Jenkins is running in docker.) while `openssl` doesn\`t have such problems.

Jenkins is running in docker under `debian:jessie-backports` and `git` package version is `2.11`, so this solution is acceptable for this configuration.

Add these steps to your `Dockerfile` or make them under running container:

```bash
apt-get install -y build-essential fakeroot dpkg-dev libcurl4-openssl-dev
echo deb-src  http://deb.debian.org/debian stretch main > /etc/apt/sources.list.d/sources.list
apt-get build-dep git
mkdir ~/git-openssl
cd ~/git-openssl
apt-get source git
cd git-2.11.0/
sed -i 's/libcurl4-gnutls-dev/libcurl4-openssl-dev/' ./debian/control
apt-get install libcurl4-openssl-dev
dpkg-buildpackage -rfakeroot -b
```

After that you\`ve got `git-<...>.deb` package. Just install it with `dpkg`. That\`s all.
