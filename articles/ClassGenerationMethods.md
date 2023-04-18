# Introduction

The classes in the EidoLib class library were generated from the YAML schema definition files from the  [NENA911/EIDO-JSON](ht[tps://github.com/NENA911/EIDO-JSON) GitHub repository. The top directory of this repository contains a single file called openapi.yaml. This file contains the definition for the EmergencyIncidentDataObjectType component. This component is the main JSON document for the EIDO that is defined in [NENA-STA-021a](https://cdn.ymaws.com/www.nena.org/resource/resmgr/standards/nena-sta-021.1a_eido_json_20.pdf). The EmergencyIncidentDataObjectType references some of the NIEM types defined in YAML files under the NIEMComponents subdirectory. The NIEMComponents directory contains 43 YAML files that specify the 1687 NIEM types required by the EIDO schema.

The original idea was to use SwaggerHub to build the C# schema files from the YAML files provided in the NENA EIDO-JSON GitHub repository. This turned out be impossible because SwaggerHub only handles a single file containing the OpenApi YAML for an application.

In the first attempt, a single YAML file was constructed from the main openapi.yaml file and the 43 YAML files from the NIEMComponents subdirectory. All external file references were changed to local references. This file was uploaded to SwaggerHub as a project. Attempts to generate an ASP .NET Core server application caused the SwaggerHub code generator to crash.

A trouble ticket was opened with SwaggerHub.com but their technical support was unable to resolve the issue.

The second attempt was to create a single file containing the 43 files in the NIEMComponents subdirectory. All external references were converted to local references. This file was uploaded to a SwaggerHub project and the SwaggerHub code generator was able to build a .NET Core server application. The resulting project files were downloaded and only the model C# files were extracted. The SwaggerHub code generator created a total of 1687 files. Each class or enumeration was created in a separate file.

The EmergenceIncidentDataObjectType on the original openapi.yaml file was hand coded into C# classes. This was not a monumental task because there are only about 20 classes defined in NENA-STA-021a.

# Class and Namespace Name Changes
The following changes were made to the generated code.

1. The generated code for the NIEM types used the IO.Swagger.Models namespace. This was changed to NiemTypes.
2. The EmergencyIncidentDataObjectType class name was changed to EidoType. Note: This change has no effect on interoperability.
3. The EidoType class and all the sub-classes defined in NENA-STA-021a are in the Eido namespace.


# NIEMComponent Generated Class Issues

## Compile Errors
There was a single compilation error when compiling the files in NiemTypes directory. The error was in the file called MVesselAugmentationType.cs.

The generated code at line 571 was:

```
public List<ComponentsschemasniemXsboolean> IsVesselNonTankVesselResponsePlan { get; set; }
```
This error appears to be caused by a bug in the SwaggerHub code generator because the YAML definition for the "isVesselNonTankVesselResponsePlan" field is:
```
        isVesselNonTankVesselResponsePlan:
          type: array
          description: >-
            True if a vessel has a Non-Tank Vessel Response Plan (NTVRP) per 33
            CFR 151,155,160; false otherwise. NIEM reference is
            m:VesselNonTankVesselResponsePlanIndicator
          items:
            $ref: /components/schemas/niem-xsboolean
```
Apparrently the SwaggerHub code generator mangled "NiemXsboolean" into "ComponentsschemasniemXsboolean".

Line 571 was changed to:

```
public List<NiemXsboolean> IsVesselNonTankVesselResponsePlan { get; set; }
```

## Warnings

The C# classes that the SwaggerHub code generated resulted in 1456 compiler warnings. There were 1056 CS0472 warnings and 400 CS0108 warnings.

### CS0472 Warnings
Here is an example of code that resulted in the CS0472 warning from the generated file called AsmvaD20BrandCodeType.cs in the NiemTypes directory.

```
/// <summary>
/// Returns true if AamvaD20BrandCodeType instances are equal
/// </summary>
/// <param name="other">Instance of AamvaD20BrandCodeType to be compared</param>
/// <returns>Boolean</returns>
public bool Equals(AamvaD20BrandCodeType other)
{
    if (ReferenceEquals(null, other)) return false;
    if (ReferenceEquals(this, other)) return true;

    return
       (
           SimpleType == other.SimpleType ||
           SimpleType != null &&
           SimpleType.Equals(other.SimpleType)
           ) &&
           (
               Context == other.Context ||
               Context != null &&
               Context.Equals(other.Context)
           );
}
```

The warning text was:

> The result of the expression is always 'true' since a value of type 'AamvaD20BrandCodeSimpleType' is never equal to 'null' of type 'AamvaD20BrandCodeSimpleType'

The reason for this warning is that SimpleType is a property and is not (actually cannot be) declared as nullable. Therefore, the expression ```SimpleType != null``` will always be true. This is actually not a serious problem because the code will work correctly. It was decided to disable this warning in the project settings rather than fixing all 1056 occurrences of it.

### CS0108 Warnings
An example of warning CS0108 occurs at line 104 of the file called JConditionalReleaseType.cs in the NiemTypes directory.

```
[DataMember(Name="@context")]
public ContextEnum? Context { get; set; }
```

The warning text was:

> 'JConditionalReleaseType.Context' hides inherited member 'NcReleaseType.Context'. Use the new keyword if hiding was intended.

This warning appears to be benign so it was decided to disable this warning in the project file settings.





