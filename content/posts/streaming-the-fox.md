---
date: '2025-04-08T00:00:00-06:00'
title: 'Streaming the Fox — Replicating Firefox history with Litestream'

ShowToc: false
---

The following steps walk through how to replicate the Firefox browser history using Litestream, a streaming replication tool for SQLite databases.

1. Follow the [Litestream / Getting Started / Prerequisites](https://litestream.io/getting-started/) steps (“Install Litestream & SQLite” and “Setting up MinIO”)

2. Copy the Firefox SQLite database to your dekstop:
```shell
❯ cp "$HOME/Library/Application Support/Firefox/Profiles/*release*/places.db" $HOME/places.db
```

3. Replicate the database to object storage (MinIO) with Litestream:
```shell
❯ export LITESTREAM_ACCESS_KEY_ID=minioadmin
❯ export LITESTREAM_SECRET_ACCESS_KEY=minioadmin
❯ litestream replicate places.sqlite s3://mybkt.localhost:9000/places.db
2025/04/08 21:11:53 INFO litestream version=v0.3.13
2025/04/08 21:11:53 INFO initialized db path=/Users/joe/Desktop/places.sqlite
2025/04/08 21:11:53 INFO replicating to name=s3 type=s3 sync-interval=1s bucket=mybkt path=places.db region=us-east-1 endpoint=http://localhost:9000
```

4. Restore a copy of the database from object storage with Litestream:
```shell
❯ export LITESTREAM_ACCESS_KEY_ID=minioadmin
❯ export LITESTREAM_SECRET_ACCESS_KEY=minioadmin
❯ litestream restore -o places_copy.db s3://mybkt.localhost:9000/places.db
2025/04/08 20:56:17 INFO restoring snapshot replica=s3 generation=bbe87a28733f79e7 index=0 path=places_copy.db.tmp
2025/04/08 20:56:17 INFO restoring wal files replica=s3 generation=bbe87a28733f79e7 index_min=0 index_max=0
2025/04/08 20:56:17 INFO downloaded wal replica=s3 generation=bbe87a28733f79e7 index=0 elapsed=3.611136ms
2025/04/08 20:56:17 INFO applied wal replica=s3 generation=bbe87a28733f79e7 index=0 elapsed=5.923536ms
2025/04/08 20:56:17 INFO renaming database from temporary location replica=s3
```

5. List the tables in the copied database:
```shell
❯ sqlite3 places_copy.db
SQLite version 3.43.2 2023-10-10 13:08:14
Enter ".help" for usage hints.

sqlite❯ .tables
_litestream_lock                    moz_items_annos
_litestream_seq                     moz_keywords
moz_anno_attributes                 moz_meta
moz_annos                           moz_origins
moz_bookmarks                       moz_places
moz_bookmarks_deleted               moz_places_extra
moz_historyvisits                   moz_places_metadata
moz_historyvisits_extra             moz_places_metadata_search_queries
moz_inputhistory                    moz_previews_tombstones
```

{{< figure
  src="/images/2025-04-08-minio-bucket.png"
  caption="MinIO console showing the bucket"
>}}

{{< figure
  src="/images/2025-04-08-minio-db.png"
  caption="MinIO console showing the replicated SQLite database"
>}}

### Resources
- [Litestream](https://litestream.io/)
- [MinIO](https://min.io/)
- [Firefox `places/Database.cpp` source code](https://searchfox.org/mozilla-release/source/toolkit/components/places/Database.cpp)
