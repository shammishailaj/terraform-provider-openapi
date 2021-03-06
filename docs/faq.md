# FAQ

This document aims to provide more insight about the general functionality of this terraform provider.
 
## <a name="howToIntegrate">I am a service provider, does my API need to follow any specification to be able to integrate with this tool?</a> 

Short answer, yes. 

This terraform provider relies on the service provider to follow certain API standards in terms of how the end points 
should be defined. These end points should be defined in a [swagger file](https://swagger.io/specification/) 
that complies with the OpenAPI Specification (OAS) and contains the definition of all the resources supported by the service. 

Please note that swagger is currently a bit behind the latest version of [OpenAPI 3.0](https://swagger.io/specification/#securitySchemeObject). 
Hence, for more information about currently supported features refer to 
[Swagger RESTful API Documentation Specification](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md) 

Additionally, to achieve some consistency across multiple service providers in the way the APIs are structured, it is expected 
the APIs to follow [Google APIs Design guidelines](https://cloud.google.com/apis/design/).

## <a name="versioning">I am service provider and need to upgrade my APIs...How will this provider handle new versions?</a>

The version topic among software engineers is rather conflicting and often involves endless discussions that most of 
the times finish with a non deterministic conclusion. Not having an official guideline that expresses the best-practise 
approach to follow, more specifically **path versioning VS content-type negotiation**, makes it even more difficult to
stick to one or the other. Therefore, you need to make a personal call and in this case the decision has leaned towards
path versioning.

Why?

- Makes the endpoints immutable which means that new versions require a complete new namespace. This helps, as far as the 
API terraform provider is concerned, to handle different versions on the same resource and makes it more explicit from
the users point of view to decide what version to use on the terraform resource type level (see the example below).
- Swagger files are easier to configure using path versioning than content-type versioning
- Each version is associated with its own backend function rather than clugging support for different content types within
the same resource function.
 
This provider expects the service providers to follow [Google APIs Design guidelines](https://cloud.google.com/apis/design/)
so refer to the guideline for any questions related to 'how' the APIs should be structured.

This terraform provider is able to read the resources exposed and the versions they belong too. So, if a service
provider is exposing the following end point with two different versions their corresponding tf configuration would look
like:

- Let's say the service provider initially had support for version 1 and the sagger file looked as follows:

```
paths:
  /v1/cdns:
    post:
      tags:
      - "cdn"
      summary: "Create cdn"
      operationId: "ContentDeliveryNetworkCreateV1"
      parameters:
      - in: "body"
        name: "body"
        description: "Created CDN"
        required: true
        schema:
          $ref: "#/definitions/ContentDeliveryNetworkV1"
    
definitions:
  ContentDeliveryNetworkV1:
    type: "object"
    required:
      - label
      - ips
      - hostnames
    properties:
      id:
        type: "string"
        readOnly: true
      label:
        type: "string"
      ips:
        type: "array"
        items:
          type: "string"
      hostnames:
        type: "array"
        items:
          type: "string"
```

The corresponding .tf resource definition would look like:

```
resource "sp_cdns_v1" "my_cdn_v1" {
  label = "label"
  ips = ["127.0.0.1"]
  hostnames = ["origin.com"]
}
```

- After a while, new functionality needs to be supported and the API has to change dramatically resulting into the new API
non being backwards compatible. Solution? Create a new version namespace with its own model object. For the sake of the 
example and to keep it small, V1 version is not shown below.

```
paths:
  .... (/v1/cdns path)
  
  /v2/cdns:
    post:
      tags:
      - "cdn"
      summary: "Create cdn"
      operationId: "ContentDeliveryNetworkCreateV2"
      parameters:
      - in: "body"
        name: "body"
        description: "Created CDN"
        required: true
        schema:
          $ref: "#/definitions/ContentDeliveryNetworkV2"
    
  ....
    
definitions:

  ... (ContentDeliveryNetworkV1 definition)

  ContentDeliveryNetworkV2:
    type: "object"
    required:
      - proxyDns
    properties:
      id:
        type: "string"
        readOnly: true
      proxyDns:
        type: "string"
        
  ...
  
```

And the corresponding .tf resource definition would look like:

```
resource "sp_cdns_v2" "my_cdn_v2"{
    proxy_dns = ""
}    
```

## <a name="multipleEnvironments">I am service provider and currently support multiple environments. How will this provider handle that?</a>

To be decided...


