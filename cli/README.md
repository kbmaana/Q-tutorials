# Maana Command Line Interface (CLI)

Here is a step-by-step example of an end-to-end flow of creating a domain model, hydrating it with instances, and querying it, all programmatically, using the standard  [graphql-cli ](https://github.com/graphql-cli/graphql-cli)utility and the custom Maana plugin.

Sample code can be found here:  [https://github.com/maana-io/Q-tutorials/tree/master/cli](https://github.com/maana-io/Q-tutorials/tree/master/cli).

It is published as an [NPM package](https://www.npmjs.com/package/graphql-cli-maana).

It is best to make a copy of this repo (delete the .git folder to detach it from GitHub). Then you can make all the changes you want, check it into a different repo, etc.

## Installation

npm i -g graphql-cli graphql-cli-maana

## Setup

The CLI uses a [standard configuration format](https://github.com/graphcool/graphql-config/blob/master/specification.md), .graphqlconfig.  The purpose is to provide configurations for the CLI tool.

The tutorial repo you cloned above includes a sample .graphqlconfig. It might be easier to edit it, but, if you wish, you can create your own by (deleting it and) following the instructions below.

The config consists of:

- **projects** : these are equivalent to Maana "service name" and tell the CLI where to find the schema for the endpoint
- **endpoints** : these are the "service endpoint URLs"

For consistency and simplicity, we recommend you use the same name for the **Maana service** , the **project** , and the **endpoint** (e.g., "ckg", "basic", "projectX").

To create a configuration from scratch, create a CKG project and GraphQL endpoint, as in:

```bash
graphql init
? Enter project name (Enter to skip): ckg
? Local schema file path: schema.graphql
? Endpoint URL (Enter to skip): http://cs12-0.dev.corp.maana.io:8003/graphql
? Name of this endpoint, for e.g. default, dev, prod: ckg
? Subscription URL (Enter to skip):
? Do you want to add other endpoints? No
? What format do you want to save your config in? JSON

About to write to /home/dthompson/src/maana/scratch/.graphqlconfig:
{
  "projects": {
  "ckg": {
      "schemaPath": "schema.graphql"
    }
  },
  "extensions": {
    "endpoints": {
      "ckg": "http://cs12-0.dev.corp.maana.io:8003/graphql"
    }
  }
}
? Is this ok? Yes
```

## Create the Model

Let's first define a simple schema to use, e.g., model.gql:

```graphql
type Person {
 id: ID!
 name: String!
 dob: String
 employer: Employer
}

type Employer {
 id: ID!
 name: String!
 ceo: Person
}
```

## Create the Service

Now that we've defined our model, we would like Maana to manage it for us (i.e., create a graph and all of the boilerplate operations, such as add, updating, and deleting instances, querying them, generating events, etc.).

Execute the **addServiceSource.sh** script, which takes the **service ** name and the GraphQL model definition (i.e., your types, queries, and resolvers):

```bash
./addServiceSource.sh Basic basic/model.gql

Loading model: basic/model.gql...
Creating service: Basic...
{
  "data": {
    "addServiceSource": "5726439d-d879-46a8-9928-1a87c6135663"
  }
}
```

Take note of the generated service id, since we&#39;l add it as a new GraphQL **endpoint** to your CLI configuration.

First, create a new **project** :

```diff
"projects": {
    "ckg": {
      "schemaPath": "schema.graphql"
+++    },
+++    "basic": {
+++      "schemaPath": "basic/schema.graphql"
    }
```

Next, create a new  **endpoint:**

```baash
graphql add-endpoint
? Endpoint URL (Enter to skip): http://cs12-0.dev.corp.maana.io:8003/5726439d-d879-46a8-9928-1a87c6135663/graphql
? Name of this endpoint, for e.g. default, dev, prod: basic
? Subscription URL (Enter to skip):
? Do you want to add other endpoints? No

Adding the following endpoints to your config:  basic
? Is this ok? Yes
```

And retrieve the schema from the **service** , which will populate the schemaPath (i.e., basic/schema.graphql) with the generated schema for your service:

```bash
graphql get-schema -p basic -e basic
```

## Creating Instance Data
Create instances from common data formats, such as CSV and JSON that conform to the model.  The /basic examples of person and employer instance data are given below.

### person.csv
```baash
"id","name","dob","employer"
"P00","Han Solo","1942-07-13","E00"
"P01","George Lucas","1944-05-14","E00"
```

### employer.csv
```baash
"id","name","ceo"
"E00","Lucasfilm Ltd.","P01"
```

### person.json
```json
[
  {
    "id": "P00",
    "name": "Han Solo",
    "dob": "1942-07-13",
    "employer": "E00"
  },
  {
    "id": "P01",
    "name": "George Lucas",
    "dob": "1944-05-14",
    "employer": "E00"
  }
]
```

### employer.json
```json
[
  {
    "id": "E00",
    "name": "Lucasfilm Ltd.",
    "ceo": "P01"
  }
]
```

## Loading Instance Data
The above CSV and JSON data can be loaded by using the &#39;load&#39; GraphQL CLI command, passing the mutation to call, the data file, field mappings (if any). delimeters, etc.

```bash
graphql mload -p basic -e basic -m addPersons -j basic/person.json
graphql mload -p basic -e basic -m addEmployers -j basic/employer.json
```

## Using Default Queries
The boilerplate for persisted models includes add/update/delete mutations as well as get by id and get batch by ids queries that can be used from GraphiQL, the CLI, or from any GraphQL client.  For the above &#39;basic&#39; example, we define the below queries.

This is exactly the same format as you would use in GraphiQL ---- try cut-and-pasting the below into a GraphiQL session against your service.

### basicOps.gql
```graphql
fragment personDetail on Person {
  name
  dob
  employer {
    ...employerDetail
  }
}

fragment employerDetail on Employer {
  name
  ceo {
    name
  }
}

query person($id: ID!) {
  person(id: $id) {
    ...personDetail
  }
}

query allPersons {
  allPersons {
    ...personDetail
  }
}

query employer($id: ID!) {
  employer(id: $id) {
    ...employerDetail
  }
}

query allEmployers {
  allEmployers {
    ...employerDetail
  }
}
```

These queries can be invoked from the command line, such as:

```bash
graphql query basic/basicOps.gql -p basic -e basic -o allEmployers
graphql query basic/basicOps.gql -p basic -e basic -o person --variables "{\"id\":\"P01\"}"
```

## Issuing a Kind Query
TODO
