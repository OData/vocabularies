
# Formatted Value Annotation for OData Services## <small>A mechanism for OData services to provide representations of property values as they are intended to be displayed to the calling user.</small>

## 1. Introduction

### 1.1	Summary

This document details a proposal for an OData v4 annotation term that allows for the transmission of formatted strings representing the properties as the service would present them to the calling user, including localizing data such as numbers, time, and currency values for display purposes as well as providing text intended to represent a link to another entity.

### 1.2	Background

OData services are designed to transmit data with full data-type fidelity between systems.  Often, one of these systems is a client application which will display that data to the user.

Rendering data for display to a user often requires more context than is present in the data including a user’s preferences for currency, culture, location, etc. as well as specific application context about the intended use of the data by the user.  Services are typically in the best position to do this work.

### 1.3	Further reading

OData v4 specifications:  http://www.odata.org/documentation/.

## 2	Usage scenarios

### 2.1	Client requests

When clients wish to display data to users in the format provided by the service they will signal this preference to the service.

#### 2.1.1	Requesting display annotations

The client will communicate its desire for the service to provide displayable values by using the `odata.include-annotations` with the value “Display.FormattedValue”, for example:

```http
Prefer: odata.include-annotations="Display.FormattedValue"
```

### 2.2	Service Responses

When a service understands the request for Formatted Values annotations and can fulfill it, it will respond with a `Preference-Applied` header that includes the annotation name.  

The service will include  annotations in responses. These annotations will contain the displayable values (in data responses), or Path values (in metadata responses) referring to entity properties that will contain the associated formatted value.

#### 2.2.1	Metadata Annotation on Properties

```xml
<Property Name="Birthday" Type="Edm.DateTimeOffset">
    <Annotation Term="Display.FormattedValue" Path="BirthdayFormatted" />
</Property>

```

#### 2.2.2	Metadata Annotation on Entities

```xml
<EntityType Name="Account">
    <Key>
        <PropertyRef Name="AccountId" />
    </Key>
    <Property Name="AccountName" Type="Edm.String">
    <Property Name="AccountNumber" Type="Edm.String">
    <Annotation Term="Display.FormattedValue" Path="AccountName" />
</EntityType>

```

#### 2.2.3	Instance Annotation on Property Values

```json
{
  "@odata.context": "http://host/service/$metadata#Customers/$entity",
  "ID": "ALFKI",
  "CompanyName": "Alfreds Futterkiste",
  "CreditLimit@Display.FormattedValue": "$10,000",
  "CreditLimit": 10000.00,
  "Birthday@Display.FormattedValue": "9/18/1969",
  "Birthday": "1969-09-18"
}
```

#### 2.2.4	Instance Annotation on Enum Values

```json
{
  "@odata.context": "http://host/service/$metadata#Customers/$entity",
  "ID": "ALFKI",
  "CompanyName": "Alfreds Futterkiste",
  "ContactName": "Maria Anders",
  "ContactTitle": "Sales Representative",
  "Phone": "030-0074321",
  "Fax": "030-0076545",
  "LoyaltyLevel @Display.FormattedValue": "Gold Medallion",
  "LoyaltyLevel": "gld"
}
```

#### 2.2.5	 Instance Annotation on Entity References

```json
{
  "@odata.context": "http://host/service/$metadata#$ref",
  "@Display.FormattedValue": "Fabrikam, Inc.",
  "@odata.id": "Customers(10643)"
}
```

```json
{
  "@odata.context": "http://host/service/$metadata#Collection($ref)",
  "value": [
    {
      "@Display.FormattedValue": "Fabrikam, Inc.",
      "@odata.id": "Orders(10643)"
    },
    {
      "@Display.FormattedValue": "Contoso Corp.",
      "@odata.id": "Orders(10759)"
    }
  ]
}
```

#### 2.2.6	Instance Annotation on Entities

```json
{
  "@odata.context": "http://host/service/$metadata#Customers/$entity",
  "@Display.FormattedValue": "Alfreds Futterkiste",
  "ID": "ALFKI",
  "CompanyName": "Alfreds Futterkiste",
  "ContactName": "Maria Anders",
  "ContactTitle": "Sales Representative",
  "Phone": "030-0074321",
  "Fax": "030-0076545"
}
```

