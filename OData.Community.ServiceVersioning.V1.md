#Service Versioning for OData Services 
##A mechanism for OData services with dynamic or evolving data models to communicate the current version of their data models to clients, and for clients to communicate data model version requirements to the service.
##1	Introduction

###1.1	Summary
This document details a proposal for an OData v4 vocabulary and a convention for header and query string usage to support communication of service version capabilities and requirements between services and their clients.

###1.2	Background
The current protocol spec for OData v4 discusses data model versioning and recommends that: 

>“Breaking changes, such as removing properties or changing the type of existing properties, require that a new service version is provided at a different service root URL for the new model.”

Some services, however, prefer to maintain stable addresses to their resources or prefer to maintain a stable service root address.   In addition, other services may be extensible and therefore expose what can be considered “dynamic” data models that depend on the current state of 3rd party or customer extensions currently hosted in the service instance.

For these services, various strategies are used today to allow the data model at the service root address to evolve over time in ways that would be considered breaking changes.  One common pattern is to have clients provide a “version” as a string value in a header or in a query string parameter.

This document attempts to formalize that pattern in a way that clients can discover, understand, and implement successfully.

###1.3	Further Reading
OData v4 specifications:  http://www.odata.org/documentation/
 
##2	Usage Scenarios
###2.1	Service Versioning
####2.1.1	Client requests service metadata 
When metadata-aware clients first retrieve metadata from a service to discover its data model (often for the purpose of generating proxy classes to simplify interaction with the service), the service can include in its response an annotation on the EntityContainer that describes the version information associated with the provided metadata.  This annotation also provides instructions to the client on how the version dependency that the client has established should be communicated to the service.

#####2.1.1.1	ServiceVersionInfo Annotation
The service may return a “ServiceVersionInfo” annotation on the EntityContainer containing a complex type named VersionInfo with the following structure:

|Property	|Type	|Description
|--------   |----   |-----------
|CurrentVersion	|Edm.String	|The version of the service associated with the returned metadata.  Required.
|Required	|Edm.Boolean	|Indicates whether the service requires that this version be included in client requests.  If true, either VersionHeaderName or VersionQueryStringParameterName should be provided.  Optional.  Defaults to false.
|VersionHeaderName	|Edm.String	|The name of the header clients can use to communicate version information.  Optional.  Nullable.  Services should provide either a header name or a query string parameter name if clients are expected to provide version information in requests.
|VersionQueryStringParameterName	|Edm.String	|The name of the query string parameter clients can use to communicate version information.  Optional.  Nullable.  Services should provide either a header name or a query string parameter name if clients are expected to provide version information in requests.

####2.1.1.2	Example: Simple, Optional Service Version
The following annotation in a $metadata response would indicate that the metadata returned represents service version 7.2, that the service supports communicating the required service version via a query string parameter named “api-version”, and that this parameter is optional:

```xml
<EntityContainer Name="DefaultContainer">
  <Annotation Term="Org.OData.ServiceVersioning.V1.ServiceVersionInfo">
    <Record>
      <PropertyValue Property="CurrentVersion" String="7.2" />
      <PropertyValue Property="VersionQueryStringParameterName" String="api-version" />
    </Record>
  </Annotation>
</EntityContainer>
```

####2.1.1.3	Example: Simple, Required Service Version
The following annotation in a $metadata response would indicate that the metadata returned represents service version 7.2, that the service supports communicating the required service version via a header named “api-version”, and that this header is required:

```xml
<EntityContainer Name="DefaultContainer">
  <Annotation Term="Org.OData.ServiceVersioning.V1.ServiceVersionInfo">
    <Record>
      <PropertyValue Property="Version" String="7.2" />
      <PropertyValue Property="Required" Bool="true" />
      <PropertyValue Property="VersionHeaderName" String="Accept-Version" />
    </Record>
  </Annotation>
</EntityContainer>
```

###2.1.2	Client calls a service with version information
After a client has received the service version information from the service, it should configure itself to transmit the appropriate version information to the service in each call.

####2.1.2.1	Version transmission logic
When service version information is provided at code generation time, client frameworks should default to sending that service version information with each request.

When scope version information is provided, and indicated as required, client frameworks should default to sending version information for those scopes with each request.

Developers should be provided the ability to override the default setting and control the transmission of version information for the service and for individual scopes.

If a service supports both header-based and query-based versioning, the client frameworks should default to passing version information over query strings, but allow developers to override this setting and chose the method of transmission they prefer.  Client frameworks should not transmit version information using both header and query strings.

####2.1.2.2	Version Information Format
When transmitting version information to the service, the string value of the requested header or query string parameter should be the exact string provided by the service in the ServiceVersionInfo annotation’s CurrentVersion property.  

####2.1.2.3	Header Version Format
When providing version information in a header, services must provide a Header Name that meets the http requirements for header names.  The client should include in requests a header with the requested header name with a value of the version information format above.

For example, given the version information in the previous example and a requested version header name of “api-version”:

```http
api-version: 7.2
```

When version information is allowed to be passed in headers, services must provide version string values that meet the HTTP requirements for header values.

####2.1.2.4	Query String Parameter Version Format
When providing version information in a query string parameter, services must provide Query String Parameter Names that meets the http requirements for query string parameter names.  The client should include in requests a query string parameters with the requested name with a value of the version format above.

When version information is allowed to be passed in query string parameters, version strings will be encoded as any other query string value.

For example, given the version information in the previous example and a requested query string parameter name of “api-version”:

```http
http://service/root/entity(123)?api-version=7.2
```

####2.1.2.5	Example: Client calls a service with service version information in Header
Given the following service version annotations in metadata:

```xml
<EntityContainer Name="DefaultContainer">
  <Annotation Term="Org.OData.ServiceVersioning.V1.ServiceVersionInfo">
    <Record>
      <PropertyValue Property="Version" String="7.2" />
      <PropertyValue Property="Required" Bool="true" />
      <PropertyValue Property="VersionHeaderName" String="api-version" />
    </Record>
  </Annotation>
</EntityContainer>
```

The client should add the api-version header as follows:

```http
GET /service/Customers HTTP/1.1
Host: odata.org
OData-Version: 4.0 
OData-MaxVersion: 4.0 
Content-Type: application/json
api-version: 7.2
```

####2.1.2.6	Example: Client calls a service with service version information in Query String
Given the following service version annotations in metadata:

```xml
<EntityContainer Name="DefaultContainer">
  <Annotation Term="Org.OData.ServiceVersioning.V1.ServiceVersionInfo">
    <Record>
      <PropertyValue Property="CurrentVersion" String="7.2" />
      <PropertyValue Property="VersionQueryStringParameterName" String="api-version" />
    </Record>
  </Annotation>
</EntityContainer>
```

The client should add the api-version query string parameter as follows:

```http
GET /service/Customers?api-version=7.2  HTTP/1.1
Host: odata.org
OData-Version: 4.0 
OData-MaxVersion: 4.0 
Content-Type: application/json
```

###2.1.3	Service Responses
When services communicate that their Version Information is required in requests, but a request is received without it, a service is expected to respond with an HTTP error status such as 400 Bad Request with a message indicating that the version header or query string parameter is required.

When a client requests a version of the service that the service understands but cannot respond compatibly with, the service is expected to respond with HTTP error status such as 501 Not Implemented or 400 Bad Request.  Error message text should also communicate that the requested version of the service is not available.

When the client requests a version that the service is compatible with, the service is expected to return a version of results that includes only non-breaking changes from the metadata that would have been provided for the requested version of the service.

