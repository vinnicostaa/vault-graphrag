---
title: "settings.local (json)"
type: source
tags: [code, json, converted]
source: 50-projects/master-sindico/.claude/settings.local.json
converted: 2026-04-22
---

# settings.local

```json
{
  "permissions": {
    "allow": [
      "WebFetch(domain:github.com)",
      "WebFetch(domain:zitadel.com)",
      "WebFetch(domain:docs.sqlc.dev)",
      "WebFetch(domain:docs.stripe.com)",
      "WebSearch",
      "Bash(gh repo *)",
      "Bash(gh api *)",
      "WebFetch(domain:context7.com)",
      "WebFetch(domain:www.npmjs.com)",
      "Bash(curl -s https://registry.npmjs.org/@stripe/mcp/latest)",
      "Bash(python3 -c \"import sys,json; d=json.load\\(sys.stdin\\); print\\('version:', d.get\\('version'\\)\\); print\\('published:', d.get\\('_npmUser',{}\\).get\\('name'\\), d.get\\('time'\\)\\); print\\('bin:', d.get\\('bin'\\)\\); print\\('description:', d.get\\('description'\\)\\)\")",
      "Bash(curl -s \"https://registry.npmjs.org/@stripe%2Fagent-toolkit\")",
      "Bash(python3 -c \"import sys,json; d=json.load\\(sys.stdin\\); print\\('name:', d.get\\('name'\\)\\); print\\('latest:', d.get\\('dist-tags',{}\\).get\\('latest'\\)\\); print\\('desc:', d.get\\('description'\\)\\)\")",
      "Bash(curl -s \"https://registry.npmjs.org/@playwright%2Fmcp\")",
      "Bash(python3 -c \"import sys,json; d=json.load\\(sys.stdin\\); print\\('latest:', d.get\\('dist-tags',{}\\).get\\('latest'\\)\\)\")",
      "Bash(curl -s \"https://registry.npmjs.org/tavily-mcp\")",
      "Bash(curl -s \"https://registry.npmjs.org/@stripe%2Fmcp\")",
      "Bash(python3 -c \"import sys,json; d=json.load\\(sys.stdin\\); print\\('name:', d.get\\('name'\\)\\); print\\('latest:', d.get\\('dist-tags',{}\\).get\\('latest'\\)\\); print\\('desc:', d.get\\('description'\\)\\); v=d.get\\('versions',{}\\).get\\(d.get\\('dist-tags',{}\\).get\\('latest'\\),{}\\); print\\('bin:', v.get\\('bin'\\)\\); print\\('dependencies:', list\\(\\(v.get\\('dependencies'\\) or {}\\).keys\\(\\)\\)[:5]\\)\")",
      "Bash(python3 -c \"import sys,json; d=json.load\\(sys.stdin\\); print\\([i.get\\('name'\\) for i in d]\\)\")"
    ]
  },
  "outputStyle": "default",
  "sandbox": {
    "enabled": false,
    "autoAllowBashIfSandboxed": false
  }
}

```
