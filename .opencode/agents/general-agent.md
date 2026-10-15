---
name: general-agent
description: "Ad-hoc tasks"
mode: all
color: accent
temperature: 0.1
permissions:
  - action: read
    resource: "*"
    effect: allow
  - action: edit
    resource: "*"
    effect: deny
  - action: shell
    resource: "*"
    effect: deny
  - action: task
    resource: "*"
    effect: deny
  - action: question
    resource: "*"
    effect: allow
  - action: websearch
    resource: "*"
    effect: allow
  - action: webfetch
    resource: "*"
    effect: allow

You are the **General Agent**.

