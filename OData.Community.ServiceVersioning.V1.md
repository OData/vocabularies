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
 
##2	Service Versioning
###2.1	Client requests service metadata 
When metadata-aware clients first retrieve metadata from a service to discover its data model (often for the purpose of generating proxy classes to simplify interaction with the service), the service can include in its response an annotation on the EntityContainer that describes the version information associated with the provided metadata.  This annotation also provides instructions to the client on how the version dependency that the client has established should be communicated to the service.

####2.1.1	ServiceVersionInfo Annotation
The service may return a “ServiceVersionInfo” annotation on the EntityContainer containing a complex type named VersionInfo with the following structure:

|Property	|Type	|Description
|--------   |----   |-----------
|CurrentVersion	|Edm.String	|The version of the service associated with the returned metadata.  Required.
|Required	|Edm.Boolean	|Indicates whether the service requires that this version be included in client requests.  If true, either VersionHeaderName or VersionQueryStringParameterName should be provided.  Optional.  Defaults to false.
|VersionHeaderName	|Edm.String	|The name of the header clients can use to communicate version information.  Optional.  Nullable.  Services should provide either a header name or a query string parameter name if clients are expected to provide version information in requests.
|VersionQueryStringParameterName	|Edm.String	|The name of the query string parameter clients can use to communicate version information.  Optional.  Nullable.  Services should provide either a header name or a query string parameter name if clients are expected to provide version information in requests.

####2.1.2	Example: Simple, Optional Service Version
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

####2.1.3	Example: Simple, Required Service Version
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

###2.2	Client calls a service with version information
After a client has received the service version information from the service, it should configure itself to transmit the appropriate version information to the service in each call.

####2.2.1	Version transmission logic
When service version information is provided at code generation time, client frameworks should default to sending that service version information with each request.

When scope version information is provided, and indicated as required, client frameworks should default to sending version information for those scopes with each request.

Developers should be provided the ability to override the default setting and control the transmission of version information for the service and for individual scopes.

If a service supports both header-based and query-based versioning, the client frameworks should default to passing version information over query strings, but allow developers to override this setting and chose the method of transmission they prefer.  Client frameworks should not transmit version information using both header and query strings.

####2.2.2	Version Information Format
When transmitting version information to the service, the string value of the requested header or query string parameter should be the exact string provided by the service in the ServiceVersionInfo annotation’s CurrentVersion property.  

####2.2.3	Header Version Format
When providing version information in a header, services must provide a Header Name that meets the http requirements for header names.  The client should include in requests a header with the requested header name with a value of the version information format above.

For example, given the version information in the previous example and a requested version header name of “api-version”:

```http
api-version: 7.2
```

When version information is allowed to be passed in headers, services must provide version string values that meet the HTTP requirements for header values.

####2.2.4	Query String Parameter Version Format
When providing version information in a query string parameter, services must provide Query String Parameter Names that meets the http requirements for query string parameter names.  The client should include in requests a query string parameters with the requested name with a value of the version format above.

When version information is allowed to be passed in query string parameters, version strings will be encoded as any other query string value.

For example, given the version information in the previous example and a requested query string parameter name of “api-version”:

```http
http://service/root/entity(123)?api-version=7.2
```

####2.2.5	Example: Client calls a service with service version information in Header
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

####2.2.6	Example: Client calls a service with service version information in Query String
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

###2.3	Service Responses
When services communicate that their Version Information is required in requests, but a request is received without it, a service is expected to respond with an HTTP error status such as 400 Bad Request with a message indicating that the version header or query string parameter is required.

When a client requests a version of the service that the service understands but cannot respond compatibly with, the service is expected to respond with HTTP error status such as 501 Not Implemented or 400 Bad Request.  Error message text should also communicate that the requested version of the service is not available.

When the client requests a version that the service is compatible with, the service is expected to return a version of results that includes only non-breaking changes from the metadata that would have been provided for the requested version of the service.

