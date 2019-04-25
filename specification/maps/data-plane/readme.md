# Azure Maps
    
> see https://aka.ms/autorest

Configuration for generating Azure Maps Proxy API SDK.

The current release is `package-1.0`.

``` yaml
title: MapsClient
openapi-type: data-plane
output-folder: ./autorest/src
```

## Command Line Tags

### Tag: package-1.0
These settings apply only when `--tag=package-1.0` is specified on the command line.

``` yaml $(tag) == 'package-1.0'
input-file: 
  - ./search/preview/v1/search.json
  - ./route/preview/v1/route.json
  - ./geolocation/preview/v1/geolocation.json
  - ./render/preview/v1/render.json
  - ./timezone/preview/v1/timezone.json
  - ./traffic/preview/v1/traffic.json
```

### Tag: ts
These settings apply only when `--tag=ts` is specified on the command line

``` yaml $(tag) == 'ts'
input-file: 
  - ./search/preview/v1/search.json
  - ./route/preview/v1/route.json

typescript: true

directive:
    - from: swagger-document
      where: $..*[?(@.name === "format" && @.enum.includes('json'))]
      transform: $.enum = ['json'];
      reason: >
        Autorest historically only supported json responses. For the SDK, the over-the-wire format
        of the data isn't important.
    - from: swagger-document
      where: $.paths..responses
      transform: >
        for (var responseCode of Object.keys($)) {
          var responseCodeInt = parseInt(responseCode);
          if (responseCodeInt >= 300 || responseCodeInt < 200) { delete $[responseCode] };     
        }
        return $;
      reason: Autorest treats all responses defined for an operation as successful.
    - from: swagger-document
      where: $..*[?(@.default !== undefined && !@.required)]
      transform: $.default = undefined;
      reason: Our APIs have mutually exclusive parameters. Removing all non-required defaults to prevent 4XX failures.
    - from: swagger-document
      where: $.paths.*[?(@.operationId.includes("Preview"))]
      transform: $ = undefined;
      reason: Not including Preview APIs in SDK
    - from: swagger-document
      where: $.paths.*[?(@.x-ms-long-running-operation)]
      transform: $ = undefined;
      reason: Not including Long Running APIs in SDK
    - from: swagger-document
      where: $.securityDefinitions
      transform: $ = undefined;
      reason: The typescript sdk has built-in security handling. Security definitions can be removed.
    - from: swagger-document
      where: $.security
      transform: $ = undefined;
      reason: The typescript sdk has built-in security handling. Security can be removed.
    - from: swagger-document
      where: $.paths..parameters
      transform: >
        for (var i = 0; i < $.length; i++) {
          if ($[i].$ref && $doc.parameters) {
            var path = $[i].$ref.split("/");
            var param = path[path.length - 1];
            if ($doc.parameters[param].name.toLowerCase() === "subscription-key") {
              $.splice(i--, 1);
            }
          }
        }
      reason: The typescript sdk has built-in security handling. Subscription key can be removed from parameters.
    - from: swagger-document
      where: $.parameters[?(@.name.toLowerCase() === "subscription-key")]
      transform: $ = undefined;
      reason: The typescript sdk has built-in security handling. Subscription key can be removed from parameters.

```

## CSharp Settings
These settings apply only when `--csharp` is specified on the command line.
``` yaml $(csharp) 
csharp: 
  license-header: MICROSOFT_MIT_NO_VERSION
  azure-arm: false
  namespace: Microsoft.Azure.Maps
  output-folder: $(csharp-sdks-folder)/Maps/Generated
  clear-output-folder: true
```

## Validation Steps
``` yaml $(validation)
azure-validator: true
semantic-validator: true
model-validator: true
```