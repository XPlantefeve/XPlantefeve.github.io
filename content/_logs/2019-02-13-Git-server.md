---
layout: post
title:  13 Feb. 2019
date:   2019-02-13
excerpt: automating deployement
log: true
tag:
  - Deployement
  - Git
  - Hooks
  - Gitea
comments: true
---

As the new McAfee settings forbid the use of the proxy for non browser, Github and Gitlab
are out, found [Gitea](https://gitea.io/) for an internal solution that just works (in
Windows, the department has very few non-windows computers). Things I played with on the
first half of the week:

- Set up a git server
- Interfaced it with AD authentication
- Server installation, update, backup & restore documentation
- Branch protection, to force pull requests & peer review
- Git hooks, and interaction between Bash & Powershell for those
- Local git hooks (don't commit that authenticode signature, man)
- Server git hooks, to automatically deploy validated pull requests
- Git documentation for dummy. Not dummy enough.
