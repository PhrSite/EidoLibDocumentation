# Introduction
The top level object in the EIDO JSON document schema is called the "EmergencyIncidentDataObjectType". In the EidoLib class library, the [EidoType](~/api/Eido.EidoType.yml) class is the main class for an EIDO JSON document. This name change was for convience only and has no impact on interoperability.

The Eido namespace contains classes for all schemas defined in [NENA-STA-021.1a-2022](https://cdn.ymaws.com/www.nena.org/resource/resmgr/standards/nena-sta-021.1a_eido_json_20.pdf). The NiemTypes namespace contains the classes for the NIEM JSON schemas.

# Parsing EIDO JSON Documents
## Deserializing EIDO JSON Documents
The [EidoHelper](~/api/Eido.EidoHelper.yml) class has a method called FromString() that can be used to deserialize a string into an EidoType object. For example:
```
EidoType eido = EidoHelper.FromString(strEido);
```
This function returns null if an exception occurs so be sure to check that the returned EidoType object is not null.

It is important to note that the JSON schema for the EIDO uses property names that are actually reserved JSON keywords for metadata. For example, the ReferenceType class has a property called "$ref".

If you try to deserialize an EIDO JSON document using the JsonConvert.DeserializeObject() method (NewtonSoft.Json), an InvalidOperationException will be thrown unless you pass it a JsonSerializerSettings object with the MetadataPropertyHandling property set to MetadataPropertyHandling.Ignore. The EidoHelper.FromString() method handles this for you.

## Serializating EidoType Objects
The EidoHelper class has a method called ToJsonString() that serizlizes an EidoType object to a JSON string. For example:
```
EidoType eido = new EidoType();
// TODO: Initialize the properties of the eido object
string strEidoJson = EidoHelper.ToJsonString(eido);
```
The ToJsonString() method returns null if an exception occurs so be sure to test the return value for null.

# Extracting Data from an EIDO
An EIDO contains everything that a functional element knows about an incident at the time that it transmits the EIDO. An EIDO contains the following blocks of data. The Section and Title columns in the following table relate to [NENA-STA-021.1a-2022](https://cdn.ymaws.com/www.nena.org/resource/resmgr/standards/nena-sta-021.1a_eido_json_20.pdf).

| Section | Title | EidoType Field Name | Type |
|--------|--------|----------------|------|
| 2.5 | Agent Data Component        | agentComponent | List&lt;[AgentType](~/api/Eido.AgentType.yml)&gt;  |
| 2.6 | Agency Data Component | agencyComponent | List&lt;[AgencyType](~/api/Eido.AgencyType.yml)&gt; |
| 2.7 | Split/Merge Data Component | mergeComponent | List&lt;[MergeInformationType](~/api/Eido.MergeInformationType.yml)&gt; |
| 2.8 | Link Data Component | linkComponent | List&lt;[LinkInformationType](~/api/Eido.LinkInformationType.yml)&gt; |
| 2.9 | Incident Data Component | incidentComponent | [IncidentInformationType](~/api/Eido.IncidentInformationType.yml) |
| 2.10 | Call Data Component | callComponent | List&lt;[CallInformationType](~/api/Eido.CallInformationType.yml)&gt; |
| 2.11 | Call Back Data Component | callbackComponent | List&lt;[CallbackType](~/api/Eido.CallbackType.yml)&gt; |
| 2.12 | Updated Call Back Number Data Component | callbackComponent.updatedCbn | [UpdatedCBNType](~/api/Eido.UpdatedCBNType.yml) |
| 2.13 | Dispatch Data Component | dispatchComponent | List&lt;[DispatchInformationType](~/api/Eido.DispatchInformationType.yml)&gt; |
| 2.14 | Disposition Data Component | incidentComponent.dispositionComponent | List&lt;[DispositionType](~/api/Eido.DispositionType.yml)&gt; |
| 2.15 | Notes Data Component | notesComponent | List&lt;[NotesType](~/api/Eido.NotesType.yml)&gt; |
| 2.16 | Person Data Component | personComponent | List&lt;[PersonInformationType](~/api/Eido.PersonInformationType.yml)&gt; |
| 2.17 | Vehicle Data Component | vehicleComponent | List&lt;[VehicleInformationType](~/api/Eido.VehicleInformationType.yml)&gt; |
| 2.18 | Location Data Component | locationComponent | List&lt;[LocationInformationType](~/api/Eido.LocationInformationType.yml)&gt; |
| 2.19 | Additional Data Component | additionalDataComponent | List&lt;[AdditionalDataType](~/api/Eido.AdditionalDataType.yml)&gt; |
| 2.20 | Emergency Resource Data Component | emergencyResourceComponent | List&lt;[EmergencyResourceType](~/api/Eido.EmergencyResourceType.yml)&gt; |
| 2.21 | Alarms and Sensors Data Component | alarmsSensorsComponent | List&lt;[AlarmsSensorsType](~/api/Eido.AlarmsSensorsType.yml)&gt; |

The classes in the Eido and NiemTypes namespaces provide support for all of the data types for all components of an EIDO except the additionalDataComponent (AdditionalDataType)and the locationComponent (LocationInformationType). The AdditionalDataType class is a container for additional data in an XML format and the LocationInformationType is a container for location data in an XML data format. The EidoLib class library does not support the XML schemas for additional data and location data.

The [Ng911Lib](https://github.com/PhrSite/Ng911Lib) class library provides classes that can assist in extracting additional data and location data from an EIDO. The following sections provide examples of how to do this.

## Parsing Additional Data in an EIDO
The following code sample shows how to use the classes from the Ng911Class library to get the additional data from an EIDO object when the additional data is passed by-value. This sample does not deal with the possibility that there is more than one instance of each type of additional data block. It also does not handle the associations between the additional data blocks and other components within the EIDO.
```
using Eido;
using NiemTypes;
using AdditionalData;
using Ng911Lib.Utilities;
using Pidf;

namespace SomeNamespace;

public void ProcessAdditionalData(EidoType eido, out CommentType Ct, out DeviceInfoType Dit, out ProviderInfoType Pit,
ServiceInfoType Sit, SubscriberInfoType Sub)
{
    Ct = null;
    Dit = null;
    Pit = null;
    Sit = null;
    Sub = null;

    if (eido.additionalDataComponent == null || eido.additionalDataComponent.Count == 0)
        return;  // No additional data in the EIDO

    foreach (AdditionalDataType Adt  in eido.additionalDataComponent)
    {
        if (Adt.additionalDataByValue != null)
        {
            string strType = GetAdditionalDataType(Adt.urlPurpose, Adt.additionalDataByValue);
            // If strType is null, then handle the error

            switch (strType)
            {
                case PurposeTypes.Comment:
                    Ct = XmlHelper.DeserializeFromString<CommentType>(Adt.additionalDataByValue);
                    // If the return value of DeserializeFromString is null then an error occurred.
                    break;
                case PurposeTypes.DeviceInfo:
                    Pit = XmlHelper.DeserializeFromString<ProviderInfoType>(Adt.additionalDataByValue);
                    break;
                case PurposeTypes.ProviderInfo:
                    ProviderInfoType Pit = XmlHelper.DeserializeFromString<ProviderInfoType>(Adt.additionalDataByValue);
                    break;
                case PurposeTypes.ServiceInfo:
                    ServiceInfoType Sit = XmlHelper.DeserializeFromString<ServiceInfoType>(Adt.additionalDataByValue);
                    break;
                case PurposeTypes.SubscriberInfo:
                    Sub = XmlHelper.DeserializeFromString<SubscriberInfoType>Adt.additionalDataByValue);
                    break;
            }
        }
        else if (string.IsNullOrEmpty(Adt.additionalDataByReferenceUrl) == false)
        {
            // TODO: De-reference the additionalDataByReferenceUrl to get the additional data by sending an HTTP GET
            // request to the URL contained in the additionalDataByReference property.
        }
    }
}
```
The urlPurpose property of the AdditionalDataType class identifies which type of additional data is being provided. This information is required in order to specify the class to deserialize the XML string into.

When additional data is provided with a call, the SIP message will contain a Call-Info header that identifies the type of additional data being provided. For example:

```
Call-Info: <cid:0123456789@atlanta.example.com>;purpose=EmergencyCallData.DeviceInfo
```
In this case, the urlPurpose property will be set to the purpose parameter of the Call-Info header, which is EmergencyCallData.DeviceInfo.

When additional data is received via SIP, the EIDO creator must set the urlPurpose to the purpose parameter from the SIP Call-Info header. According to [NENA-STA-021.1a-2022](https://cdn.ymaws.com/www.nena.org/resource/resmgr/standards/nena-sta-021.1a_eido_json_20.pdf), the EIDO creator may set the urlPurpose property but it is not required to if the additional data was not received via SIP.

If the urlPurpose is null, then the EIDO consumer must test the XML string in order to determine the schema of the data being passed. The following code sample shows how this can be done.

```
private string GetAdditionalDataType(string strPurpose, string strAddDataXml)
{
    if (string.IsNullOrEmpty(strPurpose) == false)
        return strPurpose;

    string strType = null;
    if (strAddDataXml.IndexOf(PurposeTypes.Comment) >= 0)
        strType = PurposeTypes.Comment;
    else if (strAddDataXml.IndexOf(PurposeTypes.DeviceInfo) >= 0)
        strType = PurposeTypes.DeviceInfo;
    else if (strAddDataXml.IndexOf(PurposeTypes.ProviderInfo) >= 0)
        strType = PurposeTypes.ProviderInfo;
    else if (strAddDataXml.IndexOf(PurposeTypes.ServiceInfo) >= 0)
        strType = PurposeTypes.ServiceInfo;
    else if (strAddDataXml.IndexOf(PurposeTypes.SubscriberInfo) >= 0)
        strType = PurposeTypes.SubscriberInfo;

    return strType;
}
```

## Parsing Location Data in an EIDO

The locationComponent of an EidoType object contains an array of [LocationInformationType](~/api/Eido.LocationInformationType.yml) objects. The LocationInformationType class contains many properties, but the following properties provide location information.

1. locationByValue
2. locationByReferenceUrl
3. crossStreetByValue
4. crossStreetByReferenceUrl
5. intersectingStreetByValue
6. intersectingStreetByReferenceUrl

### Location By-Value and Location By-Reference
The following code sample shows how to deserialize the location data from the locationByValue and locationByReferenceUrl properties of the LocationInformationType class.

```
using Ng911Lib.Utilities;
using Pidf;

...
Presence pres1;
Presence pres2;
string strPres1 = eido.locationComponent?[0].locationByValue;
if (string.IsNullOrEmpty(strPres1) == false)
    pres1 = XmlHelper.DeserializeFromString<Presence>(strPres1)

string strPresUrl = eido.locationComponent?[0].locationByReferenceUrl;
if (string.IsNullOrEmpty(strPresUrl) == false)
{
	string strPres2 = // Dereference the URL in strPresUrl by doing an HTTP GET request. Convert the response to a string
    if (string.IsNullOrEmpty(strPres2) == false)
    	pres2 = XmlHelper.DeserializeFromString<Presence>(strPres2);
}

```

### Cross Street Location Information
The LocationInformationType class contains properties called crossStreetByValue and crossStreetByReferenceUrl. Both of these properies are arrays (actually List&lt;string&gt;) of string values. Each crossStreetByValue string property contains a PIDF-LO XML document containing a civic address representing a street address (or maybe just a street name) relative to the location information from the locationByValue or locationByReferenceUrl properties. The crossStreetByReferenceUrl contains a URL that can be de-referenced with an HTTP GET request to get the PIDF-LO XML document.

Each array may contain one or two elements, one for each cross street.

The following example shows how to parse and process the cross street by value information in a LocationInformationType object.

```
using Ng911Lib.Utilities;
using Pidf;

...
public void GetCrossStreetsByValue(LocationInformationType lit, out string strCs1, out string strCs2)
{
	strCs1 = null;
    strCs2 = null;

    if (lit == null || lit.crossStreetByValue == null || lit.crossStreetByValue.Count < 1)
        return;

	strCs1 = GetStreetAddress(Lit.crossStreetByValue[0]);
	if (lit.crossStreetByValue.Count >= 2)
    	strCs2 = GetStreetAddress(Lit.crossStreetByValue[1]);
}

public string GetStreetAddress(string strPidf)
{
	Presence pres = XmlHelper.DeserializeFromString<Presence>(strPidf);
    if (pres == null)
    	return null;	// An error occurred

	CivicAddress civicAddr = pres.GetFirstCivicAddress();
    if (civicAddress == null)
    	return null;	// Error: No civic address in the Presence object

	return civicAddress.BuildFormattedStreetAddress();
}

```

### Intersecting Street Location Information
The LocationInformationType class contains properties called intersectingStreetByValue and intersectingStreetByReferenceUrl. Both of these properies are arrays (actually List&lt;string&gt;) of string values. Each intersectingStreetByValue string property contains a PIDF-LO XML document containing a civic address representing a street address (or maybe just a street name) relative to the location information from the locationByValue or locationByReferenceUrl properties. The intersectingStreetByReferenceUrl contains a URL that can be de-referenced with an HTTP GET request to get the PIDF-LO XML document.

The string array may contain one or two elements, one for each intersecting street.

The following example shows how to process intersecting street information by value in a LocationInformationType object.

```
using Ng911Lib.Utilities;
using Pidf;

...
public void GetIntersectingStreetsByValue(LocationInformationType lit, out string strIs1, out string strIs2)
{
	strIs1 = null;
    strIs2 = null;

    if (lit == null || lit.intersectingStreetByValue == null || lit.intersectingStreetByValue.Count < 1)
        return;

	strIs1 = GetStreetAddress(Lit.intersectingStreetByValue[0]);
	if (lit.intersectingStreetByValue.Count >= 2)
    	strCs2 = GetStreetAddress(Lit.intersectingStreetByValue[1]);
}
```

The GetStreetAddress() function is shown in the sample in the Cross Street Location Information section above.

# EIDO Timestamps
All date-time objects in the EIDO must be formatted according to Section 2.3 of NENA-STA-010.3b. For example, “2022-08-21T12:58:03.01-07:00”.

The EidoHelper class has the following two methods to handle this for you.
```
public static string GetNenaTimestamp();
public static string GetNenaTimestamp(DateTime Tm);
```

The first method uses the current local time returned by DateTime.Now. The second method takes a DateTime structure containing the date and time to format into a string.

These methods should be used to ensure consistent date/time string formatting in an EIDO.