##3	Scoped Service Versioning
Complex services may contain data models that are generated from multiple sub-systems or components.  For example, a service may allow the deployment of “extensions” to a service instance that add to the available data model or functionality.  Scoped service versioning is intended to support these advanced scenarios by allowing the communication of specific version information for each of these “scopes”.

###3.1	Client requests service metadata
When metadata-aware clients first retrieve metadata from a service to discover its data model (often for the purpose of generating proxy classes to simplify interaction with the service), the service can include in its response an annotation on the EntityContainer that describes the version information for a set of “scopes” associated with the provided metadata.  This annotation also provides instructions to the client on how a version dependency that the client has established to those scopes should be communicated to the service.

####3.1.1	ScopedServiceVersionInfo Annotation
The service may return a “ScopedServiceVersionInfo” annotation on the EntityContainer containing a collection of the ScopedVersionInfo complex type (based in the above VersionInfo) which has the following additional property:

|Property	|Type	|Description
|--------   |----   |-----------
|Scope	|Edm.String	|The scope to which this version information applies.  Each ScopedServiceVersionInfo annotation should provide a unique scope value.

####3.1.2	Example: Scope Version Information
The following annotation in a $metadata response would indicate that the metadata returned represents 2 named scopes, that the service supports communicating the required service version via a query string parameter named “solution-versions”, and that returning this version information is optional:

```xml
<EntityContainer Name="DefaultContainer">
  <Annotation Term="ServiceVersioning.ScopedServiceVersionInfo">
    <Collection>
      <Record>
        <PropertyValue Property="Scope" String="isvsolution1" />
        <PropertyValue Property="CurrentVersion" String="5.0" />
        <PropertyValue Property="VersionQueryStringParameterName" String="solution-versions" />
      </Record>
      <Record>
        <PropertyValue Property="Scope" String="isvsolution2" />
        <PropertyValue Property="CurrentVersion" String="3.1" />
        <PropertyValue Property="VersionQueryStringParameterName" String="solution-versions" />
      </Record>
    </Collection>
  </Annotation>
</EntityContainer>
```

####3.1.3	Example: Service and Scope Version Information
The following annotation in a $metadata response would indicate that the metadata returned represents service version 7.2, and includes 2 named scopes, that the service supports communicating the required service version via query string parameter named “api-version”, and the optional scope version information via a query string parameter named "solution-versions":

```xml
<EntityContainer Name="DefaultContainer">

  <Annotation Term="ServiceVersioning.ServiceVersionInfo">
    <Record>
      <PropertyValue Property="CurrentVersion" String="7.2" />
      <PropertyValue Property="Required" Bool="true" />
      <PropertyValue Property="VersionQueryStringParameterName" String="api-version" />
    </Record>
  </Annotation>

  <Annotation Term="ServiceVersioning.ScopedServiceVersionInfo">
    <Collection>
      <Record>
        <PropertyValue Property="Scope" String="isvsolution1" />
        <PropertyValue Property="CurrentVersion" String="5.0" />
        <PropertyValue Property="VersionQueryStringParameterName" String="solution-versions" />
      </Record>
      <Record>
        <PropertyValue Property="Scope" String="isvsolution2" />
        <PropertyValue Property="CurrentVersion" String="3.1" />
        <PropertyValue Property="VersionQueryStringParameterName" String="solution-versions" />
      </Record>
    </Collection>
  </Annotation>
        
</EntityContainer>
```

###3.2	Client calls a service with scope version information
After a client has received the service version information from the service, it should configure itself to transmit the appropriate scope version information to the service in each call.

####3.2.1	Version transmission logic
When scope version information is provided in metadata, client frameworks should default to sending version information for those scopes with each request.

Developers should be provided the ability to override the default setting and control the transmission of version information for individual scopes so they can avoid unnecessary scope version dependencies.  Services should document how developers can map scopes to particular parts of the data model.

If a service supports both header-based and query-based versioning, the client frameworks should default to passing version information over query strings, but allow developers to override this setting and chose the method of transmission they prefer.  Client frameworks should not transmit version information using both header and query strings.

