---
date: '2025-02-11T00:00:00-06:00'
title: TIL — SQLite UPSERT with extends
---

Today I was trying setting up a personal project that uses [sqlc](https://sqlc.dev/), a Go tool that generates code based off defined SQL queries. It's _not_ an ORM, but looks and feels kind of like one. Generally I prefer using ORMs, but none of Go's feel great. After some conversations, I wanted to give sqlc a shot.

Setting up sqlc is straight forward. You define a SQL schema, SQL queries, and a config file. I set it up using SQLite. This worked fine, until I wrote an `UPSERT` query.

```sql
-- name: SavePin :one
INSERT INTO pins (
  title, url, image_url, created_at
) VALUES (
  ?, ?, ?, ?
)
ON CONFLICT (url)
DO UPDATE SET title = ?, image_url = ?
RETURNING *;
```

sqlc happily generated Go code for this query, but at runtime the query failed: `query_test.go:43: missing argument with index 5`.

I also tried using named params, but to no avail. I created an [issue](https://github.com/sqlc-dev/sqlc/issues/3834) in the GitHub repo for sqlc and dove into the source code to try to figure our what was going on.

The root cause of the bug is that the SQL query expects six params, but only four were getting passed in the generated code:

```go
func (q *Queries) SavePin(ctx context.Context, arg SavePinParams) (Pin, error) {
  row := q.db.QueryRowContext(ctx, savePin,
    arg.Title,
    arg.Url,
    arg.ImageUrl,
    arg.CreatedAt,
  )
  var i Pin
  err := row.Scan(
    &i.ID,
    &i.Title,
    &i.Url,
    &i.ImageUrl,
    &i.CreatedAt,
  )
  return i, err
}
```

Manually updating the code to pass `QueryRowContext` the params `arg.Title` and `arg.ImageUrl` fixed the runtime error.

I forked the repo and [wrote a simple, failing test](https://github.com/sqlc-dev/sqlc/compare/main...joeriddles:sqlc:joeriddles/3834) to help demonstrate the error I saw. Taking a second look through the issues in the repo and I saw that [other](https://github.com/sqlc-dev/sqlc/issues/3439) [people](https://github.com/sqlc-dev/sqlc/issues/3508) had ran into similar issues when using SQLite.

I wrote a simple VS Code `launch.json` config to debug the code generation:

```json
{
  "configurations": [
  {
    "name": "debug",
    "type": "go",
    "request": "launch",
    "mode": "debug",
    "program": "${workspaceFolder}/cmd/sqlc/main.go",
    "cwd": "${workspaceFolder}/examples/booktest",
    "args": ["generate"]
  }
  ]
}
```

I also needed to configure VS Code to add `"go.buildTags": "examples"` to `.vscode/settings.json` because all the examples use that build tag.

After stepping line-by-line through the `internal/codegen/golang` package, I realized the error was further upstream, so I set my breakpoints earlier and stepped through the `internal/compiler` package. Eventually, I reached this line:

![Screenshot of VS Code breakpoint on](/images/2025-02-11-debug.png)

It appears that the anaylzer isn't even parsing the `RETURNING` code, nor the `DO UPDATE SET` line. I dug into the SQLite docs to get a better understanding of how the `UPSERT` works. Reading the [_first line_](https://www.sqlite.org/lang_upsert.html#:~:text=upsert%20is%20not%20standard%20sql.) yields the following:

> UPSERT is not standard SQL.

So, it seems that sqlc simply does not currently support parsing the custom `UPSERT` syntax of SQLite. Fortunately, there is a solution. [A few lines down](https://www.sqlite.org/lang_upsert.html#:~:text=to%20use%20the%20value%20that%20would%20have%20been%20inserted%20had%20the%20constraint%20not%20failed%2C%20add%20the%20special%20%22excluded.%22%20table%20qualifier%20to%20the%20column%20name.%20) is the following:

> To use the value that would have been inserted had the constraint not failed, add the special "excluded." table qualifier to the column name.

So I updated my query to the following and everything worked:

```sql
-- name: SavePin :one
INSERT INTO pins (
  title, url, image_url, created_at
) VALUES (
  ?, ?, ?, ?
)
ON CONFLICT (url)
DO UPDATE SET title = excluded.title, image_url = excluded.image_url
RETURNING *;
```

What's the lesson? ~What is the takeaway? Don't mess with Maui when he's on the breakaway.~ Read the docs.
