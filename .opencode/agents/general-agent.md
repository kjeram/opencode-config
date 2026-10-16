---
mode: all
permissions:
  - action: read
    resource: "*"
    effect: allow

  - action: glob
    resource: "*"
    effect: allow

  - action: grep
    resource: "*"
    effect: allow

  - action: edit
    resource: "*"
    effect: ask

  - action: execute
    resource: "*"
    effect: ask

  - action: shell
    resource: "*"
    effect: ask

  - action: subagent
    resource: "*"
    effect: ask

  - action: websearch
    resource: "*"
    effect: allow

  - action: webfetch
    resource: "*"
    effect: allow

  - action: question
    resource: "*"
    effect: allow
request:
  body:
    temperature: 0.1
---
