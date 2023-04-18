# Introduction
The [EidoType](~/api/Eido.EidoType.yml) class is the main class for an EIDO JSON document. The EIDO JSON schema named top level JSON document "EmergencyIncidentDataObjectType". This name change was for convience only and has no impact on interoperability.
# Parsing EIDO JSON Documents
## Deserializing EIDO JSON Documents
The EidoHelper class has a method called FromString() that can be used to deserialize a string into an EidoType object. For example:
```
EidoType eido = EidoHelper.FromString(strEido);
```
This function returns null if an exception occurs so be sure to check that the returned EidoType object is not null.

It is important to note that the JSON schema for the EIDO uses property names that are actually reserved JSON keywords for metadata. For example, the ReferenceType has a property called "$ref".

If you try to deserialize an EIDO JSON document using the JsonConvert.DeserializeObject() method, an InvalidOperationException will be thrown unless you pass it a JsonSerializerSettings object with the MetadataPropertyHandling property set to MetadataPropertyHandling.Ignore. The EidoHelper.FromString() method handles this for you.

## Serializating EidoType Objects
The EidoHelper class has a method called ToJsonString() that serizlizes an EidoType object to a JSON string. For example:
```
EidoType eido = new EidoType();
// TODO: Initialize the properties of the eido object
string strEidoJson = EidoHelper.ToJsonString(eido);
```
The ToJsonString() method returns null if an exception occurs so be sure to test the return value for null.

# EIDO Timestamps
All date-time objects in the EIDO must be formatted according to Section 2.3 of NENA-STA-010.3b. For example, “2022-08-21T12:58:03.01-07:00”.

The EidoHelper class has the following two methods to handle this for you.
```
public static string GetNenaTimestamp();
public static string GetNenaTimestamp(DateTime Tm);
```

The first method uses the current local time returned by DateTime.Now. The second method takes a DateTime structure containing the date and time to format into a string.

These methods should be used to ensure consistent date/time string formatting in an EIDO.
