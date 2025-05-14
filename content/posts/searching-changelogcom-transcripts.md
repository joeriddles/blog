---
date: '2025-04-07T00:00:00-06:00'
title: 'Searching Changelog.com Transcripts'
---

I’ve found myself becoming an avid listener of the Changelog podcast of late. I enjoy Jerod and Adam’s interview style and the topics. Recently I went poking around in the [Changelog’s GitHub repos](https://github.com/thechangelog/) and found a repo containing transcripts from all episodes.

Something else I discovered recently is [Typesense](https://typesense.org/), an “open source alternative to Algolia”. Unlike a tool like [lunr](https://lunrjs.com/), which uses JavaScript for client-side searching, Typesense requires running a dedicated server for search, or using their hosted cloud SaaS (Search-as-a-Service).

So, I built a simple web using Typesense and the Changelog transcripts. You can see it here: [https://joeriddles.github.io/changelog-transcript-search/](https://joeriddles.github.io/changelog-transcript-search/).

{{< figure
  src="/images/2025-04-07-flyio.png"
  caption="Searching the Changelog transcripts for mentions of Fly.io"
>}}

You can run this all locally using the steps in the [README](https://github.com/joeriddles/changelog-transcript-search). The UI is built with bog-standard React and Algolia’s open source UI library, [React Instance](https://www.algolia.com/doc/guides/building-search-ui/what-is-instantsearch/react/) with Typesense’s [adapter](https://github.com/typesense/typesense-instantsearch-adapter). The UI library has drop-in components for searching, filtering, and paging.

The most interesting part of this simple project is a Python script to transform and upload the Markdown transcript files to the Typesense server. Even that code is straightforward, if ugly.
