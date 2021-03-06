# apimtpl

Azure API Management template generator

## Usage

You can run `apimtpl` via Docker:

```sh
docker run --rm -it azrtools/apimtpl --help
```

If you have an input file `/home/user/api.yaml`, you can generate ARM templates
using the following command:

```sh
docker run --rm -it -v "/home/user/api.yaml:/api.yaml" azrtools/apimtpl \
    /api.yaml >/home/user/api/output.json
```

The output will be written to `/home/user/api/output.json`

## Configuration

The most basic input file looks like this:

```yaml
configuration:
  serviceName: apim-service
environments:
  - name: development
apis:
  - name: example-api
    path: example-api
    operations:
      - name: ping
        method: POST
        path: /ping
```

To also generate products and subscriptions, additionally use this input:

```yaml
products:
  - name: example
    apis:
      - example-api
subscriptions:
  - name: example
    scope:
      product: example
```

You can put the input into one file or split it into multiple files, for
example to have one API definition per file.
It's also possible to put multiple YAML documents (separated by `---`) into one
file.

### Properties

Properties (called _Named values_ in the Azure portal) can be used to store
credentials like usernames or passwords.

You can reference properties in the policy using the `$[property-name]` syntax.

```yaml
environments:
  - name: development
apis:
  - name: example-api
    path: example-api
    policies:
      inbound: |
        <base />
        <set-backend-service base-url="https://example.com" />
        <authentication-basic username="$[username]" password="$[password]" />
    properties:
      - name: username
        value: user001
      - name: password
    operations:
      - name: get-data
        method: GET
        path: /data
```

If the `value` is omitted, the property can be referenced as usual in the
configuration files, but the property itself will not be generated.

### Environment-specific configuration

The top-level `apis` key specifies API configuration that applies to all
environments. To set some configuration values only for specific environments,
you can use the `apis` key below the desired environment:

```yaml
environments:
  - name: development
apis:
  - name: example-api
    path: example-api
    policies:
      inbound: |
        <base />
        <set-backend-service base-url="https://example.com" />
    operations:
      - name: get-data
        method: GET
        path: /data
---
environments:
  - name: development
    apis:
      - name: example-api
        policies:
          inbound: |
            <base />
            <set-backend-service base-url="https://development.example.com" />
```

### Parameters

To reference parameters in your configuration, use the `parameter` key.
Parameter values can be set using the top-level `parameters` key.

```yaml
parameters:
  - name: usernameParameter
    value: user001
  - name: passwordParameter
    value: password123
---
configuration:
  serviceName: apim1
environments:
  - name: development
apis:
  - name: example-api
    path: ${name}
    properties:
      - name: username
        value:
          parameter: usernameParameter
      - name: password
        value:
          parameter: passwordParameter
        secret: true
    operations:
      - name: get-data
        method: GET
        path: /data
```

### Variables

Variables can be used for dynamic values, they will be replaced during
template generation.

The variables `name`, `environment.name` and `api.name` are built-in variables.
They will be resolved to the name of the current object, the environment name
and the current API name, respectively.

```yaml
configuration:
  serviceName: apim1
environments:
  - name: development
    variables:
      - name: backend-server
        value: dev.example.com
  - name: staging
    variables:
      - name: backend-server
        value: staging.example.com
apis:
  - name: example-api
    path: ${name}
    policies:
      inbound: |
        <base />
        <set-backend-service base-url="https://${backend-server}" />
    operations:
      - name: get-data
        method: GET
        path: /data
```

### Name validation

Use the `validation` section to enforce consistent naming patterns.

```yaml
validation:
  api:
    name: "example-.+"
    displayName: "Example-.+"
    path: "[a-z-]+"
  environment:
    name: "(development|quality)"
    displayName: "[A-Z].+"
  operation:
    displayName: "[A-Z][A-Za-z-]+"
  product:
    displayName: "[A-Z].+"
  subscription:
    displayName: "[A-Z].+"
configuration:
  serviceName: apim-service
environments:
  - name: development
apis:
  - name: example-api
    path: example-api
    operations:
      - name: ping
        method: POST
        path: /ping
```

## Related projects

- https://github.com/Azure/azure-api-management-devops-resource-kit
- https://github.com/mirsaeedi/dotnet-apim

## License

[MIT](LICENSE)
