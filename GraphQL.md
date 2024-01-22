# GraphQL Best Practices

This is a concise document that outlines the best practices for schema definition, proper field definition, documentation, and query methods.  Consider this a guideline, and a living document.  There is _always_ room for improvement.

Remember, GraphQL is a _Query Language_ first, which means it follows certain naming conventions for queries.  GraphQL is not a programming language, so programming rules mostly do not apply here.  There are a few exceptions.

## Field names

Field names must:

- Be human readable
- Be camelCase
- Not contain abbreviations unless they are commonly known (ID, GPS, DNS, HTTP, etc.)
- Not contain underscores, unless they are internally used variables - in which case, they should _start_ with underscores.
- Have common abbreviated names should be all caps: `ServerDNSAddress`, `IPAddress`, `userID`, `carVIN`

## Comments

Multi-line comments should be commented using triple quotes.  Only use this documentation method if you have multiple lines of comments, either for field names, accessors, mutators, etc.

```graphql
"""
This
Is
A
Multi-line
Comment
"""
type XYZ {
    ID: String!
    fieldName: String
    ...
}
```

Single-line comments should be used when the documentation for a field name, accessor, mutator, is concise:

```graphql
type XYZ {
    "This is an internal ID representing the data payload"
    payloadID: String!
}
```

(Some comments were omitted in the examples below for brevity.)

## Data Types Definitions

Types in GraphQL must:

- Be human readable
- Not contain abbreviations
- Not contain underscores
- Contain names in PascalCase (like `User` vs `user`, `UserProfile` vs `userProfile`, etc.)

Acceptable example:

```graphql
"""
The User is an entity that defines access to a resource through login credentials.  It is identifyable
through their ID and their e-mail address.
"""
type User {
    "The internal ID of the user"
    userID: ID!
    
    "The user's E-Mail Address"
    emailAddress: String!
    
    "Access credentials for this user"
    access: [AccessCredentials]!
}
```

Avoid:

```graphql
type accessCredentials {
    Key: String
    Value: String
    access_type: AccessTypeEnum
}
```

While `AccessTypeEnum` is a valid enumeration type, `Key` and `Value` field names should be avoided as they are not `camelCase`.  `access_type` also does not conform to the Field Name definition standard.  It really should have been called `accessType`.

## Enumerations

Enumerations must be in all caps.  These are considered constant values, and should be treated as such in GraphQL.  Enumerations should end with the name `Enum` to indicate the data type definition.

Acceptable example:

```graphql
enum TaskStateEnum {
    "Indicates a newly created task"
    CREATED
    
    "Indicates an assigned task"
    ASSIGNED
    
    "Indicates a task that is currently in progress"
    IN_PROGRESS
    
    "Indicates a task that has been paused"
    PAUSED
}
```

Avoid:

```graphql
enum TaskState {
    Created
    Assigned
    InProgress
}
```

Why?  While `TaskState` is perfectly acceptable as a naming convention (PascalCase), it indicates the name of a class definition that returns a payload, rather than an enumerated type with known values.

The same rule applies to the values of the enumeration.  While this differs language-to-language (a'la Java vs. TypeScript vs. Rust), using consistent names in GraphQL is desired.  Names in all caps indicate constant values, just as they might in other structured languages.

## Structured Types

As GraphQL builds a graph of data, your schema should be designed in such a way that the data returned is as a graph, not a top level set of objects.

For instance:

```graphql
type User {
    userID: ID!
    emailAddress: String!
    physicalAddress: Address
}

type Address {
    "This is a string because addresses can contain segments a'la '103-100' vs '150'"
    streetNumber: String
    streetName: String
    city: String
    stateProvince: String
    zipcode: String
    ...
}

type Query {
    userRecord(emailAddress: String!): User
}
```

This example has a direct relationship between the `User` and the `Address`.

Avoid:

```graphql
type User {
    userID: ID!
    emailAddress: String!
}

type Address {
    "This is a string because addresses can contain segments a'la '103-100' vs '150'"
    streetNumber: String
    streetName: String
    city: String
    stateProvince: String
    zipcode: String
    ...
}

type UserRecord {
    user: User
    address: Address
}

type Query {
    userRecord(emailAddress: String!): UserRecord
}
```

While the second example will still provide the same information, there is no true relationship between the `Address` and the `User`.  It's indicated as a separate record.

## Queries

When creating queries, it is best practice to use a name to define each query.

When querying data, only query what you need.  Querying unnecessary fields in a data set just adds increased overhead and network traffic that could be otherwise avoided.

### Simple queries

```graphql
query MySimpleQuery {
    getUsersForDomain(domain: "bestpractices.com") {
        user {
            emailAddress
            physicalAddress {
                streetNumber
                streetName
                city
                stateProvince
                zipcode
            }
        }
    }
}
```

Avoid:

```graphql
query {
    getXYZ ...
}
```

While both syntax styles are correct, it is always best practice to name a query.

### Dynamic queries

Here's an example dynamic query:

```graphql
query GetUser($userID: ID!) {
    getUserById(id: $userID) {
        user {
            emailAddress
            physicalAddress {
                ...
            }
        }
    }
}
```

This example gives you the ability to store a query, and programmatically apply it to the GraphQL server by setting a value to the variable, in this case `userID`, and telling the server to execute the query.

Avoid queries like this, unless using a notebook for the query:

```graphql
query getUser {
    getUserById(id: 5) {
        ...
    }
}
```

Although still a valid query, you lose the dynamic nature of the query.  Only perform queries like these if you're using a notebook.
