# Consuming

## URL Structure

```
/api/{module}/{controller}/{method}
```

Where `{module}` is defined by the Component's `composer.json` file.

## Responses

> @todo - complete this section

## Authentication

### Sessions

The API will respect any session cookies which are available, i.e if the user is logged in then calls to the API will also be authenticated.

### Access Tokens

Access Tokens allow for authenticated calls to the API via request headers. Set the value of the `X-Access-Token` header to the token.

#### Generating and revoking tokens

#### In code

@todo - complete this

#### Via the API

The following endpoint is available for creating and/or revoking an access token.