####3.2.2	Scope Version Information Format
When transmitting scope version information to the service, the string value for the requested header or query string parameter should be a comma separated list of scope version terms for each defined header or query string parameter.

Each scope version term should me made up of the scope name, a forward-slash (“/”), and then the exact version string provided for that scope in the ScopedServiceVersionInfo annotation’s CurrentVersion property.  

For example, given 2 scopes: solutionA with version 5.0 and solutionB with verion 3.0, the version information should be formatted as:

```
solutionA/5.0,solutionB/3.0
```

####3.2.3	Header Version Format
When providing version information in a header, services must provide a Header Name that meets the http requirements for header names.  The client should include in requests a header with the requested header name with a value of the version information format above.

For example, given the scopes in the previous example with a requested scope version header name of “solution-versions”, the header should be crafted as:

```
solution-versions: solutionA/5.0,solutionB/3.0
```

When version information is allowed to be passed in headers, services must provide version string values that meet the HTTP requirements for header values.

####3.2.4	Query String Parameter Version Format
When providing version information in a query string parameter, services must provide Query String Parameter Names that meets the http requirements for query string parameter names.  The client should include in requests a query string parameters with the requested name with a value of the version or scope version information format above.

When version information is allowed to be passed in query string parameters, version strings will be encoded as any other query string value.

For example, given the version information in the previous example and a requested query string parameter name of “solution-versions”:

Scope versions: 

```http
http://service/root/entity(123)?solution-versions=solutionA%2F5.0%2CsolutionB%2F3.0
```

Services must not include comma (“,”) or forward slash “/” characters in their scope version strings.

####3.2.5	Providing Service Version and Scope Version together
If a service requests that a service version and scope versions be provided using the same header or query string parameter, clients should provide them in the same comma-separated list, but with the service version listed first, having no scope, and no forward slash.  For example:

```
7.2,solutionA/5.0,solutionB/3.0
```

When provided in a common header this would take the form:

```
api-version: 7.2,solutionA/5.0,solutionB/3.0
```

When provided in a query string:

```http
http://service/root/entity(123)?api-version= 7.2%2CsolutionA%2F5.0%2CsolutionB%2F3.0
```

####3.2.6	Example: Client calls a service with scope version information only in a header
Given the following version information in metadata:

```xml
<EntityContainer Name="DefaultContainer">
  <Annotation Term="ServiceVersioning.ScopedServiceVersionInfo">
    <Collection>
      <Record>
        <PropertyValue Property="Scope" String="isvsolution1" />
        <PropertyValue Property="CurrentVersion" String="5.0" />
        <PropertyValue Property="VersionHeaderName" String="solution-versions" />
      </Record>
      <Record>
        <PropertyValue Property="Scope" String="isvsolution2" />
        <PropertyValue Property="CurrentVersion" String="3.1" />
        <PropertyValue Property=" VersionHeaderName " String="solution-versions" />
      </Record>
    </Collection>
  </Annotation>
</EntityContainer>
```

The client should add the solution-versions header as follows:

```http
GET /service/Customers HTTP/1.1
Host: odata.org
OData-Version: 4.0 
OData-MaxVersion: 4.0 
Content-Type: application/json
solution-versions: isvsolution1/5.0,isvsolution2/3.1
```

####3.2.7	Example: Client calls a service with scope version information only in a query string parameter
Given the following version information in metadata:

```xml
<EntityContainer Name="DefaultContainer">
  <Annotation Term="ServiceVersioning.ScopedServiceVersionInfo">
    <Collection>
      <Record>
        <PropertyValue Property="Scope" String="isvsolution1" />
        <PropertyValue Property="CurrentVersion" String="5.0" />
        <PropertyValue Property="VersionQueryStringParameterName" String="solution-versions" />
      </Record>
      <Record>
        <PropertyValue Property="Scope" String="isvsolution2" />
        <PropertyValue Property="CurrentVersion" String="3.1" />
        <PropertyValue Property="VersionQueryStringParameterName" String="solution-versions" />
      </Record>
    </Collection>
  </Annotation>
</EntityContainer>
```

