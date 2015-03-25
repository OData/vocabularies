
# Displayable Value Annotation for OData Services## <small>A mechanism for OData services to provide representations of property values as they are intended to be displayed to the calling user.</small>

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

The client will communicate its desire for the service to provide displayable values by using the `odata.include-annotations` with the value “Display.DisplayableValue”, for example:

```http
Prefer: odata.include-annotations="Display.DisplayableValue"
```

#### 2.2	Service Responses

When a service understands the request for Displayable Values annotations and can fulfill it, it will respond with a `Preference-Applied` header that includes the annotation name.  The service will include instance annotations in responses. These instance annotations will contain the displayable values.

#### 2.2.1	Instance Annotation on Property Values

```json
{
  "@odata.context": "http://host/service/$metadata#Customers/$entity",
  "ID": "ALFKI",
  "CompanyName": "Alfreds Futterkiste",
  "CreditLimit@Display.DisplayableValue": "$10,000",
  "CreditLimit": 10000.00,
  "Birthday@Display.DisplayableValue": "9/18/1969",
  "Birthday": "1969-09-18"
}
```

#### 2.2.2	 Instance Annotation on Entity References

```json
{
  "@odata.context": "http://host/service/$metadata#$ref",
  "@Display.DisplayableValue": "Fabrikam, Inc.",
  "@odata.id": "Customers(10643)"
}
```

```json
{
  "@odata.context": "http://host/service/$metadata#Collection($ref)",
  "value": [
    {
      "@Display.DisplayableValue": "Fabrikam, Inc.",
      "@odata.id": "Orders(10643)"
    },
    {
      "@Display.DisplayableValue": "Contoso Corp.",
      "@odata.id": "Orders(10759)"
    }
  ]
}
```

#### 2.2.3	Instance Annotation on Entities

```json
{
  "@odata.context": "http://host/service/$metadata#Customers/$entity",
  "@Display.DisplayableValue": "Alfreds Futterkiste",
  "ID": "ALFKI",
  "CompanyName": "Alfreds Futterkiste",
  "ContactName": "Maria Anders",
  "ContactTitle": "Sales Representative",
  "Phone": "030-0074321",
  "Fax": "030-0076545"
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
  "LoyaltyLevel @Display.DisplayableValue": "Gold Medallion",
  "LoyaltyLevel": "gld"
}
```
