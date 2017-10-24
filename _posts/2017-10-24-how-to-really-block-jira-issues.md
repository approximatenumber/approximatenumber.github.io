---
layout: post
title: JIRA`s blocking does not really block issues
tags: jira script-runner
comments: True
excerpt_separator: <!--more-->
---

![image](images/blocked_issue.png)

We thought that linking JIRA issue as blocking another one is really blocking from closing this another one. It\`s obviously that *blocked* task cannot be resolved before *blocking* task is resolved, yep? But by default there is no functionality in JIRA to process blocking issues. Link statuses like `is blocked by` or `blocks` are just usual issue field value.

I can help you to solve this problem. It could be easy done with **ScriptRunner for JIRA**.

<!--more-->

Go to your workflow configuration. Find the closing transition (we use *Resolve* transition), and add new condition "**JQL query match**" with this value:

`NOT (issueFunction in hasLinks("is blocked by")) OR (issueFunction in hasLinks("is blocked by") AND issueFunction in linkedIssuesOf("status = 'RESOLVED'", "blocks"))`

So, it means that transition of blocked issue cannot be performed if blocking issue status is not `RESOLVED`.