The client should add the solution-versions query string parameter as follows:

```http
GET /service/Customers?solution-versions=isvsolution1%2F5.0%2Cisvsolution2%2F3.1 HTTP/1.1
Host: odata.org
OData-Version: 4.0 
OData-MaxVersion: 4.0 
Content-Type: application/json
```

####3.2.8	Example: Client calls a service with service and scope version information
Given the following version information in metadata:

```xml
<EntityContainer Name="DefaultContainer">

  <Annotation Term="ServiceVersioning.ServiceVersionInfo">
    <Record>
      <PropertyValue Property="CurrentVersion" String="7.2" />
      <PropertyValue Property="Required" Bool="true" />
      <PropertyValue Property="VersionQueryStringParameterName" String="api-version" />
    </Record>
  </Annotation>

  <Annotation Term="ServiceVersioning.ScopedServiceVersionInfo">
    <Collection>
      <Record>
        <PropertyValue Property="Scope" String="isvsolution1" />
        <PropertyValue Property="CurrentVersion" String="5.0" />
        <PropertyValue Property="VersionQueryStringParameterName" String="solution-versions" />
      </Record>
      <Record>
        <PropertyValue Property="Scope" String="isvsolution2" />
        <PropertyValue Property="CurrentVersion" String="3.1" />
        <PropertyValue Property="VersionQueryStringParameterName" String="solution-versions" />
      </Record>
    </Collection>
  </Annotation>
        
</EntityContainer>
```

The client should add the solution-versions query string parameter as follows:

```http
GET /service/Customers?api-version=7.2&solution-versions=isvsolution1%2F5.0%2Cisvsolution2%2F3.1 HTTP/1.1
Host: odata.org
OData-Version: 4.0 
OData-MaxVersion: 4.0 
Content-Type: application/json
```

####3.2.9	Example: Client calls a service with service and scope version in shared query string parameter
Given the following version information in metadata:

```xml
<EntityContainer Name="DefaultContainer">

  <Annotation Term="ServiceVersioning.ServiceVersionInfo">
    <Record>
      <PropertyValue Property="CurrentVersion" String="7.2" />
      <PropertyValue Property="Required" Bool="true" />
      <PropertyValue Property="VersionQueryStringParameterName" String="api-version" />
    </Record>
  </Annotation>

  <Annotation Term="ServiceVersioning.ScopedServiceVersionInfo">
    <Collection>
      <Record>
        <PropertyValue Property="Scope" String="isvsolution1" />
        <PropertyValue Property="CurrentVersion" String="5.0" />
        <PropertyValue Property="VersionQueryStringParameterName" String="api-version" />
      </Record>
      <Record>
        <PropertyValue Property="Scope" String="isvsolution2" />
        <PropertyValue Property="CurrentVersion" String="3.1" />
        <PropertyValue Property="VersionQueryStringParameterName" String="api-version" />
      </Record>
    </Collection>
  </Annotation>
        
</EntityContainer>
```

The client should add the solution-versions query string parameter as follows:

```http
GET /service/Customers?api-version=7.2%2Cisvsolution1%2F5.0%2Cisvsolution2%2F3.1 HTTP/1.1
Host: odata.org
OData-Version: 4.0 
OData-MaxVersion: 4.0 
Content-Type: application/json
```

###3.3	Service Responses
When services communicate that their Version Information is required on client calls, but a client calls without it. A service is expected to respond with HTTP error status such as `400 Bad Request` with a message indicating that the version header or query string parameter is required.

When a client requests a version of the service that the service understands but cannot respond compatibly with, the service is expected to respond with HTTP error status such as `501 Not Implemented` or `400 Bad Request`.  Error message text should also communicate that the requested version of the service is not available.

When the client requests a version that the service is compatible with, the service is expected to return a version of results that includes only non-breaking changes from the metadata that would have been provided for the requested version.
