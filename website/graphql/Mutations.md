Relay Input Object Mutations Specification
------------------------------------------

Relay's support for mutations relies on the GraphQL server exposing
mutation fields in a standardized way. These mutations accept and emit a
identifier string, which allows Relay to track mutations and responses.

All mutations include in their input a `clientMutationId` string, which is then
returned as part of the object returned by the mutation field.

An example of this is the following query:

```graphql
mutation M {
  updateStatus(input: $input) {
    clientMutationId
    status {
      text
    }
  }
}
```

where the provided parameters are:

```json
{
  "input": {
    "clientMutationId": "549b5e7c-0516-4fc9-8944-125401211590",
    "text": "Hello world"
  }
}
```

and the response is:

```json
{
  "updateStatus": {
    "clientMutationId": "549b5e7c-0516-4fc9-8944-125401211590",
    "status": {
      "text": "Hello World!"
    }
  }
}
```

This section of the spec describes the formal requirements around mutations.

# Mutation inputs

In particular, all mutations must expose exactly one argument, named `input`.
This argument's type must be a `NON_NULL` wrapper around an `INPUT_OBJECT`. That
input object type must contain an argument named `clientMutationId`. That
argument must be a non-null `String`.

Clients may use whatever identifier they see fit for their `clientMutationId`s;
Version 4 UUIDs are a reasonable choice.

# Mutation fields

The return type of any mutation field must be an object. That object must
contain a field named `clientMutationId` which is a non-null `String`. The
value of this field must be the value of the `clientMutationId` input argument
defined above.

# Introspection

A server that correctly implements the above requirement will accept the
following introspection query, and return a response that contains the
provided response.

```graphql
{
  __schema {
    mutationType {
      fields {
        type {
          kind
          fields {
            name,
            type {
              kind
              ofType {
                name
                kind
              }
            }
          }
        }
        args {
          name
          type {
            kind
            inputFields {
              name
              type {
                kind
                ofType {
                  name
                  kind
                }
              }
            }
          }
        }
      }
    }
  }
}
```

yields 

```json
{
  "__schema": {
    "mutationType": {
      "fields": [
        // May contain many instances of this
        {
          "type": {
            "kind": "OBJECT",
            "fields": [
              // May contain more fields here.
              {
                "name": "clientMutationId",
                "type": {
                  "kind": "NON_NULL",
                  "ofType": {
                    "name": "String",
                    "kind": "SCALAR"
                  }
                }
              }
            ]
          }
          "args": [
            {
              "name": "input",
              "type": {
                "kind": "INPUT_OBJECT",
                "inputFields": [
                  // May contain more fields here
                  {
                    "name": "clientMutationId",
                    "type": {
                      "kind": "NON_NULL",
                      "ofType": {
                        "name": "String",
                        "kind": "SCALAR"
                      }
                    }
                  }
                ]
              }
            }
          }
        ]
      ]
    }
  }
}
```
