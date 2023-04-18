# The EidoLib Class Library

The EidoLib project is a class library that provides a set of classes for the data schemas required for implementing the Emergency Incident Data Object (EIDO). The EIDO is a JSON schema that is used to pass information about an emergency call or an incident between functional elements in an NG9-1-1 emergency services network. The EIDO is typically used for the following purposes.

1. To provide all of the information about an emergency when one PSAP is transferring a call to another PSAP
2. A PSAP can send an EIDO to a Computer Aided Dispatch (CAD) system.
3. CAD/PSAP systems can share information about an incident using EIDOs in situations in which multiple agencies are involved in an emergency incident.

The classes in this class library are primarily data model classes. This class library does not provide any functionality to support transport of EIDOs between functional elements. The following standards specify the methods that are used for transporting EIDOs between functional elements.

1. PSAP to PSAP call transfers. See Section 4.7.1 of [NENA-STA-010.3b](https://cdn.ymaws.com/www.nena.org/resource/resmgr/standards/nena-sta-010.3b-2021_i3_stan.pdf).
2. PSAP to CAD and CAD to CAD. See NENA Standard for the Conveyance of Emergency Incident Data Objects (EIDOs) between Next Generation (NG9-1-1) Systems and Applications. [NENA-STA-024.1a-2023](https://cdn.ymaws.com/www.nena.org/resource/resmgr/standards/nena-sta-024.1a-2023_eidocon.pdf)
3. [WebSocket Subscribe/Notify Schema](https://github.com/NENA911/WebSocket-Subscribe-Notify)

The following document specifies the schema of the EIDO.
> [NENA Standard for Emergency Incident Data Object (EIDO)](https://cdn.ymaws.com/www.nena.org/resource/resmgr/standards/nena-sta-021.1a_eido_json_20.pdf), National Emergency Number Association (NENA) Agency Systems Committee, EIDO JSON Working Group, NENA-STA-021.1a-2022, April 19, 2022.

The GitHub project called [NENA911/EIDO-JSON](https://github.com/NENA911/EIDO-JSON) provides the YAML schema definitions for the EIDO.

The classes in this classes library were generated from the NENA standard and the YAML schema definitions as described in [Class Generation Methods](~/articles/ClassGenerationMethods.md).

This portable, cross-platform class library is written in the C# language and the library package targets the .NET 7.0 environment. It may be used by applications that target the Windows (version 10 or later) or Linux operating systems.
