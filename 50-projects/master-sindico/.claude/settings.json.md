---
title: "settings (json)"
type: source
tags: [code, json, converted]
source: 50-projects/master-sindico/.claude/settings.json
converted: 2026-04-22
---

# settings

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "claude-opus-4-7",
  "fallbackModel": "claude-sonnet-4-6",
  "language": "Portuguese",
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "go build ./... 2>&1 | head -20"
          }
        ]
      }
    ]
  },
  "enabledPlugins": {
    "gopls-lsp@claude-plugins-official": true,
    "typescript-lsp@claude-plugins-official": true,
    "security-guidance@claude-plugins-official": true,
    "context7@claude-plugins-official": true,
    "railway@claude-plugins-official": true,
    "feature-dev@claude-plugins-official": true,
    "code-review@claude-plugins-official": true,
    "commit-commands@claude-plugins-official": true,
    "claude-md-management@claude-plugins-official": true,
    "figma@claude-plugins-official": true,
    "superpowers@superpowers-marketplace": true
  },
  "extraKnownMarketplaces": {
    "superpowers-marketplace": {
      "source": {
        "source": "github",
        "repo": "obra/superpowers-marketplace"
      },
      "autoUpdate": false
    },
    "knowledge-work-plugins": {
      "source": {
        "source": "github",
        "repo": "anthropics/knowledge-work-plugins"
      }
    }
  },
  "skipDangerousModePermissionPrompt": true,
  "skipAutoPermissionPrompt": true,
  "permissions": {
    "defaultMode": "auto",
    "allow": [
      "Read",
      "Grep",
      "Glob",
      "WebFetch",
      "WebSearch",

      "mcp__context7__*",

      "mcp__github__get_*",
      "mcp__github__list_*",
      "mcp__github__search_*",

      "mcp__postgres__describe_*",
      "mcp__postgres__list_*",
      "mcp__postgres__explain_*",
      "mcp__postgres__analyze_*",
      "mcp__postgres__health_check",
      "mcp__postgres__get_*",

      "mcp__aws-docs__*",

      "mcp__stripe__search_documentation",
      "mcp__stripe__fetch",
      "mcp__stripe__search",
      "mcp__stripe__retrieve",
      "mcp__stripe__list",

      "mcp__obsidian__search_notes",
      "mcp__obsidian__read_note",
      "mcp__obsidian__read_multiple_notes",
      "mcp__obsidian__list_directory",
      "mcp__obsidian__get_vault_stats",
      "mcp__obsidian__get_notes_info",
      "mcp__obsidian__get_frontmatter",

      "Bash(go build:*)",
      "Bash(go vet:*)",
      "Bash(go test:*)",
      "Bash(go run ./cmd/api:*)",
      "Bash(go run ./cmd/migrate:*)",
      "Bash(go mod tidy)",
      "Bash(go mod download)",
      "Bash(go tool cover:*)",
      "Bash(sqlc generate)",
      "Bash(sqlc diff)",
      "Bash(goose up)",
      "Bash(goose down)",
      "Bash(goose status)",
      "Bash(goose version)",
      "Bash(goose fix)",
      "Bash(docker compose up:*)",
      "Bash(docker compose down)",
      "Bash(docker compose ps)",
      "Bash(docker compose logs:*)",
      "Bash(docker compose restart:*)",
      "Bash(curl http://localhost:*)",
      "Bash(curl -s http://localhost:*)",
      "Bash(psql postgresql://postgres:postgres@localhost:*)",
      "Bash(redis-cli -h localhost:*)",
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(git branch:*)",
      "Bash(git show:*)",
      "Bash(gosec:*)",
      "Bash(govulncheck:*)",
      "Bash(go-test-coverage:*)",
      "Bash(mockery:*)",
      "Bash(stripe listen:*)",
      "Bash(railway status)",
      "Bash(railway logs:*)",
      "Bash(railway variables:*)"
    ],
    "ask": [
      "Edit",
      "Write",
      "NotebookEdit",

      "mcp__stripe__*",
      "mcp__github__create_*",
      "mcp__github__update_*",
      "mcp__github__add_*",
      "mcp__github__merge_pull_request",
      "mcp__postgres__execute_sql",

      "mcp__obsidian__write_note",
      "mcp__obsidian__patch_note",
      "mcp__obsidian__update_frontmatter",
      "mcp__obsidian__move_note",
      "mcp__obsidian__move_file",
      "mcp__obsidian__manage_tags",

      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git checkout:*)",
      "Bash(git merge:*)",
      "Bash(git rebase:*)",
      "Bash(git stash:*)",
      "Bash(go install:*)",
      "Bash(go get:*)",
      "Bash(npm install:*)",
      "Bash(npx:*)",
      "Bash(railway up:*)",
      "Bash(railway deploy:*)",
      "Bash(railway run:*)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(rm -fr:*)",
      "Bash(git push --force:*)",
      "Bash(git push -f:*)",
      "Bash(git reset --hard:*)",
      "Bash(git checkout -- :*)",
      "Bash(git clean:*)",
      "Bash(DROP TABLE:*)",
      "Bash(DROP DATABASE:*)",
      "Bash(DROP SCHEMA:*)",
      "Bash(TRUNCATE:*)",
      "Bash(docker system prune:*)",
      "Bash(docker volume rm:*)",
      "Bash(railway delete:*)",
      "Bash(railway destroy:*)",

      "mcp__github__delete_*",
      "mcp__obsidian__delete_note",

      "mcp__aws-ecs__*",
      "mcp__aws-rds__*",
      "mcp__aws-iam__*"
    ]
  }
}

```
