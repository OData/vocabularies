#Alternate Keys for OData Services
##A mechanism for OData services to express additional unique key definitions that may be used to address entities.

##1	Introduction
###1.1	Summary
This document details a proposal for an OData v4 vocabulary to describe a services ability to have certain entities be addressed by their standard key properties as well as additional sets of properties that make up alternate keys.

###1.2	Background
OData v4 currently includes the concept of a key for an entity type that defines the set of columns that make up the unique identifier for an entity.  This key is used in the URL that addresses a specific entity instance.  For example:

```http
https://host/service/customers(123)
https://host/service/customers(CustomerId=123)
```

A key can be composed of multiple properties.  For example 

```http
https://host/service/customers(Branch=ABC,CustomerId=123)
```

For some data models there can be multiple ways to uniquely identify an entity.  A customer might be identified by an application’s CustomerId property as a primary key as well as by an enterprise customer master id and an industry standard DUNS number.  Different data integration scenarios can be achieved more efficiently if entities can be addressed using these keys directly.  For example

```http
https://host/service/customers(customermasterid=987)
https://host/service/customers(DUNSNumber=665544332211)
https://host/service/customers(Branch=ABC,CustomerId=123)
```

This concept is missing from current OData specifications.

###1.3	Further Reading
OData v4 specifications:  http://www.odata.org/documentation/
 
##2	Usage Scenarios
###2.1	Client requests service metadata
When metadata-aware clients first retrieve metadata from a service to discover its data model, entity type metadata will be annotated to describe the availability of alternate keys that can be used to address that entity type.

####2.1.1	Alternate Keys Vocabulary
When a service supports alternate keys, it will annotate entity type metadata to describe the structure of the available alternate keys for each entity type.

The AlternateKeys annotation term will contain a collection of key definitions, each of which contain a collection of PropertyRef’s.  It will be declared as follows:

```xml
<Term AppliesTo="EntityType EntitySet NavigationProperty" Type="Collection(Core.AlternateKey)" Name="Core.AlternateKeys">
  <Annotation Term="Core.Description" String="Communicates available alternate keys"/>
</Term>

<ComplexType Name="AlternateKey">
  <Property Type="Collection(Core.PropertyRef)" Name="Key">
    <Annotation Term="Core.Description" String="the set of properties that make up this key"/>
  </Property>
</ComplexType>

<ComplexType Name="PropertyRef">
  <Property Type="Edm.PropertyPath" Name="Name">
    <Annotation Term="Core.Description" String="A path expression resolving to a primitive property of the entity type itself or to a primitive property of a complex property (recursively) of the entity type. The names of the properties in the path are joined together by forward slashes."/>
  </Property>
  <Property Type="Edm.String" Name="Alias">
    <Annotation Term="Core.Description" String="A SimpleIdentifier that MUST be unique within the set of aliases, structural and navigation properties of the containing entity type that MUST be used in the key predicate of URLs"/>
  </Property>
</ComplexType>
```

The `AlternateKey` and `PropertRef` complex types (and their properties) are intended to directly mirror the `edm:Key` and `edm:PropertyRef` elements in the EntityType metadata with the one exception being that alternate keys may be based on properties that are nullable.

They are redefined here as complex types so that they can be used on an annotation term.  

The following examples demonstrate how this annotation term would be used to describe different alternate key configurations.
 
####2.1.2	Example: Single alternate key with single property
The following metadata described a Person entity type with an alternate key on the SSN string property:

```xml
<EntityType Name="Person">
  <Key>
    <PropertyRef Name="ID" />
  </Key>
        
  <Annotation Term="Core.AlternateKeys">
    <Collection>    
      <Record>
        <PropertyValue Property="Key">
          <Collection>
            <Record>
              <PropertyValue Property="Name" PropertyPath="SSN" />
            </Record>
          </Collection>
        </PropertyValue>
      </Record>
    </Collection>
  </Annotation>
        
  <Property Name="ID" Type="Edm.Int64" Nullable="false" />
  <Property Name="SSN" Type="Edm.String" Nullable="true" />
</EntityType>
```
 
