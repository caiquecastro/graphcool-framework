---
alias: foatho8aip
description: An overview of the Graphcool service definition file `graphcool.yml` and its YAML structure.
---

# Service definition: `graphcool.yml`

## Overview

The service definition file `graphcool.yml` has the following root properties:

- `types`: References your type definition file(s).
- `functions`: Defines all the [functions](!alias-aiw4aimie9)  you're using in your service.
- `permissions`: Defines all the permission rules for your service.
- `rootTokens`: Lists all the [root token](!alias-eip7ahqu5o#root-tokens) you've configured for your service.

See below for the concrete [YAML structure](#yaml-structure).

## Example

Here is a simple example of a service definition file:

```yml
# Type definitions
types: ./types.graphql


# Functions
functions:

  # Resolver for authentication
  authenticateCustomer:
    handler:
      # Specify a managed function as a handler
      code:
        src: ./src/authenticate.js
        # Define environment variables for function
        environment: 
          SERVICE_TOKEN: aequeitahqu0iu8fae5phoh1joquiegohc9rae3ejahreeciecooz7yoowuwaph7
          STAGE: prod
    type: resolver

  # Operation-before hook to validate an email address
  validateEmail:
    handler:
      # Specify a managed function as a handler; since no environment variables
      # are specified, we don't need `src`
      code: ./src/validateEmail.js        
    type: operationBefore
    operation: Customer.create

  # Subscription to pipe a new message into Slack
  sendSlackMessage:
    handler:
      # Specify a webhook as a handler
      webhook:
        url: http://example.org/sendSlackMessage
        headers:
            Content-Type: application/json
            Authorization: Bearer cha2eiheiphesash3shoofo7eceexaequeebuyaequ1reishiujuu6weisao7ohc
    type: subscription
    query: ./src/sendSlackMessage/newMessage.graphql


# Permission rules
permissions:
# Everyone can read messages
- operation: Message.read

# Only authenticated users can create messages
- operation: Message.create 
  authenticated: true

# To update a message, users need to be authenticated and
# the permission query in `./permissions/updateMessage.graphql`
# has to return `true`
- operation: Message.update 
  authenticated: true
  query: ./permissions/updateMessage.graphql

# To delete a message, users need to be authenticated and
# the permission query in `./permissions/deleteMessage.graphql`
# has to return `true`
- operation: Message.delete
  authenticated: true
  query: ./permissions/deleteMessage.graphql

# Everyone can perform all CRUD operations for customers
- operation: Customer.*


# Root tokens
rootTokens:
  - authenticate
```

This service definition expects the following file structure:

```
.
├── graphcool.yml
├── src
│   ├── authenticate.js
│   ├── validateEmail.js
│   └── sendSlackMessage
│       └── newMessage.graphql
└── permissions
    ├── updateMessage.graphql
    └── deleteMessage.graphql
```


## YAML structure

### Root property: `types`

#### Info

The `types` root property accepts a **single string** or a **list of strings**. Each string references a `.graphql`-file that contains GraphQL type definitions written in the [SDL](https://medium.com/@graphcool/graphql-sdl-schema-definition-language-6755bcb9ce51). 

There are two kinds of types that can be referenced:

- **Model types**: Determine the types that are to be persisted in the database. These types need to be annotated with the `@model`-directive and typically represent entities from the application domain. Read more in the [Database](!alias-viuf8uus7o) chapter.
- **Transient types**: These types are not persisted in the database but typically represent _input_ or _return_ types for certain API operations.

#### Examples

##### Referring to a single type definition file

```yml
types: ./types.graphql
```


##### Referring to multiple type definition files

```yml
types: [./types.graphql, ./customResolver.graphql]
```

or

```yml
types: 
  - ./types.graphql
  - ./customResolver.graphql
```


### Root property: `functions`

#### Info

The `functions` root property accepts a **map from string** (which specifies the function's _name_) **to `function`**.

##### Type: `function`

**All functions** have the following three properties:

- `type`
  - **Type**: `string`
  - **Description:** Determines whether this function is a [resolver](), [subcription]() or a [hook]().
  - **Info:**
    - Required
    - Default: none
    - Possible values: `resolver`, `subscription`, `operationBefore`, `operationAfter`

- `handler`: 
  - **Type**: `handler` (described below)
  - **Description:** Specifies the details of _how_ to invoke the function. Can either contain references to a local file that contains the implementation of the function or otherwise define a webhook that'll be called when the function is invoked.
  - **Info:**
    - Required
    - Default: none
    - Possible values: any

- `isEnabled`: 
  - **Type**: `boolean`
  - **Description:** The function will only be invoked if set to `true`.
  - **Info:**
    - Optional
    - Default: `true`
    - Possible values: `true`, `false`

**Only functions of type `resolver`** have the following property:

- `schema`:
  - **Type**: `string`
  - **Description:** References a `.graphql`-file that contains the extension of the `Query` or `Mutation` type which defines the API of the resolver.
  - **Info:**
    - Optional (if not provided, the extension of `Query` or `Mutation` has to live inside a file that's referenced from the [`types`](#root-property-types) root property)
    - Default: none
    - Possible values: any string that references a `.graphql`-file

**Only functions of type `subscription`** have the following property:

- `query`:
  - **Type**: `string`
  - **Description:** References a `.graphql`-file that contains the _subscription query_ which determines the event upon which the function should be invoked as well as the payload for the event.
  - **Info:**
    - Required
    - Default: none
    - Possible values: any string that references a `.graphql`-file

**Only functions of type `operationBefore` and `operationAfter`** have the following property:

- `operation`:
  - **Type**: `string`
  - **Description:** Describes an operation from the Graphcool CRUD API. The value is composed of the name of a _model type_ and the name of an operation (`read`, `create`, `update` or `delete`), separated by a dot.
  - **Info:**
    - Required
    - Default: none
    - Possible values: `<Model Type>.<Operation>` (e.g. `Customer.create`, `Article.create`, `Image.update`, `Movie.delete`)

##### Structure: `handler`

A `handler` specifies the details of _how_ to invoke the function. It can either be a _managed function_ that references a _local file_ or otherwise define a _webhook_ that'll be called when the function is invoked.

###### Define managed function

**Managed function structure:**

```yml
code: 
  src: <file>
  environment: 
    <variable1>: <value1>
    <variable2>: <value2>
```

Notice that if no environment variables are specified, you can also use the short form without explicitly spelling out `src`:

```yml
code: <file>
```

- `code`:
  - **Type**: `map` (see _managed function structure__ above)
  - **Description:** Describes all the details about how to invoke the managed function and optionally provides environment variables that can be used inside the function at runtime.
  - **Info:**
    - Required (in the case of a managed function)
    - Default: none

- `src`: 
  - **Type**: `string`
  - **Description:** A reference to the file that contains the implementation for the function.
  - **Info:**
    - Required
    - Default: none
    - Possible values: any string that references a valid source file

- `environment`: 
  - **Type**: `[string:string]`
  - **Description:** Specifies a number of environment variables .
  - **Info:**
    - Optional
    - Default: none
    - Possible values: any combination of strings that does not contain the empty string


###### Reference webhook

**Webhook structure:**

```yml
webhook: 
  url: <file>
  headers: 
    <header1>: <value1>
    <header2>: <value2>
```

Notice that if no HTTP headers are specified, you can also use the short form without explicitly spelling out `url`:

```yml
webhook: <file>
```


- `webhook`:
  - **Type**: `map` (see _webhook structure_ above)
  - **Description:** Describes all the details about how to invoke the webhook and optionally specify HTTP headers that will be attached to the request then the webhook is called.
  - **Info:**
    - Required (in the case of a webhook)
    - Default: none

- `url`: 
  - **Type**: `string`
  - **Description:** The HTTP endpoint where the webhook can be invoked.
  - **Info:**
    - Required
    - Default: none
    - Possible values: any string that's a valid HTTP URL and references a webhook

- `headers`: 
  - **Type**: `[string:string]`
  - **Description:** Specifies a number of HTTP headers.
  - **Info:**
    - Optional
    - Default: none
    - Possible values: any combination of strings that does not contain the empty string

#### Examples

```yml
functions:

  authenticateCustomer:
    handler:
      # Specify a managed function as a handler
      code:
        src: ./src/authenticate.js
        # Define environment variables for function
        environment: 
          SERVICE_TOKEN: aequeitahqu0iu8fae5phoh1joquiegohc9rae3ejahreeciecooz7yoowuwaph7
          STAGE: prod
    type: resolver

  # Operation-before hook to validate an email address
  validateEmail:
    handler:
      # Specify a managed function as a handler; since no environment variables
      # are specified, we don't need `src`
      code: ./src/validateEmail.js        
    type: operationBefore
    operation: Customer.create

  # Subscription to pipe a new message into Slack
  sendSlackMessage:
    handler:
      # Specify a webhook as a handler
      webhook:
        url: http://example.org/sendSlackMessage
        headers:
            Content-Type: application/json
            Authorization: Bearer cha2eiheiphesash3shoofo7eceexaequeebuyaequ1reishiujuu6weisao7ohc
    type: subscription
    query: ./src/sendSlackMessage/newMessage.graphql
```

### Root property: `permissions`

#### Info

The `permissions` root property accepts a **list of `permission`s**.

##### Structure: `permission`

**All permissions** have the following three properties:

- `operation`
  - **Type**: `string`
  - **Description:** Specifies for which API operationthis permission holds. Refers to an operation from the Graphcool CRUD API. The value is composed of the name of a _model type_ and the name of an operation (`read`, `create`, `update` or `delete`), separated by a dot.
  - **Info:**
    - Required
    - Default: none
    - Possible values: `<Model Type>.<Operation>` (e.g. `Customer.create`, `Article.create`, `Image.update`, `Movie.delete`)

- `authenticate`: 
  - **Type**: `boolean`
  - **Description:** If set to `true`, only authenticated users will be able to perform the associated `operation`.
  - **Info:**
    - Optional
    - Default: `false`
    - Possible values: `true` or `false`

- `query`: 
  - **Type**: `string`
  - **Description:** References a file that contains a [permission query](!alias-iox3aqu0ee). 
  - **Info:**
    - Optional
    - Default: none
    - Possible values: any string that references a `.graphql`-file

#### Examples

```yml
permissions:
# Everyone can read messages
- operation: Message.read

# Only authenticated users can create messages
- operation: Message.create 
  authenticated: true

# To update a message, users need to be authenticated and
# the permission query in `./permissions/updateMessage.graphql`
# has to return `true`
- operation: Message.update 
  authenticated: true
  query: ./permissions/updateMessage.graphql

# To delete a message, users need to be authenticated and
# the permission query in `./permissions/deleteMessage.graphql`
# has to return `true`
- operation: Message.delete
  authenticated: true
  query: ./permissions/deleteMessage.graphql

# Everyone can perform all CRUD operations for customers
- operation: Customer.*
```

### Root property: `rootTokens`

#### Info

The `rootTokens` property accepts a **list of strings**. Each string is the name of a [root token](!alias-eip7ahqu5o#root-tokens) which will be created whenever the service deployed. 

There are two kinds of types that can be referenced:

- **Model types**: Determine the types that are to be persisted in the database. These types need to be annotated with the `@model`-directive and typically represent entities from the application domain. Read more in the [Database](!alias-viuf8uus7o) chapter.
- **Transient types**: These types are not persisted in the database but typically represent _input_ or _return_ types for certain API operations.

#### Examples

##### Referring to a single type definition file

```yml
types: ./types.graphql
```


##### Referring to multiple type definition files

```yml
types: [./types.graphql, ./customResolver.graphql]
```

or

```yml
types: 
  - ./types.graphql
  - ./customResolver.graphql
```



### Table overview

The YAML configuration file has the following _root properties_:

| Root Property | Type | Description | 
| --------- | ------------------ | --------------- | 
| `types`| `string`<br>`[string]` | Type defintions ([SDL]()) for database models, relations, enums and other types. |
| `functions` | `[string:function]` | All serverless functions that belong to the current service. The key of each element in the dictionary is a unique name for the function, the value specifies details about the function to be invoked. See the `function` type below for more info on the structure. |
| `permissions` | `[permission]` | All permissions rules that belong to the current service. See the `permission` type below for more info on the structure. |
| `modules` | `[string]` | A list of filenames that refer to configuration files of other Graphcool services which are used in the current service (_modules_). |

This is what the additional YAML types look like that are used in the file:

#### Type: `function`

| Property  | Type | Possible Values | Required (default value) | Description|
| --------- | ------------------ | --------------- | --------- | ------------------ | --------------- | --------------- | 
| `isEnabled` | `boolean` | `true`<br>`false` | No (`true`) | The function will only be executed if set to true |
| `handler` | `string`<br>`[string:string]`<br>`[string:webhook]` | any<br>`["webhook":any]`<br>`["webhook":any]` | Yes | Defines how this function should be invoked (i.e. where it lives). Either as a webhook or a local Graphcool function. | 
| `type` | `string` | `subscription`<br>`resolver`<br>`operationBefore`<br>`operationAfter` | Yes | Defines what kind of serverless function this is. |
| `operation` | `string` | `<Model>.<operation>`<br>`<Relation>.<operation>` | Only if `type` is `operationBefore` or `operationAfter` | If the function is set up in the context of an operation, this specifies the concrete operation. |
| `query` | `string` | `<Model>.<operation>`<br>`<Relation>.<operation>` | Only if `type` is `subscription` | If the function is set up as a subscription, this specifies the subscription query (which determines the input type of the function). |
| `schema` | `string` | `<Model>.<operation>`<br>`<Relation>.<operation>` | Only if `type` is `resolver` | If the function is set up as a resolver, this specifies the necessary extensions on the `Query` or `Mutation` type (and potentially additional types that represent the input or return types of the new field).


#### Type: `permission`

| Property  | Type | Possible Values | Required (default value) | Description|
| --------- | ------------------ | --------------- | --------- | ------------------ | --------------- | --------------- | 
| `operation` | `string` | `<Model>.<operation>`<br>`<Relation>.<operation>` | Yes | Specifies the operation for which this permission rule should apply. |
| `authenticated` | `boolean` | `true`<br>`false` | No (`false)` | Specifies any HTTP headers for the request that invokes the function. | Defines whether a request needs to be authenticated to perform the corresponding operation (if true, the `id` of the authenticated node is available as an argument inside the permission query. |
| `query` | `string` | any | No | Specifies the permission query that belongs to this permission rule. |
| `fields` | `[string]` | any | all fields of the model type | Specifies to which fields this permission rule should apply to.

 
#### Type: `webhook`

| Property  | Type | Possible Values | Required (default value) | Description|
| --------- | ------------------ | --------------- | --------- | ------------------ | --------------- | --------------- | 
| `url` | `string` | any | Yes | Specifies the URL where the function can be invoked. |
| `headers` | `[string:string]` | any | No | Specifies any HTTP headers for the request that invokes the function. |


## Using variables

Variables allow you to dynamically replace configuration values in your service definition file.

They are especially useful when providing _secrets_ for your service and when you have a multi-staging developer workflow.

To use variables inside `graphcool.yml`, you need to reference the values enclosed in `${}` brackets:

```yml
# graphcool.yml file
yamlKeyXYZ: ${variableSource} # see list of current variable sources below
# this is an example of providing a default value as the second parameter
otherYamlKey: ${variableSource, defaultValue}
```

A _variable source_ can be either of the following two options:

- A _recursive self-reference_ to another value inside the same service
- An _environment variable_
- The name of the currently active [environment](!alias-zoug8seen4) from [`.graphcoolrc`](!alias-zoug8seen4#.graphcoolrc)

> Note that you can only use variables in property **values** - not in property keys. So you can't use variables to generate dynamic logical IDs in the custom resources section for example.

### Recursive self-reference

You can recursively reference other property values that live inside the same `graphcool.yml` file.

When using a recursive self-reference as a variable, the value that you put into the bracket is composed of:

- the _prefix_ `self:`
- (optional) the _path_ to the referenced property

> If no path is specified, the value of the variable will be the full YAML file.

In the following example, the `createCRMEntry` function uses the same subscription query as the `sendWelcomeEmail` function:

```yml
functions:
  sendWelcomeEmail:
    handler:
      code:
        src: ./src/sendWelcomeEmail.js
    type: subscription
    query: ./src/newUserSubscription.graphql
  createCRMEntry:
    handler:
      code:
        src: ./src/createCRMEntry.js
    type: subscription
    query: ${self:functions.sendWelcomeEmail.handler.query}
```


### Environment variable

You can reference [environment variables](https://en.wikipedia.org/wiki/Environment_variable) inside the service definition file.

When using an environment variable, the value that you put into the bracket is composed of:

- the _prefix_ `env:`
- the _name_ of the environment variable

In the following example, an environment variable is referenced to specify the URL and the authentication token for a webhook:

```yml
functions:
  initiatePayment:
    handler:
      webhook:
        url: ${env:PAYMENT_URL}
        headers:
            Content-Type: application/json
            Authorization: Bearer ${env:AUTH_TOKEN}
    type: subscription
```

<InfoBox type=warning>

Note that you can not use the name `GRAPHCOOL_ENV` for your environment variables. This will automatically refer to the currently active environment (see the section below).

</InfoBox>

### Name of currently active environment

You can reference the name of the currently active [environment](!alias-zoug8seen4) inside the service definition file.

The syntax is similar to the one for referencing environment variables, except that the _name_ of the environment variable is replaced with `GRAPHCOOL_ENV`:

```yml
${env:GRAPHCOOL_ENV}
```

> Note: Unlike you might guess from the syntax, `GRAPHCOOL_ENV` is not actually set as an environment variable. 


