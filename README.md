# IceDBWorker

IceDB Insert and Merge worker for segment to write directly to it. Supports multiple tables via the URL path (see [req.http](req.http)).

Combine with [danthegoodman1/SegmentProxy](https://github.com/danthegoodman1/SegmentProxy) for a drop-in replacement for the segment JS packages (with a one-line config change).

Uses a custom merge query for deduplication on the `messageId` field.

Used in production at [Tangia](https://www.tangia.co)

## Developing

Run this in the dev container (or a ubuntu install with python 3.11 and docker), then
```
pip install git+https://github.com/danthegoodman1/icedb && pip install -r requirements.txt
```

```
docker compose up -d
cp .env.local .env
```

```
python app.py
```

## Deployment

Set the env vars to be your PG/CRDB DSN, and your S3 info. Set the `AUTH` env var to the bearer token for the `Authorization` header. You should configure the segment webhook sink to send this as well.

The top level keys that are held are:
```
"ts" int
"event" string
"user_id" string
"properties" string
"og_payload" string
```

`og_payload` is the full JSON body (contains dynamic page info, integration info, etc.). `properties` is the `properties` property of the JSON body, which exists if the event is type `track`.

`ts` is a unix ms timestamp.

`event` is named the following way:

```python
if row['type'] == 'page':
    final_row['event'] = "page.{}".format(row["name"])
elif row["type"] == "identify":
    final_row['event'] = "identify"
elif row["type"] == "track":
    final_row["event"] = row["event"]
```

See `app.py` for how rows are partitioned and sorted