####2.1.3	 Example: Single Alternate Key composed of multiple properties
The following metadata described a Person entity type with an alternate key defined on the person’s home country and passport number string properties:

```xml
<EntityType Name="Person">
  <Key>
    <PropertyRef Name="ID" />
  </Key>
        
  <Annotation Term="Core.AlternateKeys">
    <Collection>
      <Record>
        <PropertyValue Property="Key">
          <Collection>
            <Record>
              <PropertyValue Property="Name" PropertyPath="Country" />
            </Record>
            <Record>
              <PropertyValue Property="Name" PropertyPath="Passport" />
            </Record>
          </Collection>                
        </PropertyValue>
      </Record>
    </Collection>
  </Annotation>
        
  <Property Name="ID" Type="Edm.Int64" Nullable="false" />
  <Property Name="Country" Type="Edm.String" Nullable="true" />
  <Property Name="Passport" Type="Edm.String" Nullable="true" />
</EntityType> 
```

####2.1.4	Example: Single Alternate Key composed of properties from a complex type
The following metadata described a Person entity type with an alternate key defined on the person’s home country and passport number string properties which are part of a complex type.  Take note of the required use of the Alias property.

```xml
<EntityType Name="Person">
  <Key>
    <PropertyRef Name="ID" />
  </Key>
        
  <Annotation Term="Core.AlternateKeys">
    <Collection>
      <Record>
        <PropertyValue Property="Key">
          <Collection>
            <Record>
              <PropertyValue Property="Name" PropertyPath="ContactInfo/Country" />
              <PropertyValue Property="Alias" String="Country" />
            </Record>
            <Record>
              <PropertyValue Property="Name" PropertyPath="ContactInfo/Passport" />
              <PropertyValue Property="Alias" String="Passport" />
            </Record>
          </Collection>                
        </PropertyValue>
      </Record>
            
    </Collection>
  </Annotation>
        
  <Property Name="ID" Type="Edm.Int64" Nullable="false" />
  <Property Name="SSN" Type="Edm.String" Nullable="true" />
  <Property Name="ContactInfo" Type="My.ContactInfo" />
</EntityType>
```

####2.1.5	Example: Multiple Alternate Keys
The following metadata described a Person entity type with two alternate key defined.  One key uses the person’s SSN, and the other users their Employee ID.

```xml
<EntityType Name="Person">
  <Key>
    <PropertyRef Name="ID" />
  </Key>
        
  <Annotation Term="Core.AlternateKeys">
    <Collection>    
      <Record>
        <PropertyValue Property="Key">
          <Collection>
            <Record>
              <PropertyValue Property="Name" PropertyPath="SSN" />
            </Record>
          </Collection>
        </PropertyValue>
      </Record>
      <Record>
        <PropertyValue Property="Key">
          <Collection>
            <Record>
              <PropertyValue Property="Name" PropertyPath="EmployeeID" />
            </Record>
          </Collection>
        </PropertyValue>
      </Record>
    </Collection>
  </Annotation>
        
  <Property Name="ID" Type="Edm.Int64" Nullable="false" />
  <Property Name="SSN" Type="Edm.String" Nullable="true" />
  <Property Name="EmployeeID" Type="Edm.String" Nullable="true" />
</EntityType>
```

####2.1.6	Key Declaration Uniqueness
An alternate key is identified by the unique set of properties that are referenced in its declaration, regardless of the order in which properties are listed.   

Services SHOULD NOT return multiple alternate key definitions for the same entity type that are composed of the exact same set of properties.

Services MAY define keys that contain overlapping properties or that contain all the properties of another key plus additional properties.

Services SHOULD NOT provide an alternate key definition that exactly matches the property set of the primary entity key.
 
