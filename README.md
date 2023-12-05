# OPA ABAC Examples

This repository includes few examples for using Attribute-Based Access Control policies with [Open Policy Agent](https://www.openpolicyagent.org/docs/latest/).

## How to use?

### Starting OPA Locally

The fastest way to get an OPA server running locally is using docker:
```
docker run -it --rm -p 8181:8181 openpolicyagent/opa run --server --addr :8181
```

This will start a fresh OPA (No policies pre-configured) on a local port (8081).

### List Policies

We can list the currently loaded policies in OPA using cURL.

```
curl -i localhost:8181/v1/policies
```

### Uploading Policies

For each example file provided in this repo, we can upload it to OPA using cURL. For example:

```
curl http://localhost:8181/v1/policies/petshop --upload-file petshop-example.policy
```

### Uploading Data

Data such as user, object, and environment attributes can be provided to OPA either separately using the `v1/data` endpoint or in the same request for authentication. For more information, please refer to the [Data API Documentation](https://www.openpolicyagent.org/docs/latest/rest-api/#data-api).

For the first method, we can upload prepared data for the provided examples using the file with `.data` extension and cURL. For example:

```
curl http://localhost:8181/v1/data --upload-file petshop-example.data
```

### Viewing Uploaded Data

We can query OPA to view the available data using cURL:

```
curl "http://localhost:8181/v1/data?pretty"

{
  "result": {
    "abac": {
      "allow": false
    },
    "pet_attributes": {
      "cat123": {
        "adopted": false,
        "age": 1,
        "breed": "fictitious",
        "name": "cheshire"
      },
      "dog123": {
        "adopted": true,
        "age": 2,
        "breed": "terrier",
        "name": "toto"
      },
      "dog456": {
        "adopted": false,
        "age": 3,
        "breed": "german-shepherd",
        "name": "rintintin"
      },
      "dog789": {
        "adopted": false,
        "age": 2,
        "breed": "collie",
        "name": "lassie"
      }
    },
    "user_attributes": {
      "alice": {
        "tenure": 20,
        "title": "owner"
      },
      "bob": {
        "tenure": 15,
        "title": "employee"
      },
      "dave": {
        "tenure": 5,
        "title": "customer"
      },
      "eve": {
        "tenure": 5,
        "title": "employee"
      }
    }
  }
}

```

### Query the API for authorization decision

Using the endpoint `v1/data`, OPA expects an `input` dict item with the available attributes. For example:

```
curl -X POST "http://localhost:8181/v1/data?pretty" \
   -H 'Content-Type: application/json' \
   -d '{"input":{"user": "bob","action": "read","resource": "dog123"}}'

{
  "result": {
    "abac": {
      "action_is_read": true,
      "allow": true,
      "pet_is_adopted": true,
      "user_is_employee": true,
      "user_is_senior": true
    },
    ...
```

Hint: We can append `explain=full` as a query parameter to get an explanation of the order of evaluation of rules, e.g.:

```
curl -X POST "http://localhost:8181/v1/data?pretty&explain=full" \
   -H 'Content-Type: application/json' \
   -d '{"input":{"user":"eve","action": "delete","resource": "dog123"}}'
```