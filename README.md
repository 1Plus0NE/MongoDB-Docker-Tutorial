# Guide to Using MongoDB with Docker

## Preparation – Docker (Volumes and Images)

To create a volume (data persistence), where `<VOLUME_NAME>` is the name of the volume:

```bash
docker volume create <VOLUME_NAME>
```

To list existing volumes:

```bash
docker volume ls
```

To list available images:

```bash
docker images
```

## Create and Run the MongoDB Container

To create and initialize the MongoDB container, mounting the `data/db` directory:

```bash
docker run -d -p 27017:27017 --name <CONTAINER_NAME> -v <VOLUME_NAME>:/data/db mongo
```

Explanation:

- `-d` → detached mode (background)
- `-p 27017:27017` → exposes the MongoDB port
- `--name <CONTAINER_NAME>` → container name
- `-v <VOLUME_NAME>:/data/db` → mounts the volume to the data directory
- `mongo` → image used

## Starting and Stopping the Container

Start an existing container:

```bash
docker start <CONTAINER_NAME>
```

Stop a running container:

```bash
docker stop <CONTAINER_NAME>
```

To view running containers:

```bash
docker ps
```

To view container logs:

```bash
docker logs -f <CONTAINER_NAME>
```

## Accessing MongoDB

To enter the container:

```bash
docker exec -it <CONTAINER_NAME> sh
```

To enter the MongoDB shell:

```bash
mongosh
```

## Basic MongoDB Operations

To list databases:

```bash
show dbs
```

To switch databases, for example from `test` to `local`:

```bash
use local
```

To view collections in the current database:

```bash
show collections
```

## Importing JSON Files

For this section, we will use the dataset [jcrpubs](jcrpubs.json) as an example file.

To copy files into the container:

```bash
docker cp jcrpubs.json <CONTAINER_NAME>:/tmp
```

### JSON Formats Supported by MongoDB

1. JSON Array (requires `--jsonArray` during import)

```bash
[
  { ... },
  { ... }
]
```

2. Extended JSON (does not require an additional flag during import)

```bash
{
    
}
```

To import a JSON file:

```bash
mongoimport -d pubs -c pubs /tmp/jcrpubs.json --jsonArray
```

Explanation:

- `-d pubs` → database name
- `-c pubs` → collection name
- `--jsonArray` → required because the file starts with `[ ]`

To switch to the imported database:

```bash
use pubs
```

## Queries and Statistics

For this section, we will consider the imported dataset [jcrpubs](jcrpubs.json).

To display the total number of documents:

```bash
db.pubs.countDocuments()
```

To perform a simple query:

```bash
db.pubs.find("selection", "projection")
```

To list specific fields and sort by year (descending):

```bash
db.pubs.find(
  {},
  {_id:0, title:1, type:1, year:1}
).sort({year:-1})
```

To count documents of type "inproceedings":

```bash
db.pubs.find({type:"inproceedings"}).count()
```

To count documents that are NOT of type "inproceedings":

```bash
db.pubs.find({type:{$ne:"inproceedings"}}).count()
```

Query with multiple conditions:

```bash
db.pubs.find(
  {authors:"Pedro Rangel Henriques", year:"1997"},
  {id:1, title:1, _id:0}
)
```

### Important Note

MongoDB uses `_id` as the primary identifier field.

If the JSON contains:

```bash
"id": "123"
```

MongoDB will expect:

```bash
"_id": "123"
```

Otherwise, MongoDB automatically creates an `_id` field.  
To prevent this from happening, we should rename or create the `_id` field in advance.