###2.2	Client calls a service with alternate keys
When a client wishes to address an operation to an entity using alternate key values, they will provide the property name(s) and values for the intended key in the key segment.

####2.2.1	Key Predicate Formats and Key Matching
OData defines 2 types of key predicates, a simpleKey and a compoundKey as follows:

```http
http://host/service/persons(123) (simpleKey)
http://host/service/persons(ID=123) (compoundKey with 1 key value)
http://host/service/persons(Country=USA,Passport=9867) (compoundKey with 2 key values)
```

Any request using the simpleKey format should always be interpreted as expressing a value for the primary key.

When a request uses the compoundKey format, services should identify a matching key definition (either the primary entity key, or an alternate key) that uses the exact set of named properties provided.  

Using the key declaration example above, the following examples would allow retrieval of people matched with the defined keys:

```http
http://host/service/persons(id=123) (matches entity key exactly)
http://host/service/persons(SSN=123) (matches alternate key exactly)
http://host/service/persons(Country=USA,Passport=9876) (matches alternate key exactly)
```

If the set of properties does not match a defined key or alternate key, the service should return HTTP Status 400 (Bad Request).

####2.2.2	Use of Alternate Keys
Alternate key syntax should be supported in all the places that the canonical URL for an entity is used, including in in URL’s for data service requests as well as alternative entity-id’s for bind operations.

#####Address and Entity	

```http
GET /service/persons(ssn=123-456-7890)  HTTP/1.1
PATCH /service/persons(ssn=123-456-7890) HTTP/1.1
```

#####Address Contained Entity

```http
GET /service/road(90)/exit(exitNumber=20B)  HTTP/1.1
PATCH /service/road(90)/exit(exitNumber=20B)  HTTP/1.1
```

#####Bind an entity	
```json
{ 
  "@odata.type":"#Northwind.Manager",
  "EmployeeID": 1,
  "DirectReports@odata.bind": [
    "http://host/service/Employees(ssn=123-45-6789)",
    "http://host/service/Employees(ssn=111-22-3333)"
  ]
}
```
#####Addressing an Entity Reference	

```http
DELETE /service/Categories(catCode=11)/Products/$ref?$id=../../Products(sku=abc123)
DELETE /service/Products(sku=abc123)/Category/$ref
```

####2.2.3	Key Value Uniqueness
Alternate Keys are intended to allow callers to refer to a specific entity instance.  Therefore, services must allow 1 and only 1 record in an entity collection to match any set of values for the set of properties defined for each key.

Services may allow null values in properties that are used for alternate keys, but clients will not be able to retrieve those entities using a key predicate that specifies the null value.  A client request that includes a null value for a property in the key predicate MUST return HTTP Status 404 (not found).

For example, both of these requests should always return HTTP Status 404 (not found):

http://host/service/persons(ID=null)
http://host/service/persons(Country=USA,Passport=null) 

####2.2.4	Entity Urls in Service Responses
Support for alternate keys does not imply or require the modification of canonical urls returned by the service that represent links to entities (for example, odata.id, ReadUrl, etc.).  Services should continue to return stable references to resources that do not change relative to the access mechanism used by the client to retrieve those references.

For Example:

```http
GET /service/Customers('ALFKI')  HTTP/1.1

{
  "@odata.context": "http://host/service/$metadata#Customers/$entity",
  "@odata.id": "Customers('ALFKI')",
  "ID": "ALFKI",
  "CompanyName": "Alfreds Futterkiste",
  "Fax": "030-0076545",
}

GET /service/Customers(DUNS=987654)  HTTP/1.1

{
  "@odata.context": "http://host/service/$metadata#Customers/$entity",
  "@odata.id": "Customers('ALFKI')",
  "ID": "ALFKI",
  "CompanyName": "Alfreds Futterkiste",
  "Fax": "030-0076545",
}
```

