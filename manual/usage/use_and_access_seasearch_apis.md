# Use and Access SeaSearch APIs

This document briefly describes how to use SeaSearch services through SeaSearch APIs. We usually recommend using **regular users** to perform document indexing and search operations. Therefore, this document will cover the following contents:

- How to get an authorization token (auth token, therafter)

- How to create regular users using an administrator account

- How regular users can create, use, and delete indexes

For other SeaSearch APIs, you can refer [here](../api/overview.md) for the details

## Get auth token

SeaSearch's auth token is using **base64 encode** consist of `username` and `password`: 

```sh
echo -n 'username:password' | base64
dXNlcm5hbWU6cGFzc3dvcmQ=
```

Please take your auth token to access SeaSearch APIs by adding it into `headers`, i.e.,

```json
headers = {
    "Authorization": "Basic dXNlcm5hbWU6cGFzc3dvcmQ="
}
```

## Create a role for the regular user

In this case, we will create a role names `regular_user` with permissions ***index***, ***document*** and ***search*** for the regular users in SeaSearch system. You can send a **POST** request to `/api/role` to create a role, i.e.,:

```sh
curl -X 'POST' \
'https://seasearch.exmaple.com/api/role' \
-H 'accept: application/json' \
-H 'authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=' \
-H 'Content-Type: application/json' \
-d '{
"_id": "regular_user",
"permission":   [
    "index.Create",
    "index.Delete",
    "document.Create",
    "document.Get",
    "document.Update",
    "document.Delete",
    "search.SearchV1"
],
"role": "regular_user"
}'
```

Example response if the role with id `regular_user` creates successfully

```json
{
"message": "ok",
"id": "regular_user"
}
```

!!! tip "About SeaSearch APIs permissions"

    In SeaSearch, user permissions are divided by roles. Administrators (especially those generated during initialization) usually have the highest permissions (i.e., they can access all APIs). These permissions mainly involve the permission for the following operations:

    - **auth** (user opearations)
    - **index**
    - **search**
    - **document**
    - **elastic** (Elasitc data flow operations)

    You can send a **GET** request to `/api/permissions` to list all permissions

## Create a regular user

You can create a regular user by **POST `/api/user`** with the role `regular_user`, e.g.,

```sh
curl -X 'POST' \
  'http://<your IP>:4080/api/user' \
  -H 'accept: application/json' \
  -H 'authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=' \
  -H 'Content-Type: application/json' \
  -d '{
  "_id": "ru_username",
  "password": "ru_password",
  "role": "regular_user"
}'
```

After submitting a **POST** to `/api/user`, you will recive a response

```json
{
  "message": "ok",
  "id": "ru_username"
}
```


## Access SeaSearch APIs via regular user

**Similar to administrator accounts**, the regular users also need to get an auth token **before** accessing the relevant APIs:

```sh
echo -n 'ru_username:ru_password' | base64
bmV3dXNlcm5hbWU6Q29tcGxleHBhc3MjMTIz
```

Here we will take regular users creating, using and deleting an index as examples, for the whole descriptions of SeaSearch APIs, please refer to [here](../api/overview.md).

### Create an index

You can create an index by **PUT `/Index_name`** with related settings, for exmaple (we assume the index name is `index_1`):

```sh
curl -X 'POST' \
  'http://<your IP>:4080/es/index_1' \
  -H 'accept: application/json' \
  -H 'authorization: Basic cnVfdXNlcm5hbWU6cnVfcGFzc3dvcmQ=' \
  -H 'Content-Type: application/json' \
  -d '{  
  "mappings:{
    "name":{
        "type":"text",
        "highlightable":true
    }
 }    
}'
```

Example response if index `index_1` creates successfully

```json
{
  "message": "ok",
  "index": "index_1",
  "storage_type": "disk"
}
```

### Create a document

You can create a document by sending a **POST** request to `/es/<your index name>/_doc`. For example, we add a book names *Snow Crash* by the author *Neal Stephenson*:

```sh
curl -X 'POST' \
  'http://<your IP>:4080/es/index_1/_doc' \
  -H 'accept: application/json' \
  -H 'authorization: Basic cnVfdXNlcm5hbWU6cnVfcGFzc3dvcmQ=' \
  -H 'Content-Type: application/json' \
  -d '{
  "name": "Snow Crash",
  "author": "Neal Stephenson",
  "release_date": "1992-06-01",
  "page_count": 470
}'
```

Example response:

```json
{
  "message": "ok",
  "id": "2evTtrGhVf2",
  "_id": "2evTtrGhVf2",
  "_index": "index_1",
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 0,
  "result": "created"
}
```

In this case, you will get an id `2evTtrGhVf2` for this book. Also, you can add multiple documents at the same time by **POST** `/api/_bulk` (see [Bulk Document API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html))

### Search document by querying matched fields

When you have stored a considerable number of documents in an index in SeaSearch (such as `index_1` above), you can use **GET** `/api/<your index name>/_search` to search for the required document by the fields you want to match:

```sh
curl -X 'POST' \
  'http://<your IP>:4080/es/index_1/_search' \
  -H 'accept: application/json' \
  -H 'authorization: Basic cnVfdXNlcm5hbWU6cnVfcGFzc3dvcmQ=' \
  -H 'Content-Type: application/json' \
  -d '{
  "query": {
    "match": {
      "name": "Snow"
    }
  }
}'
```

The response lists up to 10 records matching the query results:

```json
{
  "took": 1,
  "timed_out": false, 
  "hits": {
    "total": {
      "value": 1
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "index_1",
        "_type": "_doc",
        "_id": "2evTtrGhVf2",
        "_score": 1,
        "_source": {
          "author": "Neal Stephenson",
          "name": "Snow Crash",
          "page_count": 470,
          "release_date": "1992-06-01"
        }
      }
    ]
  }
}
```

### Delete document

You can delete a document by **DELETE** `/api/<your index name>/_doc/<your document id>`, e.g.:

```sh
curl -X 'DELETE' \
  'http://<your IP>:4080/es/index_1/_doc/2evSk96OVLa' \
  -H 'accept: application/json' \
  -H 'authorization: Basic cnVfdXNlcm5hbWU6cnVfcGFzc3dvcmQ='
```

and a reponse with ***deleted*** message will recive if the document deletes successfully:

```js
{
  "message": "deleted",
  "index": "index_1",
  "id": "2evSk96OVLa"
}
```

### Delete index

You can delete an index by sending a **DELETE** request to `/api/index/<your index name>`, e.g.:

```sh
curl -X 'DELETE' \
  'http://<your IP>:4080/api/index/index_1' \
  -H 'accept: application/json' \
  -H 'authorization: Basic cnVfdXNlcm5hbWU6cnVfcGFzc3dvcmQ='
```

and you will recive the following message after the index has been deleted:

```json
{
  "message": "deleted"
}
```

!!! danger
    When you delete an index,** all documents in the index will be deleted**. Please **make sure** that the data is no longer needed **before** performing this operation.
