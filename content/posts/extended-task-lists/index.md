+++
title = 'Extended Task Lists'
date = 2024-02-25T12:26:30-08:00
tags = ["Today"]
+++

I've been using [Obsidian](https://obsidian.md/) as my note taking app for the [past several months]({{< ref "/hello" >}}). Before that, I would typically just pop open VS Code with a markdown file. Over time, I developed my own system for managing notes and, eventually, my own [plugin](https://obsidian.md/plugins?search=Extended%20Task%20Lists).

## Organization

Most of my notes tend to be relevant to the day or week that I write them. With that being the case, I organize my notes into a `year/month/day` hierarchy. If I were to create a note for today, February 25, 2024, it would look like this in my notes folder:

```shell
2024
└── 02_February
    └── 25_Sunday.md
```

I use the month and date numeric prefixes for file sorting and include the human readable labels for easy scanning.

## Task Lists

I tend to create a lot of task lists in my notes. For example, from April 20, 2023, my notes include:

```markdown
- [x] Text Griffin about book club
- [~] Continue work on Django potluck project... live stream?
- [~] Keep working on Python blog posts
```

The eagle eyed observer will note that `- [~]` is not technically valid Markdown syntax for task lists.[^1] I use it to denote tasks that I DNF'd (did not finish).

The problem with keeping task lists in individual files-per-day is that it's hard to keep track of tasks that were not completed day-to-day. I could copy over any unfinished items to the next day, but I would prefer to not have to do that. I also like the context of knowing the day that I created a task. My solution was to create a Python module to ingest my Markdown files and output a single `TODO.md` file: [`joeriddles/notes`](https://github.com/joeriddles/notes)

## Python module

The Python module is straightforward. You won't find a `requirements.txt` file in this git repo; it's simple enough to not require any dependencies.

The module recursively scans a given path for Markdown files and extracts the task list items within those files. Files can opt-in to being excluded by including a `<!-- exclude TODO -->` comment anywhere in the file. Entire directories can be excluded by containing an `.exclude_todos` file.

Once all the task list items for the path are found, they are simply aggregated into a single `TODO.md` file, grouped by filename. Completed task list items can be optionally excluded. The module also supports a basic watch mode that simply re-runs to whole process once per second.

An example of a generated `TODO.md` file might look like:
```markdown
- [[02_Friday.md]]
    - [ ] Schedule coffee with Scott and discuss
- [[09_Friday.md]]
    - [ ] Review Jeff's PR
```

Using this Python module has worked great for a long time, but since I began using Obsidian a few months ago, it became cumbersome to need to run a separate process to keep my `TODO.md` file up to date. So I made an Obsidian plugin.

## Obsidian plugin

I named the plugin [Extended Task Lists](https://github.com/joeriddles/extended-task-lists) and it's installable today as a community plugin. The plugin solves two main problems:

- Aggregating tasks items from within Obsidian
- Styling non-standard task items

The task aggregation feature in the plugin works basically the same way as the Python module, though it hasn't reached feature parity yet. Open issues can be found on [GitHub Issues](https://github.com/joeriddles/extended-task-lists/issues) for the repo.

I only briefly mentioned the second problem. I use `- [~]` to denote DNF'd tasks and `- [.]` to denote in-progress tasks, but neither are supported markdown syntax. I added some custom logic and CSS classes to style these unofficial task list item states differently:

Without the plugin:
{{< figure src="./images/before.png" >}}

With the plugin:
{{< figure src="./images/after.png" >}}

## Conclusion

So, that's the story of why I created [Extended Task Lists](https://github.com/joeriddles/extended-task-lists)! If it sounds like it would help you manage your tasks in Obsidian, take it for a spin. Feel free to reach out with questions, comments, or bugs via [GitHub Issues](https://github.com/joeriddles/extended-task-lists/issues) or [email](mailto:joe@josephriddle.com).

{{< figure src="./images/logo.png" caption="Extended Task List's zupah kewl logo" >}}

[^1]: Note: there are several Markdown syntaxes though to my knowledge none of them support `- [~]` or `- [.]`. The most popular Markdown syntax seems to be the [GitHub's](https://github.github.com/gfm/#task-list-item-marker).
