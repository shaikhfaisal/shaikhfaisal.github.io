---
title: Using Jenkins to drive Ansible
layout: default
excerpt: <p>Using Jenkins to drive your Ansible playbooks</p> 
---
### {{page.title}}

A common problem faced by devops teams running ansible is how do you address the following concerns:

1. Ensuring that only authorised personnel are running playbooks against your servers
2. How do you maintain an audit log of the playbooks that were run against your infrastructure
3. How do you prevent 3 devops engineers running different versions of our playbooks against the
   same infrastructure at the same time?


