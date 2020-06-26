## Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED”, “MAY", and “OPTIONAL" in this document are to be interpreted as described in RFC 2119.

The octet order over the air for all multi-octet fields is little endian (Least significant byte is sent first).


## Introduction

This document defines an application layer messaging package running over LoRaWAN to perform the following operations on a fleet of end-devices:
-	Program a multicast distribution window into a group of end-devices
-	Having all end-devices of the group switch to ClassB or ClassC temporarily at the beginning of the slot
-	Close the distribution window and revert to normal operation (e.g. return to Class A, or change to a different periodicity in Class B)

All messages described in this document are transported as application layer messages. As such, all unicast messages (uplink or downlink) are encrypted by the LoRaWAN MAC layer using the end-device’s AppSKey. Downlink multicast messages are encrypted using a multicast group McAppSKey common to all end-devices of the group. The setup of the group as well as means to convey the MCAppSKey are described in the document.
The <strong>“multicast control”</strong> package can be used to:
-	Remotely create a multicast group security context inside a group of end-devices
-	Report the list of multicast context existing in the end-device
-	Remotely delete a multicast security context.
-	Program a classC multicast session
-	Program a classB multicast session

This package uses a dedicated port to separate its traffic from the rest of the applicative traffic.


## Multicast group context definition

This package makes the following assumptions.
Inside a given end-device a multicast group is defined by the following parameters (the multicast group context):

1. A McGroupID:  an integer in [0:3], the index of the multicast group. This index is used as an end-device specific shortcut to reference one of the multicast groups defined inside the end-device. An end-device supports a maximum of 4 simultaneous multicast groups, and a minimum of 0.
	
2. Multicast address:  the 4 bytes network address of the multicast group, common to all end-devices of the group. 
	
3. A multicast group key (McKey) from which are derived a McAppSKey and a McNwkSKey. The McKey is multicast group specific (different for every multicast group), but all end-devices of a given multicast group have the same McKey associated to this group 

4. A frame counter.
	
Because the end-device can be part of up to 4 multicast groups, every multicast control command MUST first define which multicast group is concerned by the command. To minimize the protocol overhead, a 2-bit McGroupID shortcut is used instead of the full 4 bytes multicast group network address in most of the commands defined in this package.An end-device MAY support up to 4 multicast groups contexts defined simultaneously. If an end-device supports N simultaneous multicast group contexts where 1<=N<=4 then the McGroupID can only be in the range [0:N-1].

```	
For example, if an end-device is designed to support only a single multicast group, then this group can only have McGroupID=0.
```
## Multicast Control Message Package

The identifier of the multicast control package is 2. The version of this package is version 1.

The following messages are sent to each end-device individually using Unicast downlink on a port specifically used for the multicast package. The default port value is 200. These messages MUST NOT be sent using multicast. If these messages are received on a multicast address the end-device MUST drop them silently.


All unicast control messages use the same format:
<table>
  <tr> <td>Command1</td> <td>Command1 Payload</td> <td>Command2</td> <td>Command2 payload</td> <td>....</td> </tr>
</table>

A message MAY carry more than one command. The length of each command’s payload is fixed and a function of the command. Commands are executed from first to last. Each command MUST be individually acknowledged by the end-device.

The following table summarizes the list of multicast control messages

<table>
    <caption>Multicast Control messages summary </caption>
    <thead>
        <tr>
            <th>CID</th>
            <th>Command name</th>
            <th>Transmitted by end-device</th>
            <th>Transmitted by Server</th>
            <th>Short Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>0x00</td>
            <td>PackageVersionReq</td>
            <td></td>
            <td>x</td>
            <td>Used by the AS to request the package version implemented by the end-device</td>
        </tr>
	    <tr>
            <td>0x00</td>
            <td>PackageVersionAns</td>
            <td>x</td>
            <td></td>
            <td>Conveys the answer to PackageVersionReq</td>
        </tr>
        <tr>
            <td>0x01</td>
            <td>McGroupStatusReq</td>
            <td></td>
            <td>x</td>
            <td>Asks an end-device to list the multicast groups currently configured</td>
        </tr>
	<tr>
            <td>0x01</td>
            <td>McGoupStatusAns</td>
            <td>x</td>
            <td></td>
            <td>Conveys answer to the McGroupStatus request</td>
        </tr>
        <tr>
            <td>0x02</td>
            <td>McGroupSetupReq</td>
            <td></td>
            <td>x</td>
            <td>Provides an end-device will all necessary information to join a multicast group</td>
        </tr>
        <tr>
            <td>0x02</td>
            <td>McGroupSetupAns</td>
            <td>x</td>
            <td></td>
            <td></td>
        </tr>
	<tr>
            <td>0x03</td>
            <td>McGroupDeleteReq</td>
            <td></td>
            <td>x</td>
            <td>Used to delete a multicast group from an end-device</td>
        </tr>
	 <tr>
            <td>0x03</td>
            <td>McGroupDeleteAns</td>
            <td>x</td>
            <td></td>
            <td></td>
        </tr>
	 <tr>
            <td>0x04</td>
            <td>McClassCSessionReq</td>
            <td></td>
            <td>x</td>
            <td>Conveys information about the next classC multicast session the end-device shall join</td>
        </tr>
	 <tr>
            <td>0x04</td>
            <td>McClassCSessionAns</td>
            <td>x</td>
            <td></td>
            <td></td>
        </tr>
	 <tr>
            <td>0x05</td>
            <td>McClassBSessionReq</td>
            <td></td>
            <td>x</td>
            <td>Creates a class B multicast session </td>
        </tr>
	 <tr>
            <td>0x05</td>
            <td>McClassBSessionAns</td>
            <td>x</td>
            <td></td>
            <td></td>
        </tr>
    </tbody>
</table>

### PackageVersionReq & Ans

The <strong>PackageVersionReq</strong> command has no payload.
The end-device answers with a <strong>PackageVersionAns</strong> command with the following payload.

<table>
    <caption>PackageVersionAns</caption>
    <thead>
        <tr>
            <th>Field</th>
            <th>Size(bytes)</th>
        </tr>
        <tr>
            <td>PackageIdentifier</td>
            <td>1</td>
        </tr>
	<tr>
            <td>PackageVersion</td>
            <td>1</td>
        </tr>
    </tbody>
</table>

### Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in \[RFC2119\].

All sections and appendixes, except "Scope" and "Introduction", are normative, unless they are explicitly indicated to be informative.

### Definitions

<table>
    <caption>Definitions</caption>
        <tbody>
            <tr>
                <td><strong>LwM2M Bootstrap-Server Account</strong></td>
                <td>LwM2M Security Object Instance with Bootstrap-Server Resource set to 'true'.</td>
            </tr>
            <tr>
                <td><strong>LwM2M Server Account</strong></td>
                <td>LwM2M Security Object Instance with Bootstrap-Server Resource set to 'false' and associated LwM2M Server Object Instance.</td>
            </tr>
        </tbody>            
</table>

Kindly consult [\[OMADICT\]](http://openmobilealliance.org/documents/dictionary/OMA-ORG-Dictionary-V2_9-20120626-A.pdf) for more definitions used in this document.

### Abbreviations

<table>
    <caption>Abbreviations</caption>
        <tbody>
            <tr>
                <td><strong>CoAP</strong></td>
                <td>Constrained Application Protocol</td>
            </tr>
            <tr>
                <td><strong>DTLS</strong></td>
                <td>Datagram Transport Layer Security</td>
            </tr>
	     <tr>
                <td><strong>LoRaWAN</strong></td>
                <td>LOng RAnge Wide Area Network</td>
            </tr>
             <tr>
                <td><strong>NB-IoT</strong></td>
                <td>NarrowBand Internet of Things</td>
            </tr>
	    <tr>
                <td><strong>OCSP</strong></td>
                <td>Online Certificate Status Protocol</td>
            </tr>
	     <tr>
                <td><strong>SMS</strong></td>
                <td>Short Message Service</td>
            </tr>
	    <tr>
                <td><strong>TCP</strong></td>
                <td>Transmission Control Protocol</td>
            </tr>
            <tr>
                <td><strong>TLS</strong></td>
                <td>Transport Layer Security</td>
            </tr>
	     <tr>
                <td><strong>UDP</strong></td>
                <td>User Datagram Protocol</td>
            </tr>
	    <tr>
              <td><strong>OSCORE</strong></td>
              <td>Object Security for Constrained RESTful Environments</td>
           </tr>
        </tbody>            
</table>

## Introduction

This specification defines the transport bindings for the LwM2M messaging protocol used between LwM2M Clients, LwM2M Bootstrap Servers and LwM2M Servers. [Figure The Protocol Stack of the LwM2M Enabler]() shows the relationships between the transport bindings and the messaging protocol. In particular, this specification defines the following transport bindings: 

-   CoAP over UDP
-   CoAP over DTLS (over UDP)
-   CoAP over SMS 
-   CoAP over DTLS over SMS 
-   CoAP over TCP
-   CoAP over TLS (over TCP)
-   CoAP over Non-IP (includes 3GPP CIoT and LoRaWAN)

<figure>
      <img src="images/protocol_stack.svg" alt="The Protocol Stack of the LwM2M Enabler">
      <figcaption>The Protocol Stack of the LwM2M Enabler</figcaption>
</figure>

OSCORE can be used with any of the transport bindings including UDP, SMS and TCP, with or without DTLS or TLS.

## Security

The LwM2M protocol supports various transport bindings and credentials for securely communicating with LwM2M Servers. This configuration information can be provisioned through various forms of bootstrapping methods.

LwM2M supports three different types of credentials, namely

-   Certificates,

-   Raw public keys, and

-   Pre-shared secrets.

Since these credential types offer different properties, the LwM2M specification offers support for all of them. \[RFC7925\] provides the necessary details about the use of each of these credentials with TLS/DTLS. The LwM2M specification also supports application layer security based on OSCORE with pre-shared secrets. Which credential is best in a given deployment depends on a number of factors, including the threat model and IoT device constraints. 

The LwM2M protocol specifies that authorization of LwM2M Servers to access Object Instances and Resources within the LwM2M Client is provided through Access Control Object Instances within the LwM2M Client.

### Security Requirements

LwM2M may under certain circumstances be deployed with security outside the scope of LwM2M TS, as specified in other parts of this section. However, for LwM2M security solutions the requirements specified in this section apply.

The LwM2M protocol requires that all communication between LwM2M Clients, LwM2M Servers and LwM2M Bootstrap-Servers to perform mutual authentication. This means:

-   a LwM2M Client MUST authenticate a LwM2M Server prior to exchange of any data.

-   a LwM2M Server MUST authenticate a LwM2M Client prior to exchange of any data.

-   a LwM2M Client MUST authenticate a LwM2M Bootstrap-Server prior to exchange of any data.

-   a LwM2M Bootstrap-Server MUST authenticate a LwM2M Client prior to exchange of any data.

The LwM2M protocol also requires that all communication between LwM2M Clients and LwM2M Servers as well as between LwM2M Clients and LwM2M Bootstrap-Servers is encrypted and integrity protected. This means:

-   a LwM2M Client MUST encrypt and integrity protect data communicated to a LwM2M Server.

-   a LwM2M Server MUST encrypt and integrity protect data communicated to a LwM2M Client.

-   a LwM2M Client MUST encrypt and integrity protect data communicated to a LwM2M Bootstrap-Server.

-   a LwM2M Bootstrap-Server MUST encrypt and integrity protect data communicated to a LwM2M Client.

Additionally:

- A security solution MUST be able to provide replay protection of LwM2M Operations 

- A security solution MUST be able to securely bind LwM2M responses with LwM2M requests

- For certain operations, the LwM2M Client MUST be able to verify the freshness of the request 

- A security solution MUST be able to support secure fragmentation of the messages between LwM2M Server and LwM2M Client into fragments that can be verified separately, in particular in the case of firmware updates. 

The security requirements applies also to communication via LwM2M aware and LwM2M unaware intermediate nodes.

In addition to communication security care must also be taken to ensure that data at rest, such as secrets and sensor data, are also secured against unauthorized access. 

### TLS/DTLS-based Security

#### TLS/DTLS Overview

CoAP over UDP \[CoAP\] is secured using the Datagram Transport Layer Security (DTLS) 1.2 protocol \[RFC6347\], as defined in Section 9 of \[CoAP\]. DTLS is a communication security solution for datagram based protocols (such as UDP). More recently support for CoAP over TCP / TLS has been defined in \[CoAP-TCP\]. TLS 1.2 is specified in \[RFC5246\]. TLS and DTLS provide mutual authentication, data integrity and confidentiality. This version of the specification MAY use TLS 1.3

This section provides information related to the use of CoAP over DTLS and CoAP over TLS. Section [SMS Channel Security]() provides additional information regarding the use of DTLS in an SMS context. Implementations SHOULD be conformant to \[RFC7925\], which includes not only ciphersuites but also recommendations for the use of TLS/DTLS-specific extensions. 

The TLS/DTLS client and the TLS/DTLS server SHOULD keep security state, such as session keys, sequence numbers, and initialization vectors, and other security parameters, established with TLS/DTLS for as long a period as can be safely achieved without risking compromise of the security context. If such state persists across sleep cycles where the RAM is powered off, secure storage SHOULD be used for the security context.

The credentials used for authenticating the TLS/DTLS client and the TLS/DTLS server to secure the communication between the LwM2M Client and the LwM2M Server are obtained using one of the bootstrap modes defined in [LwM2M-CORE].

LwM2M Bootstrap-Servers, LwM2M Servers and LwM2M Clients MUST use different key pairs. LwM2M Clients MUST use keys, which are unique to each LwM2M Client. When a LwM2M Client is configured to utilize multiple LwM2M Servers then the LwM2M Bootstrap-Server may configure different credentials with these LwM2Ms Servers. Using different credentials with each LwM2M Server provides better unlinkability properties since each individual LwM2M Server cannot correlate requests made by the LwM2M Client to other LwM2M Servers. Deployment and application specific considerations dictate what approach to use, but using different credentials is generally recommended practice.

#### Ciphersuites

TLS/DTLS supports the concept of ciphersuites and they are securely negotiated during the handshake. This specification recommends support of a limited number of ciphersuites based on \[RFC7925\]. These ciphersuites have been chosen because of suitability for IoT devices, security reasons and to improve interoperability. Ciphersuites in TLS/DTLS 1.2 depend on the type of credential being used since the ciphersuite concept also indicates the authentication and key exchange mechanism. LwM2M Clients and LwM2M Servers MAY support additional ciphersuites that conform to the state-of-the-art security requirements.

#### Elliptic Curves

For ECDHE, the group secp256r1 SHALL be supported. For ECDSA the curve secp256r1 SHALL be supported. Elliptic curve groups of less than 255 bits SHALL not be supported.

#### Bootstrapping

The Resources in the LwM2M Security Object are used for

1.  providing TLS/DTLS communication security for "Client Registration", "Device Management & Service Enablement", and "Information Reporting" Interfaces if the LwM2M Security Object Instance relates to a LwM2M Server, or,

2.  providing channel security for the Bootstrap interface if the LwM2M Security Object instance relates to a LwM2M Bootstrap-Server.

3.  protecting the communication with a firmware repository when the LwM2M Client receives a URI in the Package URI of the Firmware Update object.

The content and the interpretation of the Resources in the LwM2M Security Object depends on the type of credential being used.

When bootstrapping from a Smartcard a secure channel between the Smartcard and the LwM2M Client SHOULD be established, as described in Appendix G of [LwM2M-CORE] and defined in \[GLOBALPLATFORM 3\], \[GP SCP03\]. Using a Smartcard with pre-shared secrets, with raw public keys, and with certificates needs no pre-existing trust relationship between LwM2M Server(s) and LwM2M Client(s): the pre-established trust relationship exists between the LwM2M Server(s) and the SmartCard.

LwM2M Clients MUST either be provisioned for use with a LwM2M Server or else be provisioned for use with a LwM2M Bootstrap-Server. Any LwM2M Client, which supports the client initiated bootstrap mode, MUST support at least one of the following secure methods:

1.  Bootstrapping with a strong (high-entropy) pre-shared secret, as described in Section [Pre-Shared Keys](). The ciphersuites defined in Section [Pre-Shared Keys]() MUST NOT be used with a low-entropy secret or with a password.

2.  Bootstrapping with a raw public key or a certificate, as described in Section [Raw Public Keys]() and Section [X.509 Certificates]().

LwM2M Clients MUST be provisioned with credentials that are unique to their device. For full interoperability in all deployment environments, a LwM2M Bootstrap-Server MUST support bootstrapping via pre-shared secrets, raw public keys, and certificates.

NOTE: The LwM2M Bootstrap-Server can also provision KIc and KID for the use of SMS Secured Packet Structure mode (see Section [SMS Channel Security]()).

Security credential dynamically provisioned to the LwM2M Client and the LwM2M Server MAY change at any time, even during the lifetime of an ongoing TLS/DTLS session. Since the TLS/DTLS protocol verifies the credentials only at the beginning of the session establishment (unless the re-negotiation feature is used) it is possible that a change in credential (for example, credentials for the use of a PSK-based ciphersuite) occurs after a TLS/DTLS handshake has already been completed. Hence, from a TLS/DTLS protocol point of view such credential change is not automatically recognized and the already established record layer security associations are in use. It is a policy decision for a TLS/DTLS client as well as a TLS/DTLS server implementation to close a connection when the credentials change. Such a decision will depend on various factors, such as the application domain in which LwM2M is used. The LwM2M specification does not mandate a specific behaviour since TLS/DTLS allows both communication parties to tear down an established TLS/DTLS session for any number of reasons.

The Security Mode Resource in the Security Object determines what credentials are being used by the LwM2M Client and the LwM2M Server or LwM2M Bootstrap-Server, respectively. Currently five security modes are defined, namely 

 * 0: Pre-Shared Key mode 
 * 1: Raw Public Key mode 
 * 2: Certificate mode 
 * 3: NoSec mode 
 * 4: Certificate mode with EST 

The Enrollment over Secure Transport (EST) bootstrap mode is a certificate management protocol for provisioning certificates from the LwM2M Bootstrap-Server to the LwM2M Client. EST allows the LwM2M Client to generate the key pair locally on the LwM2M Device. The private key therefore never leaves the LwM2M Device. In order to use this mode the "Security Mode" Resource MUST be set to value 4 and the certificate of the TLS/DTLS server MUST be provisioned to the "Server Public Key" Resource. This triggers the LwM2M Client to locally generate a public / private key pair and to initiate an EST over CoAP protocol exchange \[CoAP-EST\] to obtain a certificate. Once the certificate has been obtained it behaves like the Certificate mode. The EST over CoAP specification \[CoAP-EST\] profiles the use of EST for use in constrained environments.

Generating high quality random numbers of IoT devices, which is required by the EST mode utilized by this specification, is often a concern. When generating a public / private key pair, the random generator used by the LwM2M Client MUST respect the characteristics of a sufficiently high quality random bit generator, such as defined for example by RFC 4086 \[RFC4086\] or NIST Special Publication 800-90A Revision 1 \[SP800-90A\].

Compared to the certificate mode with the "Security Mode" Resource set to value 2 additional over-the-air overhead is introduced by this mode since the LwM2M Client needs to convey the public key to the EST server and needs to demonstrate possession of the private key using the PKCS\#10 defined mechanism, as explained in the EST specification. Depending on the deployment environment this additional overhead needs to be compared against the added security benefit of not disclosing the private key to other parties. LwM2M Clients SHOULD use an existing key pair or generate the public/private key pair locally since this does not expose the private key to the LwM2M Bootstrap-Server or other parties, i.e., the private key pair SHOULD never leave the device. 

Note: The "Secret Key" and the "Public Key or Identity" Resources are not used by the Certificate mode with EST.

Enrollment over Secure Transport (EST) offers multiple features, including

-   Simple PKI messages,

-   CA certificate retrieval,

-   CSR Attributes Request, 

-   Server-generated key request,

but only the first two are mandatory to implement when the EST over CoAP is used for bootstrapping since the functionality for server-generated key requests is already covered as part of the security mode (1 - Raw Public Key mode and 2 - Certificate mode). CSR Attributes Request is also not required for this specification since the LwM2M Bootstrap Server is typically in possession of the required attributes for generating a certificate. 

#### Unbootstrapping

The term 'unbootstrapping' refers to the process of deleting an instance of a LwM2M Security Object. If such an instance of a Security Object is to be deleted related resources and configurations need to be deleted or modified as well. This ensures that there is no orphan data, for example LwM2M Server Object instances with no security context. Therefore, when the Delete operation is sent via the Bootstrap Interface, the Client MUST execute the following procedure.

1. The Server Object Instance, which the deleted Security Object referred to, MUST be deleted. The Client MAY send the "De-register" operation to the LwM2M Server. 

2. If there is an Object Instance that can be accessed only by a LwM2M Server of the Server Object Instance (i.e., the LwM2M Server is Access Control Owner and the LwM2M Server can access the Object Instance only in an Access Control Object Instance), the Object Instance and the corresponding Access Control Object Instance MUST be deleted.

3. If an Object Instance can be accessed by multiple LwM2M Servers including the LwM2M Server which Security Object Instance is to be deleted, then

- The ACL Resource Instance for the LwM2M Server in the Access Control Object Instance for the Object Instance MUST be deleted.

- If the LwM2M Server is the Access Control Owner of the Access Control Object Instance, then the Access Control Owner MUST be changed to another LwM2M Server according to the following rules: The Client MUST choose the LwM2M Server who has highest sum of each number assigned to an access right (Write: 1, Delete: 1) for the Access Control Owner. If two or more LwM2M Servers have the same sum, the Client MUST choose one of them as the new Access Control Owner.

4. Observation operations from the LwM2M Server Object Instance MUST be deleted.

Note: To monitor the change of the Access Control Owner the LwM2M Server MAY observe the Access Control Owner Resource.

#### Endpoint Client Name

This specification recommends, but does not mandate, transmission of the endpoint client name in the Bootstrap-Request and in the Register message. Since the endpoint client name is not authenticated at the application layer the LwM2M Server MUST compare the received endpoint client name identifier with the identifier used at the TLS/DTLS handshake. This comparison may either be an equality match or may involve a dedicated lookup table to ensure that LwM2M Clients cannot intentionally or due to misconfiguration impersonate other LwM2M Clients. The LwM2M Server MUST respond with a "4.00 Bad Request" to the LwM2M Client if these fields do not match.

#### LwM2M and TLS/DTLS Roles

In the LwM2M enabler, the LwM2M Client is always the TLS/DTLS client. The LwM2M Bootstrap-Server and the LwM2M Server are always acting as TLS/DTLS servers. 

#### DTLS Connection ID

In the current version of DTLS, the IP address and port of the peer are used to identify the DTLS association.  Unfortunately, in some cases, such as NAT rebinding, these values are insufficient. This is a particular issue when devices enter extended sleep periods to increase their battery lifetime. The NAT rebinding leads to connection failure, with the resulting cost of a new handshake.

\[DTLS-1.3\] and \[DTLS-1.2-CID\] define extensions to DTLS 1.3 and DTLS 1.2 to add a Connection ID (CID), an identifier carried in the record layer header that gives the recipient additional information for selecting the appropriate security association. The use of this extension allows LwM2M Clients to maintain an established DTLS security context for a longer period of time.

#### Credential Types

[RFC7925] gives recommendations for three types of credentials, namely pre-shared keys, raw public keys, and X.509 certificates. LwM2M works with all three types of credentials but the performance and security trade-offs for these three mechanisms are different. As a summary, the three credential types have the following properties:

- The pre-shared key profile offers the most efficient solution for integration of TLS/DTLS into LwM2M since pre-shared ciphersuites recommended in [RFC7925] require a minimum amount of flash space as well as RAM size. Symmetric cryptographic algorithms require only a minimal computational overhead. The size of the exchanged messages is also kept at a minimum. There is, however, a downside as well: symmetric keys need to be pre-configured to both communication endpoints.

- The certificate-based profile re-uses widely deployed X.509 certificates. This allows both tools as well as existing infrastructure, such as Certification Authorities (CAs) and the Public Key Infrastructure (PKI), to be re-used. Unlike the typical web browser use of certificates, [RFC7925] specifies the use of certificates for mutual authentication between clients and servers. The use of certificates comes at a price: The use of asymmetric cryptography is more complex to implement, requires more bandwidth for the exchanged messages, is computationally more demanding, and requires a larger code size as well as more RAM. The benefits are, in addition to the re-use of existing technologies, the need to only share the certificates with other communication partners in an authentic fashion and to keep the private key local to each party (at least in the case where EST is used). This property of asymmetric cryptography reduces the risk of exposing private keying material.

- The raw public key profile offers features that sit between the pre-shared key and the certificate-based profile and combines the benefits of these two profiles. The use of asymmetric cryptography offers improved security but avoids the overhead associated with certificates and the PKI.

The following sub-sections define the use of various resources in the LwM2M Security Object for the specific credential types. The "LwM2M Server URI", and the "Bootstrap Server" Resources are populated according to the description in [LwM2M-CORE].

##### Pre-Shared Keys

If a LwM2M Server supports the pre-shared key credentials it MUST support the following:
* TLS\_PSK\_WITH\_AES\_128\_CCM\_8, as defined in \[RFC6655\] and mandated in \[RFC7925\]
* TLS\_PSK\_WITH\_AES\_128\_CBC\_SHA256, as defined in [RFC5487]. 

The LwM2M Client SHOULD NOT use the TLS\_PSK\_WITH\_AES\_128\_CBC\_SHA256 ciphersuite as RFC 7457 [RFC7457] has identified security attacks against these TLS/DTLS ciphersuites.

A LwM2M v1.1 or v1.2 Client MUST support TLS\_PSK\_WITH\_AES\_128\_CCM\_8 and MAY support additional ciphersuites.

This mode requires the following resources of the Security Object to be populated:

-   The "Security Mode" Resource MUST contain the value 0.

-   The "Public Key or Identity" Resource MUST be used to store the PSK identity, described in \[RFC7925\]. Clients and Servers MUST support arbitrary PSK Identities of up to 128 bytes, as mandated in \[RFC7925\].

-   The "Secret Key" Resource MUST be used to store the PSK, defined in \[RFC4279\]. Since the default PSK ciphersuite defined in this specification use a 128-bit AES key it is RECOMMENDED to provision a 16 byte (128 bit) key or longer in the Secret Key Resource.  Clients and Servers MUST support PSK keys of up to 64 bytes in length, as required by \[RFC7925\]. Recommendations for generating random keys are provided in RFC 4086 [RFC4086] and in NIST Special Publication 800-90A Revision 1 \[SP800-90A\].

-   The "Server Public Key" Resource MUST NOT be used in the Pre-Shared Key mode.

##### Raw Public Keys

If a LwM2M Server supports the raw public key credentials it MUST support the following:
* TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_CCM\_8, as defined in \[RFC6655\] and mandated in \[RFC7925\]
* TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_CBC\_SHA256, as defined in [RFC5289]

The LwM2M Client SHOULD NOT use the TLS\_PSK\_WITH\_AES\_128\_CBC\_SHA256 ciphersuite as RFC 7457 [RFC7457] has identified security attacks against these TLS/DTLS ciphersuites.

A LwM2M v1.1 or v1.2 Client MUST support TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_CCM\_8 and MAY support additional ciphersuites. Ciphersuites SHOULD have ECDSA authentication and SHOULD have ECDHE key exchange.

This mode requires the following resources of the Security Object to be populated:

-   The "Security Mode" Resource MUST contain the value 1.

-   The information stored in the "Public Key or Identity" Resource depends on the "Certificate Usage" Resource, as discussed in Section [Certificate Usage Field](). If the "Certificate Usage" Resource is missing this Resource MUST be used to store the raw public key encoded using the SubjectPublicKeyInfo structure, as described in [RFC7250].

-   The "Secret Key" Resource MUST be used to store the private key of the TLS/DTLS client encoded as OneAsymetricKey, as defined in [RFC5958].

-   The "Server Public Key" Resource MUST be used to store the raw public key of the TLS/DTLS server encoded using the SubjectPublicKeyInfo structure, as described in [RFC7250].

This security mode is appropriate for LwM2M deployments where the benefits of asymmetric cryptography are needed but the PKI functionality is undesirable.

The TLS/DTLS client MUST check that the raw public key presented by the TLS/DTLS server matches this stored public key.

The TLS/DTLS server MUST store its own private and public keys, and MUST have a stored copy of the expected client public key. The TLS/DTLS server MUST check that the raw public key presented by the TLS/DTLS client exactly matches this stored public key.

##### X.509 Certificates

The X.509 Certificate mode requires the use of X.509v3 certificates \[RFC5280\]. The X.509 certificates SHOULD be conformant with the profiling given in \[RFC7925\]. The curve secp256r1 SHALL be supported. Elliptic curve groups of less than 255 bits SHALL NOT be supported.

If a LwM2M Server supports X.509 Certificate mode it MUST support:
* TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_CCM\_8, as defined in \[RFC7251\] and mandated in \[RFC7925\]
* TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_CBC\_SHA256, as defined in [RFC5289]

The LwM2M Client SHOULD NOT use the TLS\_PSK\_WITH\_AES\_128\_CBC\_SHA256 ciphersuite as RFC 7457 [RFC7457] has identified security attacks against these TLS/DTLS ciphersuites.

A LwM2M v1.1 or v1.2 Client MUST support TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_CCM\_8 and MAY support additional ciphersuites. Ciphersuites SHOULD have ECDSA authentication and SHOULD have ECDHE key exchange.

This mode requires the following resources of the Security Object to be populated:

-   The "Security Mode" Resource MUST contain the value 2.

-   The information stored in the "Public Key or Identity" Resource depends on the Certificate Usage Resource, as discussed in Section [Certificate Usage Field](). If the Certificate Usage Resource is missing this Resource MUST be used to store the X.509 certificate of the TLS/DTLS client in a DER encoded binary format. This backwards compatibility mode matches the semantic of value 3 ("domain-issued certificate") in the Certificate Usage Resource. 

-   The "Secret Key" Resource MUST be used to store the private key of the TLS/DTLS client as a DER encoded asymmetric key package OneAsymetricKey version 1 structure, as defined in Section 2 of [RFC5958]. Note that this version of the specification does not support version 2 of RFC 5958 and the encrypted private key info structure defined in Section 3 of [RFC5958].

-   The "Server Public Key" Resource MUST be used to store either a trust anchor certificate suitable for path validation of the certificate of the TLS/DTLS server, or directly the certificate of the TLS/DTLS server ("domain-issued certificate mode"). The use of it is explained in more detail below.

When certificate-based authentication is used in TLS/DTLS to protect the LwM2M communication at least two types of certificates need to be distinguished, namely client (or device) certificates and certificates presented by the server. This distinction matters since the identifiers used in the certificates will be different and the procedure for identifier matching will be different as well. 

Client certificates SHOULD contain the endpoint client name in the subject CN (Common Name) attribute. The LwM2M Server MUST perform the security checks outlined in [Endpoint Client Name]().

The subsequent text talks about the certificates used by the server. 

The algorithm for verifying the service identity, as described in Section 4.4.1 of \[RFC7925\] and in \[RFC6125\], is essential for ensuring proper security when certificates are used and MUST be implemented and used by the TLS/DTLS client. Terms like reference identifier and presented identifier are defined in RFC 6125.

Comparing the reference identifier against the presented identifier obtained from the certificate is required to ensure the TLS/DTLS client is communicating with the intended TLS/DTLS server. To cater for the case that the Server Public Key Resource directly contains the certificate of the TLS/DTLS server, TLS/DTLS client does not compare the reference identifier against the presented identifier if the certificate from the Server Public Key Resource matches the certificate provided by the TLS/DTLS server during the TLS/DTLS handshake. If that is not the case, the TLS/DTLS client MUST compare the reference identifier against the presented identifier as described below, and perform path validation.

The algorithm description from RFC 6125 assumes that fully qualified DNS domain names (FQDN) are used. If a server node is provisioned with a FQDN, then the TLS/DTLS server certificate MUST contain this "FQDN" (i.e., presented identifier). This FQDN is stored in the SubjectAltName or in the Common Name (CN) component of the subject name, as explained in Section 9.1.3.3 of \[RFC7252\], and this FQDN is used by the TLS/DTLS client to match it against the FQDN used during the lookup process (i.e., reference identifier) before contacting the LwM2M Server, as described in \[RFC6125\]. 

Note that the Server Name Indication (SNI) extension \[RFC6066\] allows a TLS/DTLS client to tell a TLS/DTLS server the name of the TLS/DTLS server it is trying to contact. This is an important feature when the server is part of a hosting solution where multiple virtual servers are using a single underlying network address. Section 3 of \[RFC6066\] only allows FQDN hostname of the TLS/DTLS server in the ServerName field. For the TLS/DTLS client running on a LwM2M Server the SNI extension allows the LwM2M Server to indicate what certificate it is expecting. The SNI extension MUST be set by the LwM2M client.

##### Deployments without DNS

In some deployment scenarios, DNS is not used and LwM2M Clients need to use the IP literal, such as coaps://\[2001:db8::2:1\]/, stored in the "LwM2M Server URI" Resource. To avoid having to use IP addresses also in the Common Name (CN) component of the server certificate or in a field of URI type in the SubjectAltName set this specification uses the Server Name Indication (SNI) Resource. 

The procedure for a client is as follows:

-   The LwM2M Client uses the IP address from the "LwM2M Server URI" Resource to connect to the LwM2M Server using a TLS/DTLS handshake. 

-   The TLS/DTLS Client uses the value stored in the SNI Resource to help the LwM2M Server select the appropriate server certificate. The SNI becomes the reference identifier.

-   The TLS/DTLS stack of the LwM2M Server returns a Certificate message as part of the handshake that contains a certificate. The identifier extracted from the server certificate becomes the presented identifier.

-   The TLS/DTLS client matches the reference identifier against the presented identifier. If the two match, the client continues with the certificate verification according to RFC 5280, otherwise it aborts the handshake with a fatal alert.

##### Certificate Expiry  

For the use of certificates devices need to be equipped with a real-time clock and need to be provisioned with a reference time since the certificate expiry field needs to be checked. 

If the Bootstrap-Server certificate has an expiration time then out-of-band mechanisms may be used to recover from an expired certificate. The LwM2M specification does not offer a way to recover from an expired Bootstrap-Server certificate. Out-of-band mechanism may include replacing the LwM2M Bootstrap-Server certificate via the firmware update mechanism or using a commissioning tool.

The LwM2M Bootstrap-Server or an authorized LwM2M Server MAY use the Current Time Resource of the LwM2M Device Object to provision a date/time reference value to the LwM2M Client. The LwM2M Client SHOULD verify the freshness of the request for setting the Current Time Resource. The LwM2M Client can use this reference value together with the current real-time clock setting to determine the current time to check the expiry date of a LwM2M Server certificate. The ability to modify the value of the Current Time Resource is security critical and therefore appropriate access control settings need to be applied. 

##### Certificate Revocation 

A LwM2M Client needs to determine when a LwM2M Server certificate has been revoked. Various approaches for dealing with expired LwM2M Server certificates are possible and the most promising candidates are:  

1. The LwM2M Client MAY use the error recovery procedures that will be invoked when a TLS / DTLS client encounters a problem verifying the LwM2M Server certificate during the handshake. 

2. The LwM2M Client MAY use the Online Certificate Status Protocol (OCSP) as part of the OCSP stapling mechanism \[RFC6961\]. This does, however, assume that the LwM2M Client and the LwM2M Server regularly execute the TLS / DTLS handshake, which may depend on the deployment. Note that the OCSP stapling comes with an extra cost of conveying the OCSP information inside the TLS / DTLS handshake. 

3. The server-initiated bootstrapping procedure is used by a LwM2M infrastructure operator to trigger a LwM2M Client to re-run the bootstrapping exchange to obtain new credentials for use with LwM2M Servers. The server-initiated bootstrapping procedure may not be usable by the LwM2M Server that has gotten its certificate revoked. 

If the Bootstrap-Server certificate has been revoked then out-of-band mechanisms may be used to replace the revoked certificate. The LwM2M specification does not offer a way to recover from a revoked Bootstrap-Server certificate. Out-of-band mechanism may include replacing the LwM2M Bootstrap-Server certificate via the firmware update mechanism or using a commissioning tool.

##### Certificate Usage Field 

The Certificate Usage Resource in the LwM2M Security Object specifies the semantics 
of the certificate or raw public key stored in the Server Public Key Resource, which 
is used to match the certificate presented in the TLS/DTLS handshake. 
RFC 6698 defines for types, which are reused in this specification: 

 - 0: Certificate usage 0 ("CA constraint")

      This mode is used to specify a CA certificate, or
      the public key of such a certificate, that MUST be found in any of
      the PKIX certification paths for the end entity certificate given
      by the server in TLS or DTLS.  This certificate usage is sometimes
      referred to as "CA constraint" because it limits which CA can be
      used to issue certificates for a given service on a host.  The
      presented certificate MUST pass PKIX certification path
      validation, and a CA certificate that matches the Server Public Key
      Resource content MUST
      be included as part of a valid certification path.  Because this
      certificate usage allows both trust anchors and CA certificates,
      the certificate might or might not have the basicConstraints
      extension present.

 - 1: Certificate usage 1 ("service certificate constraint")

      This mode is used to specify an end entity
      certificate, or the public key of such a certificate, that MUST be
      matched with the end entity certificate given by the server in
      TLS.  This certificate usage is sometimes referred to as "service
      certificate constraint" because it limits which end entity
      certificate can be used by a given service on a host.  The target
      certificate MUST pass PKIX certification path validation and MUST
      match the Server Public Key Resource content.

 - 2: Certificate usage 2 ("trust anchor assertion")

      This mode is used to specify a certificate, or the
      public key of such a certificate, that MUST be used as the trust
      anchor when validating the end entity certificate given by the
      server in TLS.  This certificate usage is sometimes referred to as
      "trust anchor assertion" and allows a domain name administrator to
      specify a new trust anchor -- for example, if the domain issues
      its own certificates under its own CA that is not expected to be
      in the end users' collection of trust anchors.  The target
      certificate MUST pass PKIX certification path validation, with any
      certificate matching the Server Public Key
      Resource content considered to be a trust
      anchor for this certification path validation.

 - 3: Certificate usage 3 ("domain-issued certificate")

      This mode is used to specify a certificate, or the
      public key of such a certificate, that MUST match the end entity
      certificate given by the server in TLS.  This certificate usage is
      sometimes referred to as "domain-issued certificate" because it
      allows for a domain name administrator to issue certificates for a
      domain without involving a third-party CA.  The target certificate
      MUST match the Server Public Key
      Resource content.  The difference between certificate
      usage 1 and certificate usage 3 is that certificate usage 1
      requires that the certificate pass PKIX validation, but PKIX
      validation is not tested for certificate usage 3.

#### Error Handling

The TLS / DTLS specifications offer ways to report error situations via the Alert protocol. These errors may have multiple reasons and the process to recover from them is essential for a stable LwM2M implementation. Nevertheless recovering from these errors may be challenging since many IoT devices are unattended and a failure in the establishment of a security association may result in a non-functioning device. Since a LwM2M Bootstrap-Server supervises the LwM2M Client it can support with the recovery of certain types of errors, i.e., the LwM2M Client is asked to fall back to the bootstrap mode in order to obtain new credentials. 

This section provides default recommended error handling. LwM2M Server Object configuration provides additional capabilities via optional resources, such as the Bootstrap on Registration Failure Resource, to define the behavior in error conditions. There is no such fallback mechanism defined for the interaction between the LwM2M Client and the LwM2M Bootstrap-Server.

<table>
    <caption>TLS/DTLS Error Handling Recommendations</caption>
        <thead>
            <tr>
                <th>Value</th>
                <th>Alert Description</th>
                <th>RFC</th>
                <th>Recommendation</th>
            </tr>
        </thead>
    <tbody>
        <tr>
            <td>0</td>
            <td>close_notify</td>
            <td>RFC 5246</td>
            <td>Retry</td>
        </tr>
        <tr>
            <td>10</td>
            <td>unexpected_message</td>
            <td>RFC 5246</td>
            <td>Retry</td>
        </tr>
        <tr>
            <td>20</td>
            <td>bad_record_mac</td>
            <td>RFC 5246</td>
            <td>Retry</td>
        </tr>
        <tr>
            <td>21</td>
            <td>decryption_failed</td>
            <td>RFC 5246</td>
            <td>Retry</td>
        </tr>
        <tr>
            <td>22</td>
            <td>record_overflow</td>
            <td>RFC 5246</td>
            <td>Retry</td>
        </tr>
        <tr>
            <td>30</td>
            <td>decompression_failure</td>
            <td>RFC 5246</td>
            <td>Ignore*</td>
        </tr>
        <tr>
            <td>40</td>
            <td>handshake_failure</td>
            <td>RFC 5246</td>
            <td>Retry</td>
        </tr>
        <tr>
            <td>41</td>
            <td>no_certificate_RESERVED</td>
            <td>RFC 5246</td>
            <td>Ignore*</td>
        </tr>
        <tr>
            <td>42</td>
            <td>bad_certificate</td>
            <td>RFC 5246</td>
            <td>Fail</td>
        </tr>
        <tr>
            <td>43</td>
            <td>unsupported_certificate</td>
            <td>RFC 5246</td>
            <td>Fail</td>
        </tr>
        <tr>
            <td>44</td>
            <td>certificate_revoked</td>
            <td>RFC 5246</td>
            <td>Fail</td>
        </tr>
        <tr>
            <td>45</td>
            <td>certificate_expired</td>
            <td>RFC 5246</td>
            <td>Fail</td>
        </tr>
        <tr>
            <td>46</td>
            <td>certificate_unknown</td>
            <td>RFC 5246</td>
            <td>Fail</td>
        </tr>
        <tr>
            <td>47</td>
            <td>illegal_parameter</td>
            <td>RFC 5246</td>
            <td>Retry</td>
        </tr>
        <tr>
            <td>48</td>
            <td>unknown_ca</td>
            <td>RFC 5246</td>
            <td>Fail</td>
        </tr>
        <tr>
            <td>49</td>
            <td>access_denied</td>
            <td>RFC 5246</td>
            <td>Fail</td>
        </tr>
        <tr>
            <td>50</td>
            <td>decode_error</td>
            <td>RFC 5246</td>
            <td>Retry</td>
        </tr>
        <tr>
            <td>51</td>
            <td>decrypt_error</td>
            <td>RFC 5246</td>
            <td>Retry</td>
        </tr>
        <tr>
            <td>60</td>
            <td>export_restriction_RESERVED</td>
            <td>RFC 5246</td>
            <td>Ignore*</td>
        </tr>
        <tr>
            <td>70</td>
            <td>protocol_version</td>
            <td>RFC 5246</td>
            <td>Retry</td>
        </tr>
        <tr>
            <td>71</td>
            <td>insufficient_security</td>
            <td>RFC 5246</td>
            <td>Retry</td>
        </tr>
        <tr>
            <td>86</td>
            <td>internal_error</td>
            <td>RFC 7507</td>
            <td>Retry</td>
        </tr>
        <tr>
            <td>90</td>
            <td>user_canceled</td>
            <td>RFC 5246</td>
            <td>Retry</td>
        </tr>
        <tr>
            <td>100</td>
            <td>no_renegotiation</td>
            <td>RFC 5246</td>
            <td>Retry**</td>
        </tr>
        <tr>
            <td>110</td>
            <td>unsupported_extension</td>
            <td>RFC 5246</td>
            <td>Ignore.</td>
        </tr>
        <tr>
            <td>111</td>
            <td>certificate_unobtainable</td>
            <td>RFC 6066</td>
            <td>Retry***</td>
        </tr>
        <tr>
            <td>112</td>
            <td>unrecognized_name</td>
            <td>RFC 6066</td>
            <td>Fail***</td>
        </tr>
        <tr>
            <td>113</td>
            <td>bad_certificate_status_response</td>
            <td>RFC 6066</td>
            <td>Fail***</td>
        </tr>
        <tr>
            <td>114</td>
            <td>bad_certificate_hash_value</td>
            <td>RFC 6066</td>
            <td>Fail***</td>
        </tr>
        <tr>
            <td>115</td>
            <td>unknown_psk_identity</td>
            <td>RFC 4279</td>
            <td>Fail</td>
        </tr>
    </tbody>
</table>

Legend: 
 - (*): This error should never occur in a proper TLS / DTLS implementation. 
 - (**): This error should not occur since an implementation compliant to RFC does not use the renegotiation feature. 
 - (***): This error is only applicable when used with the appropriate extension. 

The 'fail' recommendation in the table above indicates that the LwM2M Client MUST consider the handshake exchange failed and that it cannot by itself recover from the error situation. The 'retry' recommendation in the table above indicates that the LwM2M Client MUST consider the handshake failed but that the error situation MAY be transient and retrying the handshake protocol run may result in a successful completion. In both cases the LwM2M Client MUST follow the error handling procedures defined by resources in the LwM2M Server Object, if present.  

An implementation may set a limit to the number of attempts to re-establish a TLS/DTLS connection and may decide to fall back to bootstrapping when a certain threshold is exceeded. Implementations should also use an exponential back-off between retransmissions. The number of retries and the retransmission timers MAY be set by resources in the LwM2M Server resources.

Note: Per RFC 6347 DTLS implementations SHOULD silently discard records with bad MACs or that are otherwise invalid. 

###  Lower- or Higher-Layer Security with the "NoSec" mode

The LwM2M protocol MUST NOT be deployed without appropriate security. Because LwM2M may be deployed with security outside the scope of this specification (e.g. at lower layers), this may mean that neither transport-layer security (TLS/DTLS), nor application-layer security (OSCORE) is applied. In case communication security is not used at the LwM2M layer ("NoSec" mode), the deployment MUST use alternative security mechanisms.

The LwM2M Server MUST compare the endpoint client name identifier used during the Register and the Bootstrap-Request message with the identifier used for network access authentication (typically used to setup link-layer security procedures).

### SMS Channel Security

This section defines the security modes for the transport of CoAP over SMS.

Channel security for \[CoAP\] has been defined for the UDP transport and is based on DTLS \[RFC6347\].

LwM2M Clients only supporting the SMS binding MAY support the NoSec mode when the SMS Channel is only used for debugging purposes otherwise they MUST support SMS Secured Mode. In any security mode except for debugging purposes, when an SMS message is received from an MSISDN that is not recorded in the LwM2M Server SMS Number resource of the LwM2M Server Access Security, the SMS message MUST be silently ignored.

LwM2M Clients supporting UDP and SMS bindings, when the SMS binding is only used for triggering, MUST support the adequate mechanism for securing CoAP over UDP. Those clients MAY use any SMS security mode. In particular SMS NoSec mode can be used for SMS triggering since all other communication will be secured by UDP channel security. Note: Using SMS NoSec for SMS triggering could introduce "Denial of Service" (DoS). 

The "Secret Key" and "Public Key or Identity" Resources are used to configure the keying material that a Client uses with a particular LwM2M Server. The SMS key parameters are stored in the order KIc, KID, SPI, TAR (KIc is byte 0). Ordering of bits within bytes SHALL follow ETSI TS 102 221, Section 3.4 "Coding Conventions" (b8 MSB, b1 LSB).

#### SMS "NoSec" Mode

It is RECOMMENDED to always use LwM2M with one of the security mechanisms described in this specification. However, there are few scenarios and use cases where security is provided by lower layers. For example, LwM2M devices in a controlled environment behind a gateway, or, tests focusing first on other functions before performing end-to-end tests including security.

The use of SMS "NoSec" Mode while the Resource "OSCORE Security Mode" is present indicates that the SMS Channel is protected with OSCORE.

This security profile is also useful to support SMS triggering when all other exchanges utilize CoAP over TLS or CoAP over DTLS. 

#### SMS Secured Mode

The SMS Secured mode specified in this section MUST be supported when the SMS binding is used.

A LwM2M Client which uses the SMS binding MUST either be directly provisioned for use with a target LwM2M Server (Factory Bootstrap or Bootstrap from Smartcard) or else be able to bootstrap via the UDP binding.

The end-point for the SMS channel (delivery of mobile terminated SMS, and sending of mobile originated SMS) MAY be either on the Smartcard or on the Device. When the LwM2M Client device doesn’t support a Smartcard, the end-point is on the LwM2M Client device.

A LwM2M Client, Server or Bootstrap-Server supporting SMS binding MUST discard SMS messages which are not correctly protected using the expected parameters stored in the "SMS Binding Key Parameters" Resource and the expected keys stored in the "SMS Binding Secret Keys" Resource, and MUST NOT respond with an error message secured using the correct parameters and keys.

##### Device End-Point

The Secured Packet Structure is based on \[3GPP TS 31 115\] / \[ETSI TS 102 225\] which was originally designed for securing packet structures for UICC based applications. However, for LwM2M it is suitable for securing the SMS payload exchanged between client and server. Usage of Secured Packet Structure Packet mode in LwM2M device needs evolution towards the introduction of a secure environment. The intention is to evolve the specifications in the next LwM2M release.

If the SMS channel end-point is on the Device, the Channel security for \[CoAP\] is based on the Datagram Transport Layer Security (DTLS) \[RFC6347\]. For that reason the text in Section [TLS/DTLS-based Security]() concerning the DTLS binding to CoAP is also applicable to this section.

Appendix A of \[RFC7925\] describes how to bind CoAP/DTLS message to the SMS channel and specifies the restrictions on DTLS for fitting the SMS channel specific functioning and narrow bandwidth.

###### Header Definitions (for one SMS)

a) SMS Frame for basic Request/Response Interaction message (no Token field required)

<figure>
      <img src="images/header_definition_SMS_no_token.svg" alt="SMS header definition (no token) ">
      <figcaption>SMS header definition (no token) </figcaption>
</figure>

Model calculation using these header definitions,

-   Overall TPDU : 140 bytes

    -   DTLS requires 29 bytes: 13 bytes header according to (RFC 6347 and Appendix B of \[RFC7925\] + 8 bytes for the explicit nonce and 8 bytes for the integrity check value when an AES-128-CCM-8 ciphersuite is used. This ciphersuite uses a short integrity check value.

    -   CoAP header of variable length with at least 4 bytes \[CoAP\]

    -   Available bytes for the effective LwM2M payload from one SMS: 107 bytes

b) SMS Frame for messages of the Information Reporting Interface (Token field required)

<figure>
      <img src="images/header_definition_SMS_token.svg" alt="SMS header definition">
      <figcaption>SMS header definition</figcaption>
</figure>

Model calculation using these header definitions,

-   DTLS takes 29 bytes: 13 bytes (reference, RFC 6347) of header + 16 bytes of integrity check for CoAP in DTLS \[RFC 6655\] . Cipher suite mandated by CoAP (AES-128)

-   CoAP header 4+8 \[CoAP\] (Token field required)

-   Available bytes for the effective LwM2M Payload from one SMS: 99 bytes

###### Smartcard End-Point

If the SMS channel end-point is on the smart card, a CoAP message as defined in \[CoAP\] MUST be encapsulated in \[3GPP 31.115\] Secured Packets, in implementing - for SMS Point to Point (SMS\_PP) - the general \[ETSI 102 225\] specification for UICC based applications.

The following settings MUST be applied:

Class 2 SMS as specified in \[3GPP TS 23.038\]. The \[3GPP TS 23.040\] SMS header MUST be defined as below:

-   TP-PID : 111111 (USIM Data Download) as specified in \[3GPP TS 23.040\]

-   TP-OA : the TP-OA (originating address as defined in \[3GPP 23.040\] of an incoming command packet (e.g. CoAP request) MUST be re-used as the TP-DA of the outgoing packet (e.g. CoAP response)

**Secure SMS Transfer to UICC**

A SMS Secured Packet encapsulating a CoAP request received by the LwM2M device, MUST be – according to \[ETSI TS 102 225\]/\[3GPP TS 31.115\] - addressed to the LwM2M UICC Application in the Smartcard where it will be decrypted, aggregated if needed, and checked for integrity.

If decryption and integrity verification succeed, the message contained in the SMS MUST be provided to the LwM2M Client.

If decryption or integrity verification failed, SMS MUST be discarded.

The mechanism for providing the decrypted CoAP Request to the LwM2M Client relies on basic GET\_DATA commands of \[GP SCP03\] .This data MUST follow the format as below:

> data\_rcv \_ ::= &lt;address&gt; &lt;coap\_msg&gt;
>
> address ::= TP\_OA ; originated address
>
> coap\_msg ::= CoAP\_TAG &lt;coap\_request\_length&gt; &lt;coap\_request&gt;
>
> coap\_request\_length ::= 16BITS\_VALUE
>
> coap\_request ::= CoAP message payload

NOTE: In current LwM2M release, the way the LwM2M Client Application is triggered for retrieving the available message from the Smartcard is device specific: i.e. a middle class LwM2M Device implementing \[ETSI TS 102 223\] ToolKit with class "e" and "k" support could be automatically triggered by Toolkit mechanisms, whereas a simpler LwM2M device could rely on a polling mechanisms on Smartcard for fetching data when available.

**Secured SMS Transfer to LwM2M Server**

For sending a CoAP message to the LwM2M Server, the LwM2M Client prepares a data containing the right TP-DA to use, concatenated with the CoAP message and MUST provide that data to the LwM2M UICC Application in using the \[GP SCP03\] STORE-DATA command.

According to \[ETSI TS 102 225\] / \[3GPP TS 31.115\] the Smartcard will be in charge to prepare (encryption / concatenation) the CoAP message before sending it as a SMS Secure Packet (\[ETSI TS 102 223\] SEND\_SMS command).

The SMS Secured Packet MUST be formatted as Secured Data specified in Section [SMS Secured Packet Binding for CoAP messages]().

The Secure Channel as specified in Appendix H of [LwM2M-CORE] SHOULD be used to provide the prepared data to the Smartcard.

###### SMS Secured Packet Binding for CoAP messages

In SMS Secured Packet Structure mode, a CoAP message as defined in \[CoAP\] MUST be encapsulated in \[3GPP 31.115\] Secured Packets, in implementing - for SMS Point to Point (SMS\_PP) - the general \[ETSI 102 225\] specification for UICC based applications.

-   The "Command Packet" command specified in \[3GPP 31.115\] /\[ETSI TS 102 225\] MUST be used for both CoAP Request and Response message

-   The Structure of the Command Packet contained in the Short Message MUST follow \[3GPP 31.115\] specification

-   SPI MUST be set as follow (see coding of SPI in \[ETSI TS 102 225\], Section 5.2.1):

    -   use of cryptographic checksum

    -   use of ciphering

        -   The ciphering and crypto graphic checksum MUST use either AES or Triple DES

        -   Single DES MUST NOT be used

        -   AES SHOULD be used

        -   When Triple DES is used , then it MUST be used in outer CBC mode and 3 different keys MUST be used

        -   When AES is used it MUST be used with CBC mode for ciphering (see coding of KIc in \[ETSI TS 102 225\], Section 5.2.2) and in CMAC mode for integrity (see coding of KID in \[ETSI TS 102 225\], Section 5.2.3).

    -   process if and only if counter value is higher than the value in the RE

    -   PoR depends on LwM2M Server Policy

-   TAR MUST be set to ‘B2 02 03’ value for the LwM2M UICC Application as registered in \[ETSI TS 101 220\] Appendix D

-   Secured Data : contains the Secured Application Message which MUST be coded as a BER-TLV, the Tag (TBD : e.g. 0x05) will indicate the type (e.g. CoAP type) of that message

### OSCORE-based Security

#### OSCORE Overview

Object Security for Constrained RESTful Environments [OSCORE] is an application layer security protocol protecting CoAP message exchanges. OSCORE is applicable to protocol messages which can be mapped to CoAP or a subset of CoAP, including HTTP and LwM2M.

The use of OSCORE with LwM2M is optional. It MAY be used for protecting the communication between a LwM2M Client and the LwM2M Bootstrap-Server. It MAY be used between a LwM2M Client and a LwM2M Server.

OSCORE MAY also be used between LwM2M endpoint and non-LwM2M endpoint, e.g. between an Application Server and a LwM2M Client via a LwM2M server. Both the LwM2M endpoint and non-LwM2M endpoint MUST implement OSCORE and be provisioned with an OSCORE Security Context as defined in [OSCORE]. The bootstrapping of the non-LwM2M endpoint is out of scope for this specification. The intermediate LwM2M node needs to map the messages from non-LwM2M nodes to LwM2M operations and responses. That mapping is also out of scope for this specification.

The use of OSCORE in CoAP is indicated with the OSCORE Option present in the CoAP message [OSCORE]. It operates by transforming (encrypting etc.) an unprotected CoAP message into a protected CoAP message using the compact secure message format COSE \[RFC8152\]. The protected CoAP message is sent instead of the unprotected message, and is verified and transformed back to the original message at the receiving end.

OSCORE protects the Request/Response sublayer of CoAP, but leaves the binding to the transport layer protocol unprotected. As a consequence, OSCORE does not depend on the underlying layer and can therefore be applied to CoAP over UDP, TCP, SMS as well as Non-IP, directly over IEEE 802.15.4 or 3GPP CIoT (see [LwM2M over 3GPP CIoT - NB-IoT and LTE-M]()). OSCORE is invariant under proxy operations translating between different transport layer protocols, performing application layer forwarding (e.g. CoAP proxies), address translations, etc. and thus enables end-to-end security through these kinds of intermediate nodes.

Not only is OSCORE applicable to CoAP-mappable messages, but the protected messages can also be transported directly in CoAP-mappable protocols. For example, with the HTTP header field "OSCORE" [OSCORE], OSCORE protected messages can be transported in HTTP, providing end-to-end security for CoAP-mappable HTTP. OSCORE also supports proxies translating between HTTP and CoAP as defined in \[RFC8075\], allowing a mix of HTTP and CoAP along the end-to-end path, which is useful, e.g. for firewall traversals on the Internet. 

OSCORE is based on a shared symmetric key and provides security context derivation, encryption, integrity and replay protection, and a secure binding of application layer responses to requests. OSCORE enables mutual authentication between LwM2M Client and LwM2M Bootstrap-Server, and between LwM2M Client and LwM2M Server.

OSCORE is independent of security protocols at other layers, such as DTLS/TLS. If OSCORE is applied with a "Security Mode" Resource value 0-2 (corresponding to DTLS with mode "Pre-Shared Key"/"Raw Public Key"/"Certificate") the message will be protected by both OSCORE and DTLS. When OSCORE is applied with "Security Mode" Resource value 3 (corresponding to DTLS with mode "NoSec"), the message is protected by OSCORE but not DTLS.

OSCORE can be used to secure multicast requests and responses, for example providing low latency in building automation scenarios (lighting, emergency etc.) and for efficient bandwidth usage during firmware updates.

#### Cryptographic Algorithms

OSCORE is defined for use with an Authenticated Encryption with Additional Data (AEAD) algorithm and a HMAC-based Key Derivation Function (HKDF) \[RFC5869\]. OSCORE mandates the implementation of the AEAD algorithm AES-CCM-16-64-128 (also known as CCM*) as defined in \[RFC8152\], and specifies the use of HMAC with SHA-256 as default. LwM2M Clients and LwM2M Servers MAY support additional algorithms that conform to the state-of-the-art security requirements.

#### Bootstrapping

The OSCORE related Resources in the LwM2M OSCORE Object are used analogously to DTLS/TLS when applied with pre-shared keys (see [Bootstrapping]()). The OSCORE related Resources in the LwM2M OSCORE Object are used for:

1.  providing OSCORE communication security in "Client Registration", "Device Management & Service Enablement", and "Information Reporting" Interfaces if the LwM2M Security Object Instance relates to a LwM2M Server;

2.  providing OSCORE communication security in the "Bootstrap" Interface if the LwM2M Security Object instance relates to a LwM2M Bootstrap-Server;

3.  providing OSCORE communication security with a firmware repository server when the LwM2M Client receives a URI in the Package URI of the Firmware Update object.

In case of Bootstrap from Smartcard, a secure channel between the Smartcard and the LwM2M Client SHOULD be established, as described in Appendix G of [LwM2M-CORE] and defined in \[GLOBALPLATFORM 3\], \[GP SCP03\]. Using Smartcard with the pre-shared key needs no pre-existing trust relationship between LwM2M Server(s) and LwM2M Client(s). The pre-established trust relationship exists between the LwM2M Server(s) and the SmartCard(s).

If OSCORE is used, then LwM2M Clients MUST either be provisioned for use with a LwM2M Server (manufacturer pre-configuration bootstrap mode) or else be provisioned for use with a LwM2M Bootstrap-Server. Any LwM2M Client, which supports client or server initiated bootstrap mode, MUST support bootstrapping with a strong (high-entropy) pre-shared key. The ciphersuites MUST NOT be used with a low-entropy pre-shared key or a password as pre-shared key. The LwM2M Client MUST be provisioned with a pre-shared key that is unique to a device and to the security protocol. For full interoperability a LwM2M Bootstrap-Server MUST support bootstrapping with pre-shared key.

The OSCORE security context is derived from a small set of input parameters (see Section 3.2 of [OSCORE]). With use of the OSCORE input parameters, the security contexts for sending and receiving OSCORE messages are derived as defined in [OSCORE]. The conditions for reuse of input parameters (see Section 3.3 of [OSCORE]) MUST be complied with. An implementation SHALL follow the procedure described in Appendix B2 of [OSCORE] when the OSCORE security context is used for the first time.

New OSCORE input parameters MAY be provisioned for use between endpoints having an existing OSCORE security context requiring the endpoints to derive new contexts. The Echo option MUST be used by the receiving endpoint when the OSCORE security context is used for the first time (e.g. Bootstrap-Request or Register) and SHOULD be used when the requested operation (e.g. Device Management Write) requires freshness that cannot be guaranteed by other means.

#### Unbootstrapping

The unbootstrapping procedure described in [Unbootstrapping]() applies also to OSCORE.

#### Endpoint Client Name

The same verification of Endpoint Client Name in the Bootstrap-Request and in the Register messages as described in [Endpoint Client Name]() applies also to OSCORE. However, when using OSCORE, the Endpoint Client Name MAY be authenticated at the application layer, by setting the "OSCORE Sender ID" Resource value (see [OSCORE Related Resources]()) to the Endpoint Client Name.

If the OSCORE Sender ID is not set to Endpoint Client Name, then the LwM2M Server MUST compare the received Endpoint Client Name identifier with the OSCORE Sender ID of the LwM2M Client. This comparison may either be an equality match or may involve a dedicated lookup table to ensure that LwM2M Clients cannot intentionally or due to misconfiguration impersonate other LwM2M Clients. The LwM2M Server MUST respond with a "4.00 Bad Request" to the LwM2M Client if these fields do not match.

#### OSCORE Roles

The OSCORE roles (Client and Server) coincide with the CoAP roles. 

OSCORE specifies that each endpoint has a Sender ID and a Recipient ID. The Sender ID is used when sending a message and recipient ID when receiving a message, independently of whether the endpoint has the role of an OSCORE Client or an OSCORE Server. Hence the Sender ID of the OSCORE Client has the same value as the Recipient ID of the OSCORE Server, and the Recipient ID of the OSCORE Client has the same value as the Sender ID of the OSCORE Server.

#### Credential Types

OSCORE requires that a shared secret key, called "OSCORE Master Secret", is established in the communicating endpoints. In LwM2M version 1.1 and therafter, it is assumed that the Master Secret is pre-shared.

When the Master Secret used with a LwM2M Server is provisioned to the LwM2M Client, the Master Secret is also known to the provisioning party, e.g. the LwM2M Bootstrapping Server. Applications need to ensure that this assumption is compliant with their trust model.

##### OSCORE Related Resources

The use of OSCORE between LwM2M Client and LwM2M Server, or between LwM2M Client and LwM2M Bootstrap-Server is indicated with the optional "OSCORE Security Mode" Resource in the Security Object description.

-   If present, the "OSCORE Security Mode" Resource MUST contain the link to an Instance of the LwM2M OSCORE Object.

The presence of the "OSCORE Security Mode" Resource in a Security Object Instance requires the following resources of the LwM2M OSCORE Object to be populated:

-   The "OSCORE Master Secret" Resource MUST be used to store the Master Secret, as defined in [OSCORE]. The OSCORE Client and OSCORE Server use the same Master Secret. Recommendations for generating random keys are provided in \[RFC4086\].

- The "OSCORE Sender ID" Resource MUST be used to store the Sender ID of the LwM2M Client as an UTF-8 string, as defined in [OSCORE].

- The "OSCORE Recipient ID" Resource MUST be used to store the Recipient ID of the LwM2M Client as an UTF-8 string, as defined in [OSCORE].

Additionally, the following resources of the LwM2M OSCORE Object MAY be populated:

-   The "OSCORE AEAD Algorithm" Resource MUST be used to store the AEAD Algorithm, as defined in [OSCORE].

-  The "OSCORE HMAC Algorithm" Resource MUST be used to store the HMAC Algorithm used with HKDF, as defined in [OSCORE].

-   The "OSCORE Master Salt" Resource MUST be used to store the Master Salt, as defined in [OSCORE].


### Identification of Blockwise Requests 

The specification of Blockwise \[CoAP\_Blockwise\] is vulnerable to interchange of blocks between different requests to the same resource \[CoAP_ERT\]. The attack may be performed when the replay window size of the the security protocol is greater than 1 even if the requests are not inverleaved and the attack applies both to DTLS and OSCORE. The attack does not apply when a connection-oriented transport, like CoAP over TCP is used, or when a replay window size of 1 is selected with DTLS. 

A solution is to use the CoAP Request-Tag Option \[CoAP_ERT\] for unique tagging of requests of a certain scope. The Request-Tag is analogous to the CoAP E-Tag Option, but tags requests instead of responses.

LwM2M Clients and LwM2M Servers supporting Blockwise SHOULD implement the CoAP Request-Tag Option.


### Freshness

A LwM2M Client and LwM2M Server MUST be able to verify the freshness of certain LwM2M operations.

LwM2M operations may have freshness requirements that are not possible to guarantee by protecting individual messages such as with DTLS Record Layer or OSCORE. This is accentuated with unreliable transport.
Since datagram transport does not provide reliable or in-order delivery of data, DTLS and OSCORE preserves this property. Although duplicate messages are rejected through the use of anti-replay mechanisms, unordered delivery is still allowed, e.g. using a sliding receive window. As a consequence maliciously delayed message are accepted as long as they fall within the window. 

For example, a Write operation, maliciously blocked from reaching the LwM2M Client at one time, may under these circumstances be successfully injected at a later time, potentially overwriting a more recent Write operation.

Operations protected by a security protocol with keys derived from a TLS/DTLS handshake are at least as fresh as the handshake. However, frequent use of the handshake protocols may be prohibitive in constrained environments. In order to avoid unnecessary processing, a more lightweight solution to verify freshness is provided by the CoAP Echo Option \[CoAP\_ERT\], illustrated with the example above: The LwM2M Client, receiving a Write operation of uncertain freshness may respond with an error message containing an Echo option including a random nonce as value. The LwM2M Server receiving the error response to a valid Write operation retransmits the request with the Echo option and value included. The LwM2M Client receiving a request with an Echo option verifies that the nonce corresponds to a recent request, and only in that case performs the operation (for details, see Section 2 of \[CoAP_ERT\]).

Applications need to understand the freshness requirements of the operations both in LwM2M Client and LwM2M server. The LwM2M implementation SHOULD enable timely freshness verifications to be performed without unnecessary overhead. For interoperability both LwM2M Client and LwM2M Server SHOULD implement the CoAP Echo option.

The attack does not apply when a connection-oriented transport, like CoAP over TCP is used, or when a replay window size of 1 is selected with DTLS. 

## CoAP Transport Binding

This section defines the CoAP transport binding used by LwM2M interfaces.

### Features

The CoAP transport binding utilizes the following features: 

-   CoAP, as defined in \[CoAP\], MUST be supported by the LwM2M Client and the LwM2M Server. 

-  CoAP over TCP/TLS, as defined in \[CoAP-TCP\], SHOULD be used to enable improved firewall and NAT traversal capabilities.

-   CoAP Observe, as defined in \[Observe\], MUST be supported by the LwM2M Client and the LwM2M Server for the Information Reporting interface. Non-Confirmable messages MAY be used by a Client for sending Information Reporting notifications as per \[Observe\].

-   CoAP Blockwise transfer for CoAP \[CoAP\_Blockwise\] MUST be supported by the LwM2M Client when the Firmware Update Object (ID:5) is implemented by the client and MUST be supported by the LwM2M Server. This functionality is motivated by limitations of CoAP, as defined in \[RFC7252\] since CoAP was not designed for transmission of large payloads. Because the CoAP header itself does not contain length information the UDP length header is used instead. The maximum UDP datagram size is limited to ~64 KiB and transmitting data beyond the (path) maximum transmission (MTU) size will additionally lead to inefficiency because of fragmentation at lower layers (IP layer, adaptation layer, and link layer). Blockwise Transfer for CoAP \[CoAP_Blockwise\] was specifically designed to lift this limitation in order to transfer large payloads larger than ~64 KiB via CoAP, such as firmware images. \[CoAP\_Blockwise\] is also beneficial for use with firmware images smaller than 64 KiB since the block-wise transmission allows the server to deliver firmware images in chunks suitable to the MTU and thereby avoiding fragmentation at lower layers. A LwM2M client MAY choose to support block-wise transfer for objects other than the Firmware Update object. This may, for example, be useful with objects that are larger in size, such as the security object which may contain certificates. The specifics of how this functionality is utilized by a LwM2M Server are out of scope for this release of LwM2M.

-  The CoAP OSCORE Option MAY be used to enable OSCORE.

-  The CoAP Request-Tag Option \[CoAP_ERT\] SHOULD be used to detect interchange of blocks between different blockwise requests to the same resource over unreliable transport.

-  The CoAP Echo Option \[CoAP_ERT\] SHOULD be used to enable lightweight freshness verifications.

### Firewall Traversal

For a firewall to support LwM2M using CoAP over UDP and/or CoAP over DTLS it MUST at a minimum be configured to allow outgoing packets to destination port 5683 (for CoAP over UDP), and port 5684 (for CoAP over DTLS), and allow incoming UDP/DTLS packets back to the source address/port of the outgoing UDP packet for a period of at least 240 seconds. 

While Queue Mode is not necessary for firewall configuration it should be noted that LwM2M Clients may enable Queue Mode for example to lower power consumption for a battery powered device. 

For those deployments where changes to firewall configurations are not possible CoAP over TCP/TLS SHOULD be used instead. Firewall rule settings for CoAP over TCP/TLS are similar to those described above for CoAP over UDP/DTLS since the port numbers are equivalent, i.e., port 5683 (for CoAP over TCP), and port 5684 (for CoAP over TLS). 

### NAT Traversal

A NAT exhibits a behavior similar to a stateful packet filtering firewall where incoming packets are only allowed to traverse the NAT after the client behind the NAT initiated an outgoing connection first. The NAT creates a NAT binding based on the outgoing communication attempt, which is re-used for any packets from the device outside the NAT (i.e., typically the server).  

Where NATs are present along the communication path, CoAP over TCP leads to different NAT traversal behavior than CoAP over UDP. NATs often calculate expiration timers based on the transport layer protocol being used by application protocols.  Many NATs maintain TCP-based NAT bindings for longer periods based on the assumption that a transport layer protocol, such as TCP, offers additional information about the session lifecycle.  UDP, on the other hand, does not provide such information to a NAT and timeouts tend to be much shorter, as described in \[CoAP-TCP\].
      
Queue Mode may help with NAT traversal by allowing a LwM2M Server to send messages to the LwM2M Client only after the LwM2M Client has initiated the communication to the LwM2M server first. This ensures that NAT bindings exist that allow incoming packets to the LwM2M Client. 

### URI Identifier & Operation Mapping

Although CoAP supports a URI in requests, it is not used in the same way as in HTTP. The URI in CoAP is broken down into binary parts, minimizing overhead and complexity. In LwM2M only path segment and query string URI components are needed. The URI path is used to simply identify the interface, Object Instance or Resource that the request is for, and is encoded in Uri-Path options. The LwM2M Registration interface also makes use of query string parameters to pass on meta-data with the request separately from the payload. Each query parameter is encoded in a Uri-Query Option. Likewise, the LwM2M operations for each interface are mapped to CoAP Methods.

#### Alternate Path

By default, the LwM2M Objects are located under the root path. However, devices might be hosting other CoAP Resources on an endpoint, and there may be the need to place LwM2M Objects under an alternate path.

When registering, or updating its registration, a LwM2M Client MAY include an OMA LwM2M link in addition to the Object links in the registration payload. The link is identified by the RFC 6690 [RFC6690] Resource Type parameter "oma.lwm2m".

This link MUST NOT contain numerical URI segment.

For instance, the Example Client from Appendix F of [LwM2M-CORE] may place Objects under the "/lwm2m" path. The registration payload would be as follows:

&lt;/lwm2m&gt;;rt="oma.lwm2m", &lt;/lwm2m /1/0&gt;,&lt;/lwm2m /1/1&gt;,&lt;/lwm2m /2/0&gt;,&lt;/lwm2m /2/1&gt;,&lt;/lwm2m /2/2&gt;,&lt;/lwm2m /2/3&gt;,&lt;/lwm2m /2/4&gt;,&lt;/lwm2m /3/0&gt;,&lt;/lwm2m /4/0&gt;,&lt;/lwm2m /5&gt;

When using the Device Management & Service Enablement Interface and the Information Reporting Interface, the LwM2M Server MUST prepend the OMA LwM2M link to the path in the CoAP messages. Example: GET /lwm2m/3/0/0.

When using the Bootstrap Interface, the LwM2M Bootstrap-Server MUST use CoAP paths only in the form /{Object ID}/{Object Instance ID}/{Resource ID}. It is the responsibility of the LwM2M Client to map these paths to its alternate path.

The Resource Type value "oma.lwm2m" is part of IANA registry.

#### Bootstrap Interface

The Bootstrap Interface is used to optionally configure a LwM2M Client so that it can successfully register with a LwM2M Server. The client bootstrap operation is performed by sending a CoAP request to the LwM2M Bootstrap-Server at the path specified in [Table Operation to Method and URI Mapping (Bootstrap Interface)]() including the Endpoint Client Name as a query string parameter. When the bootstrap operation is terminated the Bootstrap-Server MUST send a "Bootstrap-Finish" operation.

The Client Bootstrap operation is initiated by the LwM2M Client itself. In addition, this operation can be requested by an authorized LwM2M Server executing the "Bootstrap-Request Trigger" Resource of a Server Object Instance, or even by a proprietary mechanism (e.g. based on SMS).
Note: the execution of a "Bootstrap-Request Trigger" Resource by a LwM2M Server in a LwM2M Client is performed through an already established registration and is therefore covered by the access rights mechanisms.   

During the Bootstrap Phase, the Client MAY ignore requests and flush all pending responses not related to the Bootstrap sequence.

In "Client Initiated Bootstrap" mode, if the Bootstrap-Server receives the "Bootstrap-Request" operation, the Bootstrap-Server can perform the "Bootstrap-Write", "Bootstrap-Discover", "Bootstrap-Read" and the "Bootstrap-Delete" operations. The "Bootstrap-Delete" operation targets an Object or an Object Instance, the "Bootstrap-Discover" operation targets an Object, a "Bootstrap-Write" operation targets Object, Object Instance or a Resource, while the "Bootstrap-Read" operation is limited to read a single Instance or all Instances of the Access Control Object (an error is returned otherwise). The "Bootstrap-Write", "Bootstrap-Discover", "Bootstrap-Read" and the "Bootstrap-Delete" operations can be sent multiple times. Only in Bootstrap Interface, the "Bootstrap-Delete" operation MAY target to "/" URI to delete all the existing Object Instances - except LwM2M Bootstrap-Server Account - in the LwM2M Client, for initialization purpose before LwM2M Bootstrap-Server sends "Bootstrap-Write" operation(s) to the LwM2M Client. Different from "Write" operation in Device Management and Service Enablement interface, the LwM2M Client MUST write the value included in the payload regardless of an existence of the targeting Object Instance(s) or Resource and access rights.

In "Client Initiated Bootstrap" mode, if the Bootstrap-Server receives the "Bootstrap-Pack-Request" operation, the Bootstrap-Server may respond with a "Bootstrap-Pack" to the LwM2M Client. The "Bootstrap-Pack-Request" operation is performed by sending a CoAP GET request to the LwM2M Bootstrap Server at the "/bspack" path.

Only in Bootstrap Interface, the "Bootstrap-Discover" operation MAY target to "/" URI to discover all Objects and Object Instances supported in the Device.

The Bootstrap-Server MUST send a "Bootstrap-Finish" operation after it has sent all object instances/resources. "Bootstrap-Finish" operation is not needed if the "Bootstrap-Pack" is used for bootstrapping. The "Bootstrap-Finish" operation is mapped to a CoAP POST to "/bs" location path with an empty payload.

If OSCORE is used to secure Client Bootstrap then the Bootstrap-Server receiving the "Bootstrap-Request" or "Bootstrap-Pack-Request" operation MUST use the Echo Option \[CoAP_ERT\].

<table>
    <caption>Operation to Method and URI Mapping (Bootstrap Interface)</caption>
        <thead>
            <tr>
                <th>Operation</th>
                <th>CoAP Method</th>
                <th>URI</th>
            </tr>
        </thead>
    <tbody>
        <tr>
            <td>Bootstrap-Request</td>
            <td>POST</td>
            <td>/bs?ep={Endpoint Client Name}&pct={Preferred Content Format}</td>
        </tr>
        <tr>
            <td>Bootstrap-Read</td>
            <td>GET Accept:Content Format ID: TLV, LwM2M CBOR, SenML CBOR or SenML JSON</td>
            <td>/{Object ID} in LwM2M 1.1 and therafter, Object ID MUST be '2' (Access Control Object)</td>
        </tr>
        <tr>
            <td>Bootstrap-Write</td>
            <td>PUT</td>
            <td>/{Object ID}<br>
                /{Object ID}/{Object Instance ID}<br>
                /{Object ID}/{Object Instance ID}/{Resource ID}</td>
        </tr>
        <tr>
            <td>Bootstrap-Delete</td>
            <td>DELETE</td>
            <td>/{Object ID}/{Object Instance ID}</td>
        </tr>
        <tr>
            <td>Bootstrap-Discover</td>
            <td>GET Accept:
                application/link-format</td>
            <td>/{Object ID}</td>
        </tr>
        <tr>
            <td>Bootstrap-Finish</td>
            <td>POST</td>
            <td>/bs</td>
        </tr>
	<tr>
            <td>Bootstrap-Pack-Request</td>
            <td>GET Accept:Content Format ID: SenML CBOR, SenML JSON, or LwM2M CBOR</td>
            <td>/bspack?ep={Endpoint Client Name}</td>
        </tr>
    </tbody>
</table>

Available CoAP response codes to the operations are given in [Table Response Codes: Bootstrap Interface]()

<figure>
   <img src="images/client_initiated_bootstrap_exchange.svg" alt="Example of Client initiated Bootstrap exchange">
   <figcaption>Example of Client initiated Bootstrap exchange</figcaption>
 </figure>

 <figure>
   <img src="images/client_initiated_bootstrap_exchange_with_bootstrap_pack.svg" alt="Example of Client initiated Bootstrap with Bootstrap-Pack exchange">
   <figcaption>Example of Client initiated Bootstrap with Bootstrap-Pack exchange</figcaption>
 </figure>

<figure>
   <img src="images/server_initiated_bootstrap_exchange.svg" alt="Example of Server initiated Bootstrap exchange">
   <figcaption>Example of Server initiated Bootstrap exchange</figcaption>
 </figure>


#### Registration Interface

The registration interface is used by a LwM2M Client to register with a LwM2M Server, identified by the LwM2M Server URI.

Registration is performed by sending a CoAP POST to the LwM2M Server URI /rd, with the LwM2M registration parameters passed as URI query strings as per [Table Registration Parameters to URI Parameter Mapping]() and [Table Operation to Method and URI Mapping (Registration Interface)](). The response includes Location-Path Options, which indicate the path to use for updating or deleting the registration. The LwM2M Server MUST return a location under the /rd path segment.

As the network connectivity may be limited or intermittent, it is advised to make several retries of the Registration if no reply is received from the LwM2M Server before considering the registration as failed.

When a new security context is created (e.g. TLS/DTLS), the Client MUST register again to the LwM2M Server. 

When the LwM2M Client IP address changes in NoSec mode, the Client MUST register again to the LwM2M Server. 

If OSCORE is used to secure the Registration then the Echo Option \[CoAP_ERT\] MUST at least be used with the first operation.

Registration update is performed by sending a CoAP POST to the Location path returned to the LwM2M Client as a result of a successful registration.

De-registration is performed by sending a CoAP DELETE to the Location path returned to the LwM2M Client as a result of a successful registration.

<table>
    <caption>Registration Parameters to URI Query String Mapping</caption>
    <thead>
    <tr>
        <th>LwM2M Registration Parameter</th>
        <th>URI Query String</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td>Endpoint Client Name</td>
        <td>ep</td>
    </tr>
    <tr>
        <td>Lifetime</td>
        <td>lt</td>
    </tr>
    <tr>
        <td>LwM2M Version</td>
        <td>lwm2m</td>
    </tr>
    <tr>
        <td>Binding Mode</td>
        <td>b</td>
    </tr> 
    <tr>
        <td>Queue Mode</td>
        <td>Q</td>
    </tr> 
    <tr>
        <td>SMS Number</td>
        <td>sms</td>
    </tr> 
    <tr>
        <td>Profile ID</td>
        <td>pid</td>
    </tr>
  </tbody>
</table>
<table>
    <caption>Operation to Method and URI Mapping (Registration Interface)</caption>
    <thead>
        <tr>
            <th>Operation</th>
            <th>CoAP Method</th>
            <th>URI</th>
        </tr>
    </thead>
    <tbody>
    <tr>
        <td><strong>Register</strong></td>
        <td>POST</td>
        <td>/rd?ep={Endpoint Client Name}&amp;lt={Lifetime}&amp;lwm2m={version}&amp;b={binding}&amp;Q&amp;sms={MSISDN}&amp;pid={ProfileID}</td>
    </tr>
    <tr>
        <td><strong>Update</strong></td>
        <td>POST</td>
        <td>/{location}?lt={Lifetime}&amp;b={binding}&amp;Q&amp;sms={MSISDN}</td>
    </tr>
    <tr>
        <td><strong>De-register</strong></td>
        <td>DELETE</td>
        <td>/{location}</td>
    </tr>
    </tbody>
</table>


Available CoAP response codes to the operations are given in [Table Response Codes: Client Registration Interface]()

Note: Throughout the present document the format of the MSISDN must be as specified in \[3GPP-TS\_23.003\]. According to this definition "+" is not preceding the country code.

<figure class="text-center">
      <img src="images/register_update_de-register_operation.svg" alt="Example register, update and de-register operation exchanges (shorthand in \[CoAP\] example style, actual messages using CoAP binary headers">
      <figcaption>Example register, update and de-register operation exchanges (shorthand in \[CoAP\] example style, actual messages using CoAP binary headers</figcaption>
</figure>

#### Device Management & Service Enablement Interface

The Device Management & Service Enablement Interface is used to access a Resource, a Resource Instance, an array of Resource Instances, an Object Instance or all the Object Instances of an Object. An Object Instance is identified by the path /{Object ID}/{Object Instance ID}. If Object doesn't support multiple Object Instances, the Object Instance is identified by the path /{Object ID}/0. A Resource is identified by the path /{Object ID}/{Object Instance ID}/{Resource ID}. An Instance of a Multiple-Instance Resource is identified by the path /{Object ID}/{Object Instance ID}/{Resource ID}/{Resource Instance ID}. When the rules expressed in [LwM2M-CORE] and related to the path in the command on this LwM2M Interface, are not respected, the error code 4.05 MUST be returned as response code. 

An Object Instance, a Resource or a Resource Instance are read by sending a CoAP GET to the corresponding path. The response includes the value in the corresponding format according to the specified Content-Format (see [LwM2M-CORE]).The request MAY specify an Accept option containing the preferred Content-Format to receive. When the specified Content-Format is not supported by the LwM2M Client, the request MUST be rejected (error code 4.06 as defined in \[CoAP\]).

Note that the response payload may be empty, for instance when performing a Read operation on an Object with no Object Instance. In this case the response code is still 2.05 Content.

An Object Instance, a Resource or a Resource Instance are written by sending a CoAP PUT request to the corresponding path. The request includes the value to be written in the corresponding Plain Text, Opaque, TLV, LwM2M CBOR, SenML CBOR, or SenML JSON format according to the Content-Format option which MUST be specified \[CoAP\]. In addition, an Object Instance can be written by sending a CoAP POST request; in that case the specified Content-Format MUST be one of the TLV, CBOR, SenML CBOR or SenML JSON formats. CoAP PUT is used for Replace. CoAP POST or CoAP iPatch is used for Partial Update.

The Write request MUST be rejected :
- when the specified Content-Format is not supported by the LwM2M Client (error code 4.15)
- when a requested operation on a Multiple-Instance Resource is not authorized by the LwM2M Client (error code 4.01) as described in [LwM2M-CORE].

A Resource is Executed by sending a CoAP POST to the corresponding path. The definition of the "Execute" operation is described in [LwM2M-CORE]. The "Execute" operation MAY contain a payload, if arguments are required. When a payload in plain text format is used then the ABNF syntax for the arguments described in [LwM2M-CORE] MUST be used.

Note that the behaviour of the "Execute" operation is specified in the Resource description of the Object.

An Object Instance is Created by sending a CoAP POST to the corresponding path. The request includes the value to be written in the corresponding TLV, LwM2M CBOR, SenML CBOR or SenML JSON format according to the Content-Format option which MUST be specified. The rules governing the creation of Resources in the targeted Object Instance are specified in [LwM2M-CORE]. If Object Instance is not listed at the request, the LwM2M Client MUST assign ID of that Object Instance and send back Object Instance ID with "2.01 Created" to the LwM2M Server when Object Instance is Created.

The Create request at object level cannot be distinguished from a Write request at object level as both are translated to a CoAP POST, thus the WRITE operation at object level is not supported for the CoAP binding.

An Object Instance is Deleted by sending a CoAP DELETE to the corresponding path.

When a Resource supports multiple instances, the Resource value is an array of Resource Instances.

&lt;NOTIFICATION&gt; Class Attributes MAY be set by a LwM2M Server using the "Write-Attributes" operation by sending a CoAP PUT on the corresponding path, and can be accessed using the "Discover" operation. The Discover operation (uses a CoAP GET on the corresponding path along with the application/link-format Content type, to retrieve a list of Objects, Object Instances, Resources and their attached or assigned attributes, from the LwM2M Client (see [LwM2M-CORE] for more details on "Discover" operation). With the "Write-Attributes" operation one or more Attributes can be written at a time. When several Attributes are specified in the same "Write-Attributes" operation, they MUST be consistent according to the rules defined in [LwM2M-CORE], otherwise the "Write-Attributes" operation MUST be rejected (4.00 Bad Request).The values of these Attributes are used by the Information Reporting interface to determine how often Notifications are sent regarding that Resource. A LwM2M Client MAY support a separate set of configured Attributes for each individual LwM2M Server.

A "Write-Attributes" operation specifies which value is set to which Attribute and at which level (Object / Object Instance / Resource / Resource Instance). In a similar way, the same operation without value for the specified Attribute, MUST be used to unset this Attribute for the given level; then the precedence rules applies when notification occurs (see [LwM2M-CORE]).

As example:

1.  Write-Attributes /3/0/9?pmin=1 means the Battery Level value will be notified to the Server with a minimum interval of 1sec; this value is set at the Resource level.

2.  Write-Attributes /3/0/9?pmin means the Battery Level will be notified to the Server with a minimum value (pmin) given by the default one (resource 2 of Object Server ID=1), or with another value if this Attribute has been set at another level (Object or Object Instance: see [LwM2M-CORE]).

3.  Write-Attributes /3/0?pmin=10 means that all Resources of Object Instance 0 of the Object ‘Device (ID:3)' will be notified to the Server with a minimum interval of 10 sec; this value is set at the Object Instance level.

4.  Write-Attributes /3/0/9?gt=45&st=10 means the Battery Level will be notified to the Server when:

    1.  old value is 45 and new value is 50 due to gt condition

    2.  old value is 38 and new value is 49 due to both gt and step conditions
    

  Note: if old value is 48 and new value is 42, it is not considered as satisfying any condition.

5.  Write-Attributes /3/0/9?lt=20&gt=85&st=10 means the Battery Level will be notified to the Server when:

    1.  old value is 75 and new value is 90 due to both gt and step conditions

    2.  old value is 50 and new value is 10 due to both lt and step conditions

    3.  old value is 87 and new value is 99 due to step condition


   Note: if old value is 17 and new value is 24, this is not considered for any action by the client as no condition is satisfied

{:page-break}
<table>
    <caption>Operation to Method and URI Mapping (Device Management &amp; Service Enablement Interface)</caption>
    <thead>
        <tr>
            <th>Operation</th>
            <th>CoAP Method and Content Formats</th>
            <th>Path</th>
        </tr>
    </thead>
    <tbody>
    <tr>
        <td>Read</td>
        <td>GET<br>
            Accept: see [LwM2M-CORE]</td>
        <td>/{Object ID}/{Object Instance ID}/{Resource ID}/{Resource Instance ID}</td>
    </tr>
    <tr>
        <td>Read-Composite</td>
        <td>FETCH<br>
            Content Format: SenML CBOR or SenML JSON<br>
            Accept: SenML CBOR or SenML JSON (see [LwM2M-CORE])</td>
        <td>URI paths are provided in request payload</td>
    </tr>        
    <tr>
        <td>Discover</td>
        <td>GET<br>
            Accept: application/link-format</td>
        <td>/{Object ID}/{Object Instance ID}/{Resource ID}?depth={Depth}</td>
    </tr>
    <tr>
        <td>Write (Replace)</td>
        <td>PUT<br>
            Content Format: see [LwM2M-CORE]</td>
        <td>/{Object ID}/{Object Instance ID}<br>
            /{Object ID}/{Object Instance ID}/{Resource ID}<br>
            /{Object ID}/{Object Instance ID}/{Resource ID}/{Resource Instance ID}</td>
    </tr>
    <tr>
        <td>Write (Partial Update)</td>
        <td>POST<br>
            Content Format: see [LwM2M-CORE]</td>
        <td>/{Object ID}/{Object Instance ID}<br>
            /{Object ID}/{Object Instance ID}/{Resource ID} only for Multiple-Instance Resources</td>
    </tr>
    <tr>
        <td>Write-Attributes</td>
        <td>PUT</td>
        <td>/{Object ID}/{Object Instance ID}/{Resource ID}/{Resource Instance ID}?pmin={minimum period}&amp;pmax={maximum period}&amp;gt={greater than}&amp;lt={less than}&amp;st={step}&epmin={minimum evaluation period}&epmax={maximum evaluation period}&edge={0 or 1}&con={0 or 1}</td>
    </tr>
    <tr>
        <td>Write-Composite</td>
        <td>iPATCH<br>
            Content Format: SenML CBOR or SenML JSON (see [LwM2M-CORE])</td>
        <td>URI path and a new value for each of resource to be written to is provided in request payload</td>
    </tr>
    <tr>
        <td>Execute</td>
        <td>POST<br>
            Content Format: none, or text/plain (see [LwM2M-CORE])</td>
        <td>/{Object ID}/{Object Instance ID}/{Resource ID}</td>
    </tr>
    <tr>
        <td>Create</td>
        <td>POST<br>
            Content Format: SenML CBOR, SenML JSON, or TLV (see [LwM2M-CORE]) </td>
        <td>/{Object ID}</td>
    </tr>
    <tr>
        <td>Delete</td>
        <td>DELETE</td>
        <td>/{Object ID}/{Object Instance ID}<br>/{Object ID}/{Object Instance ID}/{Resource ID}/{Resource Instance ID}</td>
    </tr>
    </tbody>
</table>


Available CoAP response codes to the operations are given in [Table Response Codes: Device Management and Service Enablement Interface]()

<figure class="text-center">
      <img src="images/dm_se_interface_exchanges.svg" alt="Example of Device Management & Service Enablement interface exchanges">
      <figcaption>Example of Device Management & Service Enablement interface exchanges</figcaption>
</figure>

<figure class="text-center">
      <img src="images/object_creation_deletion.svg" alt="Example of Object Creation and Deletion">
      <figcaption>Example of Object Creation and Deletion</figcaption>
</figure>

#### Information Reporting Interface

Periodic and event-triggered reporting about Resource values from the LwM2M Client to the LwM2M Server is achieved through CoAP Observation \[OBSERVE\]. The mechanism to configure and manage observations and notifications is specified in \[OBSERVE\]. The LwM2M Server may set the Observe attributes of a Resource to affect the behavior of its notifications using the "Write-Attributes" operation (see \[LwM2M-CORE\]).

The value for an Object, an Object Instance, a Resource or a Resource Instance can also be sent by a LwM2M Client in an unsolicited manner to the LwM2M Server by using the "Send" operation. That operation is carried out by sending a CoAP POST request to a predetermined path on the LwM2M Server. That path is /dp, standing for "data push". The "Content-Format" CoAP option (header) MUST be set to inform the LwM2M Server of the format of the payload sent in that CoAP POST request. The content format MUST be either SenML JSON or SenML CBOR format described in [LwM2M-CORE].

As example:

Reporting the location of the LwM2M Client and the radio signal strength:
```
CoAP POST Uri-Path:"dp" Content-Format:application/senml+json
Payload:
[
  {"n":"/6/0/0", "v":43.61092},
  {"n":"/6/0/1", "v":3.87723},
  {"n":"/4/0/2", "v":-49}
]
```

<table>
    <caption>Operation to Method and URI Mapping (Information Reporting Interface)</caption>
    <thead>
        <tr>
            <th>Operation</th>
            <th>CoAP Method</th>
            <th>Path</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Observe</td>
            <td>GET with Observe option = 0</td>
            <td>/{Object ID}/{Object Instance ID}/{Resource ID}/{Resource Instance ID}?pmin={minimum period}&amp;pmax={maximum period}&amp;gt={greater than}&amp;lt={less than}&amp;st={step}&epmin={minimum evaluation period}&epmax={maximum evaluation period}&edge={0 or 1}&con={0 or 1}</td>
        </tr>
	    <tr>
            <td>Observe-Composite</td>
            <td>FETCH with Observe option = 0, Content Format ID: SenML CBOR or SenML JSON, Accept: Content Format ID: SenML CBOR or SenML JSON (see [LwM2M-CORE])</td>
            <td>/?pmin={minimum period}&amp;pmax={maximum period}&amp;epmin={minimum evaluation period}&amp;epmax={maximum evaluation period}&amp;con={0 or 1}</br>URI paths for resources to be observed are provided in request payload</td>
        </tr>
        <tr>
            <td>Cancel Observation</td>
            <td>GET with Observe option= 1</td>
            <td>/{Object ID}/{Object Instance ID}/{Resource ID}/{Resource Instance ID}</td>
        </tr>
	<tr>
            <td>Cancel Observation-Composite</td>
            <td>FETCH with Observe option= 1</td>
            <td>URI paths for exactly the same resources listed in the "Observe-Composite" operation that is being canceled are provided in request payload</td>
        </tr>
        <tr>
            <td>Notify</td>
            <td>Asynchronous Response</td>
            <td></td>
        </tr>
	<tr>
            <td>Send</td>
            <td>POST<br>
                 Content Format: SenML CBOR or SenML JSON (See [LwM2M-CORE])</td>
            <td>/dp</td>
        </tr>
    </tbody>
</table>


Available CoAP response codes to the operations are given in [Table Response Codes: Information Reporting Interface]()

<figure class="text-center">
      <img src="images/information_reporting_exchange.svg" alt="Example of an Information Reporting exchange">
      <figcaption>Example of an Information Reporting exchange</figcaption>
</figure>

### Queue Mode Operation

The LwM2M Server MUST support Queue Mode and the LwM2M Client SHOULD support Queue Mode.

The LwM2M Client indicates support of Queue Mode in the registration to the LwM2M Server. When the LwM2M Client is not online, the LwM2M Server does not immediately send downlink requests, but instead waits until the LwM2M Client is online. As such, the Queue Mode offers functionality for a LwM2M Client to inform the LwM2M Server that it may be disconnected for an extended period of time. The LwM2M Server uses this information to adjust timers and relay messages from and to the LwM2M Client accordingly.

The LwM2M Client lets the LwM2M Server know it is awake by sending a registration update message as a Confirmable message. Absent any application specific profiles, it is RECOMMENDED that the LwM2M Client waits at least MAX\_TRANSMIT\_WAIT seconds \[CoAP\] from the last CoAP message it sent to the LwM2M Server before intentionally going offline.

In order to find out whether a message was successfully delivered from the LwM2M server to the LwM2M client the LwM2M server has to rely on a response. This response tells the server that the message has been received and processed (regardless of what the result of the processing was). A response can be conveyed to the server in two ways:

-   ACK piggybacking the response, or

-   Separate CON/non-CON containing the response.

If message delivery fails, for example, because the message got lost due to network connectivity issues or because the LMW2M Client was sleeping then CoAP re-transmission behaviour at the LwM2M Server will try to retransmit the message. The CoAP stack at the LwM2M Server will resend the message up to a certain number of attempts, as described in Section 4.2 of \[CoAP\]. If these retransmission attempts fail, the CoAP stack at the LwM2M Server will give up and inform the LwM2M layer. The LwM2M Server has to inform the application about this failed delivery but this API is outside the scope of the LwM2M specification.

Due to the congestion control approach used by CoAP the LwM2M Server has to wait for a response to a request before sending out the next request from the queue since \[CoAP\] limits the number of simultaneous outstanding interactions to 1.

Despite the title of the functionality, i.e. Queue Mode, this specification does not mandate an implementation to use queues nor does it specify where such a queue would exist (or any details of such queuing mechanism).

A typical Queue Mode sequence follows the following steps:

1.  The LwM2M Client registers to the LwM2M Server and requests the LwM2M Server to run in Queue mode by using the correct setting in the registration.

2.  The LwM2M Client is recommended to use the CoAP MAX\_TRANSMIT\_WAIT parameter to set a timer for how long it shall stay awake since last sent/received message to/from the LwM2M Server. After MAX\_TRANSMIT\_WAIT seconds without any messages from the LwM2M Server, the LwM2M Client enters a sleep mode.

3.  At some point in time the LwM2M Client wakes up again and transmits a registration update message. Note: During the time the LwM2M Client has been sleeping the IP address assigned to it may have been released and / or existing NAT bindings may have been released. If this is the case, then the client needs to re-run the TLS/DTLS handshake with the LwM2M Server since an IP address and/or port number change will destroy the existing security context. For performance and efficiency reasons it is RECOMMENDED to utilize the TLS/DTLS session resumption.

4.  When the LwM2M Server receives a message from the Client, it determines whether any messages need to be sent to the LwM2M Client, as instructed by the application server.

Below is an example flow for Queue Mode in relation to Device Management & Service Enablement Interface.

<figure class="text-center">
      <img src="images/dm_se_interface_exchanges_queue_mode.svg" alt="Example of Device Management & Service Enablement interface exchanges for Queue Mode">
      <figcaption>Example of Device Management & Service Enablement interface exchanges for Queue Mode</figcaption>
</figure>

Below is an example flow for Queue Mode in relation to Information Reporting Interface

<figure class="text-center">
      <img src="images/information_reporting_exchange_queue_mode.svg" alt="Example of an Information Reporting exchange for Queue Mode">
      <figcaption>Example of an Information Reporting exchange for Queue Mode</figcaption>
</figure>

### SMS Trigger Mechanism

When the LwM2M Client has registered with a Current Transport Binding different than "S", if the Trigger Resource in the matching Server Object Instance exists and is true, the LwM2M Server MAY send an Execute operation over SMS. The LwM2M Client MUST NOT send a response to the Execute operation. 

The LwM2M Client MUST ignore any other LwM2M operation sent over SMS when the Current Transport Binding is different than "S".

The LwM2M Client informs the LwM2M Server of its capability to receive trigger messages over SMS by including the SMS Number parameter in its registration message as defined in [LwM2M-CORE].

#### Registration Update Trigger

When the LwM2M Client has registered to a LwM2M Server, the LwM2M Server can make the LwM2M Client update its registration by executing the Registration Update Trigger Resource in the matching Server Object Instance (see Appendix E of [LwM2M-CORE]).

Below is an example flow to trigger the LwM2M Client in Queue Mode to send Update message to the LwM2M Server regardless of expiration of Lifetime. Post /1/x/8 would bring the LwM2M Client online to connect to the LwM2M server, where "x" represents the right instance pointing to the server. Optionally, a transport can be included in the Registration Update Trigger. Upon receiving POST /1/x/8?0=U, the LwM2M Client MAY use this transport binding to reconnect with the LwM2M Server. In this example, U refers to the UDP Binding. The possible values for the Binding Resource are listed in [LwM2M-CORE].

<figure class="text-center">
      <img src="images/queue_mode_sms_registration_update_trigger.svg" alt="Example of Device Management & Service Enablement interface exchanges for Queue Mode with SMS Registration Update Trigger">
      <figcaption>Example of Device Management & Service Enablement interface exchanges for Queue Mode with SMS Registration Update Trigger</figcaption>
</figure>

#### Bootstrap Trigger

When the LwM2M Client has registered to a LwM2M Server, the LwM2M Server can make the LwM2M Client initiate a "Client Initiated Bootstrap" procedure by executing the Bootstrap-Request Trigger Resource in the matching Server Object Instance (see Appendix E of [LwM2M-CORE]).

{:page-break}

### Response Codes

This section lists available response codes of each operation. The codes are divided into each interface. These are the only valid response codes defined in for the LwM2M Enabler.

<table>
    <caption>Response Codes: Bootstrap Interface</caption>
    <thead>
        <tr>
            <th>Operations</th>
            <th>Available CoAP Response Codes</th>
            <th>Reason Phrase</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="3">Bootstrap-Request</td>
            <td>2.04 Changed</td>
            <td>Bootstrap-Request is completed successfully</td>
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>Unknown Endpoint Client Name</td>
        </tr>
	<tr>
            <td>4.15 Unsupported content format</td>
            <td>The specified format is not supported</td>
        </tr>
        <tr>
            <td rowspan="6">Bootstrap-Read</td>
            <td>2.05 Content</td>
            <td>"Read" operation is completed successfully</td>
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>4.01 Unauthorized</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>4.04 Not Found</td>
            <td> URI of "Read" operation is not found</td>
        </tr>
        <tr>
            <td>4.05 Method Not Allowed</td>
            <td>Target is not allowed for "Read" operation</td>
        </tr>
        <tr>
            <td>4.06 Not Acceptable</td>
            <td>None of the preferred Content-Formats can be returned</td>
        </tr>
        <tr>
            <td rowspan="3">Bootstrap-Write</td>
            <td>2.04 Changed</td>
            <td>"Write" operation is completed successfully</td>
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>The format of data to be written is different</td>
        </tr>
        <tr>
            <td>4.15 Unsupported content format</td>
            <td>The specified format is not supported</td>
        </tr>
        <tr>
            <td rowspan="3">Bootstrap-Discover</td> 
            <td>2.05 Content</td>
            <td>"Discover" operation is completed successfully</td>
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>4.04 Not Found</td>
            <td>URI of "Discover" operation is not found</td>            
        </tr>
        <tr>
            <td rowspan="2">Bootstrap-Delete</td>
            <td>2.02 Deleted</td>
            <td>"Delete" operation is completed successfully</td>
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>Bad or unknown URI provided</td>
        </tr>
        <tr>
            <td rowspan="3">Bootstrap-Finish</td>
            <td>2.04 Changed</td>
            <td>Bootstrap-Finished is completed successfully</td> 
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>Bad URI provided</td>            
        </tr>
        <tr>
            <td>4.06 Not Acceptable</td>
            <td>Inconsistent loaded configuration</td>            
        </tr>
	<tr>
            <td rowspan="6">Bootstrap-Pack-Request</td>
            <td>2.05 Content</td>
            <td>The response includes the Bootstrap-Pack.</td>
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>4.01 Unauthorized</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>4.04 Not Found</td>
            <td> URI of "Bootstrap-Pack-Request" operation is not found</td>
        </tr>
        <tr>
            <td>4.05 Method Not Allowed</td>
            <td>Target is not allowed for "Bootstrap-Pack-Request" operation</td>
        </tr>
        <tr>
            <td>4.06 Not Acceptable</td>
            <td>None of the preferred Content-Formats can be returned</td>
        </tr>
    </tbody> 
</table>


{:page-break}

<table>
    <caption>Response Codes: Client Registration Interface</caption>
    <thead>
        <tr>
            <th>Operations</th>
            <th>Available CoAP Response Codes</th>
            <th>Reason Phrase</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="5">Register</td>
            <td>2.01 Created</td>
            <td>"Register" operation is completed successfully</td>
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>The mandatory parameter is not specified or unknown parameter is specified<br>Unknown Endpoint Client Name<br>Endpoint Client Name does not match with CN field of X.509 Certificates</td>
        </tr>
        <tr>
            <td>4.03 Forbidden</td>
            <td>The Endpoint Client Name registration in the LwM2M Server is not allowed.</td>
        </tr>
        <tr>
            <td>4.09 Conflict</td>
            <td>The Client registration cannot be successfully used by the LwM2M Server to determine the list of Objects supported and Object Instances available on the LwM2M Client.
</td>
        </tr>
        <tr>
            <td>4.12 Precondition Failed</td>
            <td>Supported LwM2M Versions of the Server and the Client are not compatible</td>
        </tr>
        <tr>
            <td rowspan="3">Update</td>
            <td>2.04 Changed</td>
            <td>"Update" operation is completed successfully</td>
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>The mandatory parameter is not specified or unknown parameter is specified</td>
        </tr>
        <tr>
            <td>4.04 Not Found</td>
            <td>URI of "Update" operation is not found</td>
        </tr>
        <tr>
            <td rowspan="3">De-register</td>
            <td>2.02 Deleted</td>
            <td>"De-register" operation is completed successfully</td>
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>4.04 Not Found</td>
            <td>URI of "De-register" operation is not found</td>
        </tr>
    </tbody>
</table>

<table>
    <caption>Response Codes: Device Management and Service Enablement Interface</caption>
    <thead>
        <tr>
            <th>Operations</th>
            <th>Available CoAP Response Codes</th>
            <th>Reason Phrase</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="7">Create</td>
            <td>2.01 Created</td>
            <td>"Create" operation is completed successfully</td>
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>Target (i.e., Object) already exists<br>Mandatory Resources are not specified<br>Content Format is not specified</td>
        </tr>
        <tr>
            <td>4.01 Unauthorized</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>4.04 Not Found</td>
            <td>URI of "Create" operation is not found</td>
        </tr>
        <tr>
            <td>4.05 Method Not Allowed</td>
            <td>Target is not allowed for "Create" operation</td>
        </tr>
	<tr>
            <td>4.06 Not Acceptable</td>
            <td></td>
        </tr>
        <tr>
            <td>4.15 Unsupported content format</td>
            <td>The specified format is not supported</td>
        </tr>
        <tr>
            <td rowspan="6">Read</td>
            <td>2.05 Content</td>
            <td>"Read" operation is completed successfully</td>
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>4.01 Unauthorized</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>4.04 Not Found</td>
            <td> URI of "Read" operation is not found</td>
        </tr>
        <tr>
            <td>4.05 Method Not Allowed</td>
            <td>Target is not allowed for "Read" operation</td>
        </tr>
        <tr>
            <td>4.06 Not Acceptable</td>
            <td>None of the preferred Content-Formats can be returned</td>
        </tr>
	<tr>
            <td rowspan="7">Read-Composite</td>
            <td>2.05 Content</td>
            <td>"Read" operation is completed successfully</td>
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>4.01 Unauthorized</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>4.04 Not Found</td>
            <td> URI of "Read" operation is not found</td>
        </tr>
        <tr>
            <td>4.05 Method Not Allowed</td>
            <td>Target is not allowed for "Read" operation</td>
        </tr>
        <tr>
            <td>4.06 Not Acceptable</td>
            <td>None of the preferred Content-Formats can be returned</td>
        </tr>
	<tr>
            <td>4.15 Unsupported Content-Format</td>
            <td>The specified format is not supported</td>
        </tr>
        <tr>
            <td rowspan="10">Write</td>
            <td>2.04 Changed</td>
            <td>"Write" operation is completed successfully</td>
        </tr>
        <tr>
            <td>2.31 Continue</td>
            <td></td>
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>The format of data to be written is different</td>
        </tr>
        <tr>
            <td>4.01 Unauthorized</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>4.04 Not Found </td>
            <td>URI of "Write" operation is not found </td>
        </tr>
        <tr>
            <td>4.05 Method Not Allowed</td>
            <td>Target is not allowed for "Write" operation </td>
        </tr>
	<tr>
            <td>4.06 Not Acceptable</td>
            <td></td>
        </tr>
        <tr>
            <td>4.08 Request Entity Incomplete</td>
            <td></td>
        </tr>
        <tr>
            <td>4.13 Request entity too large</td>
            <td></td>
        </tr>
        <tr>
            <td>4.15 Unsupported content format</td>
            <td>The specified format is not supported</td>
        </tr>
        <tr>
	<tr>
            <td rowspan="6">Write-Composite</td>
            <td>2.04 Changed</td>
            <td>"Write" operation is completed successfully</td>
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>The format of data to be written is different</td>
        </tr>
        <tr>
            <td>4.01 Unauthorized</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>4.04 Not Found </td>
            <td>URI of "Write" operation is not found </td>
        </tr>
        <tr>
            <td>4.05 Method Not Allowed</td>
            <td>Target is not allowed for "Write" operation </td>
        </tr>
	<tr>
            <td>4.06 Not Acceptable</td>
            <td></td>
        </tr>
        <tr>
            <td rowspan="5">Delete</td>
            <td>2.02 Deleted</td>
            <td>"Delete" operation is completed successfully</td> 
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>4.01 Unauthorized </td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>4.04 Not Found </td>
            <td>URI of "Delete" operation is not found</td>
        </tr>
        <tr>
            <td>4.05 Method Not Allowed</td>
            <td>Target is not allowed for "Delete" operation</td>
        </tr>
        <tr>
            <td rowspan="5">Execute</td>
            <td>2.04 Changed</td>
            <td>"Execute" operation is completed successfully</td> 
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>The LwM2M Client doesn't understand the argument in the payload</td>
        </tr>
        <tr>
            <td>4.01 Unauthorized </td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>4.04 Not Found</td>
            <td>URI of "Execute" operation is not found</td>
        </tr>
        <tr>
            <td>4.05 Method Not Allowed</td>
            <td>Target is not allowed for "Execute" operation</td>
        </tr>
        <tr>
            <td rowspan="5">Write-Attributes</td>
            <td>2.04 Changed</td>
            <td>"Write-Attributes" operation is completed successfully</td> 
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>The format of attribute to be written is different</td>
        </tr>
        <tr>
            <td>4.01 Unauthorized</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>4.04 Not Found </td>
            <td>URI of "Write-Attributes" operation is not found</td>
        </tr>
        <tr>
            <td>4.05 Method Not Allowed</td>
            <td>Target is not allowed for Write-Attributes operation</td>
        </tr>
        <tr>
            <td rowspan="5">Discover</td>
            <td>2.05 Content</td>
            <td>"Discover" operation is completed successfully</td> 
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>4.01 Unauthorized</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>4.04 Not Found </td>
            <td>URI of "Discover" operation is not found</td>
        </tr>
        <tr>
            <td>4.05 Method Not Allowed</td>
            <td>Target is not allowed for Discover operation</td>
        </tr>
    </tbody>
</table>

{:page-break}

<table>
    <caption>Response Codes: Information Reporting Interface</caption> 
    <thead>
        <tr>
            <th>Operations</th>
            <th>Available CoAP Response Codes</th>
            <th>Reason Phrase</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="6">Observe<br>Cancel Observation<br>Cancel Observation-Composite</td>
            <td>2.05 Content</td>
            <td>Operation is completed successfully</td>
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>4.01 Unauthorized</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>4.04 Not Found</td>
            <td>URI of Operation is not found or not supported</td>
        </tr>
        <tr>
            <td>4.05 Method Not Allowed</td>
            <td>Target is not allowed for the Operation</td>
        </tr>
        <tr>
            <td>4.06 Not Acceptable</td>
            <td>None of the preferred Content-Formats can be returned</td>
        </tr>
	<tr>
            <td rowspan="7">Observe-Composite</td>
            <td>2.05 Content</td>
            <td>Operation is completed successfully</td>
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>4.01 Unauthorized</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>4.04 Not Found</td>
            <td>URI of Operation is not found or not supported</td>
        </tr>
        <tr>
            <td>4.05 Method Not Allowed</td>
            <td>Target is not allowed for the Operation</td>
        </tr>
        <tr>
            <td>4.06 Not Acceptable</td>
            <td>None of the preferred Content-Formats can be returned</td>
        </tr>
	<tr>
            <td>4.15 Unsupported Content-Format</td>
            <td>The specified format is not supported</td>
        </tr>
        <tr>
            <td>Notify</td>
            <td>2.05 Content</td>
            <td>"Notify" operation is completed successfully</td>
        </tr>
	<tr>
            <td rowspan="5">Send</td>
            <td>2.04 Changed</td>
            <td>"Send" operation completed successfully</td> 
        </tr>
        <tr>
            <td>4.00 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>4.04 Not Found </td>
            <td>Reported Object was not registered to the LwM2M Server</td>
        </tr>
    </tbody>
</table>

If any operation cannot be completed in the client and the reason cannot be described by a 4.xx response code, then a response code from the "Server Error" list of 5.xx return codes MUST be returned.

### CoAP Transport Bindings

The LwM2M Server and the LwM2M Client MUST support the UDP binding, specified in Section [UDP Binding](), and the LwM2M Server SHOULD support the SMS binding specified in Section [SMS Binding](). The LwM2M Client MAY support other bindings defined in this specification.

#### UDP Binding

The CoAP binding for UDP is defined in \[CoAP\]. The protocol has an IANA registered scheme of coap:// and a default port of 5683. The UDP binding is used in NoSec (no security) mode. The CoAP binding for DTLS is defined in \[CoAP\]. The URI scheme is coaps:// and the default port is 5684. 

Reliability of CoAP messages over the DTLS transport is provided by the built-in retransmission mechanism of CoAP. All the LwM2M operations using CoAP layer MUST be Confirmable CoAP messages, except as follows:

  * "Notify" which may be a Non-Confirmable CoAP message
  * Triggering an execution on a resource can be a Non-Confirmable CoAP message

#### TCP Binding

The CoAP binding for TCP is defined in \[CoAP_TCP\]. The protocol has an IANA registered scheme of coap+tcp:// and a default port of 5683. The TCP binding is used in NoSec (no security) mode. The CoAP binding for TLS is defined in \[CoAP_TCP\]. The URI scheme for CoAP over TLS is coaps+tcp:// and the default port is 5684.

#### SMS Binding

CoAP is used over SMS in this transport binding by placing a CoAP message in the SMS payload using 8-bit encoding. SMS concatenation MAY be used for messages larger than 140 characters. CoAP retransmission is disabled for this binding. A LwM2M Client indicates the use of this binding by including a parameter ("sms") in its registration to the LwM2M Server including the node's MSISDN number.

#### LoRaWAN Binding

When using LwM2M over LoRaWAN, the LoRaWAN Endpoint MUST be the LwM2M Client. The LwM2M Server and the LwM2M Bootstrap-Server MUST be LoRaWAN Application Servers as defined by [LoRaWAN].

The LoRaWAN binding is used in NoSec (no security) mode. Security is provided by the LoRaWAN transport. 

The Server URI MUST be in the form "lorawan://{port}", *port* being the FPort between 1 and 255 as defined in [LoRaWAN].

Due the limited bandwidth of the LoRaWAN transport, the "register" operation has different parameter requirements on this transport.

<table>
    <caption>LoRaWAN binding registration parameters</caption>
    <thead>
    <tr>
        <th>Parameter</th>
        <th>Required</th>
        <th>Default Value</th>
        <th>Notes</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td>Endpoint Client Name</td>
        <td>No</td>
        <td>DevEUI</td>
        <td>This parameter is omitted if the endpoint name is identical to the endpoint DevEUI</td>
    </tr>
    <tr>
        <td>Lifetime</td>
        <td>No</td>
        <td>2 592 000 (30 days)</td>
        <td></td>
    </tr>
    <tr>
        <td>LwM2M Version</td>
        <td>No</td>
        <td>1.1 or 1.2</td>
        <td>This parameter can be omitted if the LwM2M enabler version is 1.1</td>
    </tr>
    <tr>
        <td>Binding Mode</td>
        <td>No</td>
        <td></td>
        <td>By nature, the LoRaWAN network is equivalent to Queue Mode.</td>
    </tr> 
    <tr>
        <td>SMS Number</td>
        <td>No</td>
        <td></td>
        <td>The value of this parameter is the MSISDN where the LwM2M Client can be reached for use with the SMS binding.</td>
    </tr> 
    <tr>
        <td>Objects and Object Instances</td>
        <td>No</td>
        <td></td>
        <td>The list of Objects supported and Object Instances available on the LwM2M Client (Security Object ID:0 MUST not be part of this list).</td>
    </tr>
    </tbody>
</table>

The Objects and Object Instances list MAY be omitted in the first registration attempt. If the LwM2M Server can not retrieve this list by out-of-band means (e.g. preconfiguration of the endpoint on the Application Server), it MUST reply to the registration message with a 4.09 (Conflict) code. In this case, the LwM2M Client SHOULD retry to register with a registration message that MUST include the Objects and Object Instances list.

CoAP is used over LoRaWAN in this transport binding by placing a CoAP message in the LoRaWAN packet payload. Piggybacked responses MUST be used.

Due to the high latency of the LoRaWAN network, its asymmetric nature and the intermittent availability of the LoRaWAN endpoints, CoAP retransmission is disabled for this binding and is replaced by the logic defined in [CoAP over LoRaWAN (Normative)]().

#### CIoT Binding

CoAP can be used over 3GPP CIoT [3GPP-TR_23.720] as part of IP and non-IP data transmissions over User Plane and Control Plane. This is described in more detail in Appendix D of [LwM2M-CORE].

## HTTP Transport Binding

This section defines the HTTP transport binding used by LwM2M interfaces. The HTTP transport binding places minimal demands on the version of the HTTP protocol. This transport binding is optional to implement and optional to use. When implemented, this transport binding SHOULD use HTTP/1.1 \[RFC2616\] and MAY use HTTP/2 \[RFC8132\]. If an implementation claims conformance to this transport binding it MUST support the features described in this section. For the Information Reporting Interface only a subset of the functionality is provided by this version of the specification, namely the Send operation. 

HTTP communication is secured using TLS.  

### URI Identifier & Operation Mapping

The URI path is used to identify the interface, Object, Object Instance and Resource the request is for, and is encoded as described in Section 3.2 of RFC 2616. 

#### Alternate Path

By default, the LwM2M Objects are located under the root path. However, devices might be hosting other HTTP Resources on an endpoint, and there may be the need to place LwM2M Objects under an alternate path.

When registering, or updating its registration, a LwM2M Client MAY include an OMA LwM2M link in addition to the Object links in the registration payload. The registration payload uses the CoRE Link Format, which extends the HTTP Link Header field specified in\[RFC5988\]. The link is identified Resource Type parameter "oma.lwm2m", as defined in RFC 6690 \[RFC6690\]. 

This link MUST NOT contain numerical URI segment.

For instance, the Example Client from Appendix F of [LwM2M-CORE] may place Objects under the "/lwm2m" path. The registration payload would be as follows:

&lt;/lwm2m&gt;;rt="oma.lwm2m", &lt;/lwm2m /1/0&gt;,&lt;/lwm2m /1/1&gt;,&lt;/lwm2m /2/0&gt;,&lt;/lwm2m /2/1&gt;,&lt;/lwm2m /2/2&gt;,&lt;/lwm2m /2/3&gt;,&lt;/lwm2m /2/4&gt;,&lt;/lwm2m /3/0&gt;,&lt;/lwm2m /4/0&gt;,&lt;/lwm2m /5&gt;

When using the Device Management & Service Enablement Interface and the Information Reporting Interface, the LwM2M Server MUST prepend the OMA LwM2M link to the path in the HTTP messages. Example: GET /lwm2m/3/0/0.

When using the Bootstrap Interface, the LwM2M Bootstrap-Server MUST use HTTP paths only in the form /{Object ID}/{Object Instance ID}/{Resource ID}. It is the responsibility of the LwM2M Client to map these paths to its alternate path.

The Resource Type value "oma.lwm2m" is part of IANA registry.

#### Bootstrap Interface

This section provides a mapping of the LwM2M Bootstrap operations to the underlying transport. 

<table>
    <caption>Operation to Method and URI Mapping (Bootstrap Interface)</caption>
        <thead>
            <tr>
                <th>Operation</th>
                <th>HTTP Method</th>
                <th>URI</th>
            </tr>
        </thead>
    <tbody>
        <tr>
            <td>Bootstrap-Request</td>
            <td>POST</td>
            <td>/bs?ep={Endpoint Client Name}&pct={Preferred Content Format}</td>
        </tr>
        <tr>
            <td>Bootstrap-Read</td>
            <td>GET Accept:Content Format ID: TLV, SenML CBOR or SenML JSON</td>
            <td>/{Object ID} in LwM2M 1.1 and therafter, Object ID MUST be '2' (Access Control Object)</td>
        </tr>
        <tr>
            <td>Bootstrap-Write</td>
            <td>PUT</td>
            <td>/{Object ID}/{Object Instance ID}/{Resource ID}</td>
        </tr>
        <tr>
            <td>Bootstrap-Delete</td>
            <td>DELETE</td>
            <td>/{Object ID}/{Object Instance ID}</td>
        </tr>
        <tr>
            <td>Bootstrap-Discover</td>
            <td>GET Accept:
                application/link-format</td>
            <td>/{Object ID}</td>
        </tr>
        <tr>
            <td>Bootstrap-Finish</td>
            <td>POST</td>
            <td>/bs</td>
        </tr>
	<tr>
            <td>Bootstrap-Pack-Request</td>
            <td>GET Accept:Content Format ID: SenML CBOR or SenML JSON</td>
            <td>/bspack?ep={Endpoint Client Name}</td>
        </tr>
    </tbody>
</table>

Available HTTP response codes to the operations are given in [Table Response Codes: Bootstrap Interface]()

#### Registration Interface

The registration interface is used by a LwM2M Client to register with a LwM2M Server, identified by the LwM2M Server URI.

Registration is performed by sending a HTTP POST to the LwM2M Server URI /rd, with registration parameters passed as query string parameters as per [Table Operation to Method and URI Mapping (Registration Interface)]() and Object and Object Instances included in the payload as specified in [LwM2M-CORE]. The response includes the path to use for updating or deleting the registration. The LwM2M Server MUST return a location under the /rd path segment.

As the network connectivity may be limited or intermittent, it is advised to make several retries of the Registration if no reply is received from the LwM2M Server before considering the registration as failed.

When a new security context is created, the Client MUST register again to the LwM2M Server. 

Registration update is performed by sending a HTTP POST to the path returned to the LwM2M Client as a result of a successful registration.

De-registration is performed by sending a HTTP DELETE to the path returned to the LwM2M Client as a result of a successful registration.

<table>
    <caption>Operation to Method and URI Mapping (Registration Interface)</caption>
    <thead>
        <tr>
            <th>Operation</th>
            <th>HTTP Method</th>
            <th>URI</th>
        </tr>
    </thead>
    <tbody>
    <tr>
        <td><strong>Register</strong></td>
        <td>POST</td>
        <td>/rd?ep={Endpoint Client Name}&amp;lt={Lifetime}&amp;lwm2m={version}&amp;b={binding}&amp;Q&amp;sms={MSISDN}&amp;pid={ProfileID}</td>
    </tr>
    <tr>
        <td><strong>Update</strong></td>
        <td>POST</td>
        <td>/{location}?lt={Lifetime}&amp;b={binding}&amp;Q&amp;sms={MSISDN}</td>
    </tr>
    <tr>
        <td><strong>De-register</strong></td>
        <td>DELETE</td>
        <td>/{location}</td>
    </tr>
    </tbody>
</table>

Available HTTP response codes to the operations are given in [Table Response Codes: Client Registration Interface]()

Note: Throughout the present document the format of the MSISDN must be as specified in \[3GPP-TS\_23.003\]. According to this definition "+" is not preceding the country code.

#### Device Management & Service Enablement Interface

The Device Management & Service Enablement Interface is used to access a Resource, a Resource Instance, an array of Resource Instances, an Object Instance or all the Object Instances of an Object.

An Object Instance, a Resource or a Resource Instance are read by sending a HTTP GET to the corresponding path. The response includes the value in the corresponding format according to the specified Content-Format (see [LwM2M-CORE]). The request MAY specify an Accept option containing the preferred Content-Format to receive. When the specified Content-Format is not supported by the LwM2M Client, the request MUST be rejected.

Note that the response payload may be empty, for instance when performing a Read operation on an Object with no Object Instance. 

An Object Instance, a Resource or a Resource Instance are written by sending a HTTP PUT request to the corresponding path. The request includes the value to be written in the corresponding Plain Text, Opaque, TLV, CBOR, SenML CBOR, or SenML JSON format according to the Content-Format option, which MUST be specified \[RFC7540\]. In addition, an Object Instance can be written by sending a HTTP POST request; in that case the specified Content-Format MUST be one of the TLV, SenML CBOR or SenML JSON formats. HTTP PUT is used for Replace. HTTP POST or HTTP PATCH \[RFC5789\] is used for Partial Update.

The Write request MUST be rejected:
- when the specified Content-Format is not supported by the LwM2M Client
- when a requested operation on a Multiple-Instance Resource is not authorized by the LwM2M Client as described in [LwM2M-CORE].

A Resource is Executed by sending a HTTP POST to the corresponding path. The definition of the "Execute" operation is described in [LwM2M-CORE]. The "Execute" operation MAY contain a payload, if arguments are required. When a payload in plain text format is used then the ABNF syntax for the arguments described in [LwM2M-CORE] MUST be used.

Note that the behaviour of the "Execute" operation is specified in the Resource description of the Object.

An Object Instance is Created by sending a HTTP POST to the corresponding path. The request includes the value to be written in the corresponding TLV, SenML CBOR or SenML JSON format according to the Content-Format option which MUST be specified. The rules governing the creation of Resources in the targeted Object Instance are specified in [LwM2M-CORE]. If Object Instance is not listed at the request, the LwM2M Client MUST assign ID of that Object Instance and send back Object Instance ID to the LwM2M Server when Object Instance is Created.

An Object Instance is Deleted by sending a HTTP DELETE to the corresponding path.

When a Resource supports multiple instances, the Resource value is an array of Resource Instances.

&lt;NOTIFICATION&gt; Class Attributes MAY be set by a LwM2M Server using the "Write-Attributes" operation by sending a HTTP PUT on the corresponding path, and can be accessed using the "Discover" operation. The Discover operation (uses a HTTP GET on the corresponding path along with the application/link-format Content type, to retrieve a list of Objects, Object Instances, Resources and their attached or assigned attributes, from the LwM2M Client (see [LwM2M-CORE] for more details on "Discover" operation). With the "Write-Attributes" operation one or more Attributes can be written at a time. When several Attributes are specified in the same "Write-Attributes" operation, they MUST be consistent according to the rules defined in [LwM2M-CORE], otherwise the "Write-Attributes" operation MUST be rejected. The values of these Attributes are used by the Information Reporting interface to determine how often Notifications are sent regarding that Resource. A LwM2M Client MAY support a separate set of configured Attributes for each individual LwM2M Server.

Note: Notifications in the Information Reporting interface are provided only via the Send operation; no further operations from the Information Reporting interface are supported by this version of the transport binding. 

A "Write-Attributes" operation specifies which value is set to which Attribute and at which level (Object / Object Instance / Resource / Resource Instance). In a similar way, the same operation without value for the specified Attribute, MUST be used to unset this Attribute for the given level; then the precedence rules applies when notification occurs (see [LwM2M-CORE]).

Note: The HTTP transport binding does not support the Read-Composite and the Write-Composite operations. 

{:page-break}
<table>
    <caption>Operation to Method and URI Mapping (Device Management &amp; Service Enablement Interface)</caption>
    <thead>
        <tr>
            <th>Operation</th>
            <th>HTTP Method and Content Formats</th>
            <th>Path</th>
        </tr>
    </thead>
    <tbody>
    <tr>
        <td>Read</td>
        <td>GET<br>
            Accept: see [LwM2M-CORE]</td>
        <td>/{Object ID}/{Object Instance ID}/{Resource ID}/{Resource Instance ID}</td>
    </tr>
    <tr>
        <td>Discover</td>
        <td>GET<br>
            Accept: application/link-format</td>
        <td>/{Object ID}/{Object Instance ID}/{Resource ID}?depth={Depth}</td>
    </tr>
    <tr>
        <td>Write</td>
        <td>PUT or POST<br>
            Content Format: see [LwM2M-CORE]</td>
        <td>/{Object ID}/{Object Instance ID}/{Resource ID}/{Resource Instance ID}<br>
            /{Object ID}/{Object Instance ID}</td>
    </tr>
    <tr>
        <td>Write-Attributes</td>
        <td>PUT</td>
        <td>/{Object ID}/{Object Instance ID}/{Resource ID}/{Resource Instance ID}?pmin={minimum period}&amp;pmax={maximum period}&amp;gt={greater than}&amp;lt={less than}&amp;st={step}&epmin={minimum evaluation period}&epmax={maximum evaluation period}&edge={0 or 1}</td>
    </tr>
    <tr>
        <td>Execute</td>
        <td>POST<br>
            Content Format: none, text/plain, SenML CBOR, or SenML JSON (see [LwM2M-CORE])</td>
        <td>/{Object ID}/{Object Instance ID}/{Resource ID}</td>
    </tr>
    <tr>
        <td>Create</td>
        <td>POST<br>
            Content Format: SenML CBOR, SenML JSON, or TLV (see [LwM2M-CORE]) </td>
        <td>/{Object ID}</td>
    </tr>
    <tr>
        <td>Delete</td>
        <td>DELETE</td>
        <td>/{Object ID}/{Object Instance ID}<br>/{Object ID}/{Object Instance ID}/{Resource ID}/{Resource Instance ID}</td>
    </tr>
    </tbody>
</table>

Available HTTP status codes to the operations are given in [Table Response Codes: Device Management and Service Enablement Interface]()

#### Information Reporting Interface

This version of the HTTP binding supports only a subset of the functionality of the Information Reporting interface: only the Send operation is supported. The value for an Object, an Object Instance, a Resource or a Resource Instance is thereby sent by a LwM2M Client to the LwM2M Server by using the "Send" operation. That operation is carried out by sending a HTTP POST request to a predetermined path on the LwM2M Server. That path is /dp, standing for "data push". The "Content-Format" HTTP option (header) MUST be set to inform the LwM2M Server of the format of the payload sent in that HTTP POST request. The content format MUST be either SenML JSON or SenML CBOR format described in [LwM2M-CORE].

<table>
    <caption>Operation to Method and URI Mapping (Information Reporting Interface)</caption>
    <thead>
        <tr>
            <th>Operation</th>
            <th>HTTP Method</th>
            <th>Path</th>
        </tr>
    </thead>
    <tbody>
	<tr>
            <td>Send</td>
            <td>POST<br>
                 Content Format: SenML CBOR or SenML JSON (See [LwM2M-CORE])</td>
            <td>/dp</td>
        </tr>
    </tbody>
</table>

Available HTTP response codes to the operations are given in [Table Response Codes: Information Reporting Interface]()

### Queue Mode Operation

The LwM2M Server MUST support Queue Mode and the LwM2M Client SHOULD support Queue Mode.

The LwM2M Client indicates support of Queue Mode in the registration to the LwM2M Server. When the LwM2M Client is not online, the LwM2M Server does not immediately send downlink requests, but instead waits until the LwM2M Client is online. As such, the Queue Mode offers functionality for a LwM2M Client to inform the LwM2M Server that it may be disconnected for an extended period of time. The LwM2M Server uses this information to adjust timers and relay messages from and to the LwM2M Client accordingly.

The LwM2M Client lets the LwM2M Server know it is awake by sending a registration update message.

{:page-break}

### Response Codes

This section lists available response codes of each operation. The codes are divided into each interface. These are the only valid response codes defined in for the LwM2M Enabler.

<table>
    <caption>Response Codes: Bootstrap Interface</caption>
    <thead>
        <tr>
            <th>Operations</th>
            <th>Available HTTP Status Codes</th>
            <th>Reason Phrase</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="3">Bootstrap-Request</td>
            <td>200 OK</td>
            <td>Bootstrap-Request is completed successfully</td>
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>Unknown Endpoint Client Name</td>
        </tr>
	<tr>
            <td>415 Unsupported Media Type</td>
            <td>The specified format is not supported</td>
        </tr>
        <tr>
            <td rowspan="6">Bootstrap-Read</td>
            <td>200 OK or 204 No Content</td>
            <td>"Read" operation is completed successfully</td>
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>401 Unauthorized or 403 Forbidden</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>404 Not Found</td>
            <td> URI of "Read" operation is not found</td>
        </tr>
        <tr>
            <td>400 Bad Request or 405 Method Not Allowed</td>
            <td>Target is not allowed for "Read" operation</td>
        </tr>
        <tr>
            <td>406 Not Acceptable</td>
            <td>None of the preferred Content-Formats can be returned</td>
        </tr>
        <tr>
            <td rowspan="3">Bootstrap-Write</td>
            <td>200 OK or 204 No Content</td>
            <td>"Write" operation is completed successfully</td>
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>The format of data to be written is different</td>
        </tr>
        <tr>
            <td>415 Unsupported Media Type</td>
            <td>The specified format is not supported</td>
        </tr>
        <tr>
            <td rowspan="3">Bootstrap-Discover</td> 
            <td>200 OK</td>
            <td>"Discover" operation is completed successfully</td>
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>404 Not Found</td>
            <td>URI of "Discover" operation is not found</td>            
        </tr>
        <tr>
            <td rowspan="2">Bootstrap-Delete</td>
            <td>200 OK or 204 No Content</td>
            <td>"Delete" operation is completed successfully</td>
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>Bad or unknown URI provided</td>
        </tr>
        <tr>
            <td rowspan="3">Bootstrap-Finish</td>
            <td>200 OK or 204 No Content</td>
            <td>Bootstrap-Finished is completed successfully</td> 
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>Bad URI provided</td>            
        </tr>
        <tr>
            <td>406 Not Acceptable</td>
            <td>Inconsistent loaded configuration</td>            
        </tr>
	<tr>
            <td rowspan="6">Bootstrap-Pack-Request</td>
            <td>200 OK</td>
            <td>The response includes the Bootstrap-Pack.</td>
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>401 Unauthorized</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>404 Not Found</td>
            <td> URI of "Bootstrap-Pack-Request" operation is not found</td>
        </tr>
        <tr>
            <td>405 Method Not Allowed</td>
            <td>Target is not allowed for "Bootstrap-Pack-Request" operation</td>
        </tr>
        <tr>
            <td>406 Not Acceptable</td>
            <td>None of the preferred Content-Formats can be returned</td>
        </tr>
    </tbody> 
</table>


{:page-break}

<table>
    <caption>Response Codes: Client Registration Interface</caption>
    <thead>
        <tr>
            <th>Operations</th>
            <th>Available HTTP Response Codes</th>
            <th>Reason Phrase</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="5">Register</td>
            <td>201 Created</td>
            <td>"Register" operation is completed successfully</td>
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>The mandatory parameter is not specified or unknown parameter is specified<br>Unknown Endpoint Client Name<br>Endpoint Client Name does not match with CN field of X.509 Certificates</td>
        </tr>
        <tr>
            <td>401 Unauthorized or 403 Forbidden</td>
            <td>The Endpoint Client Name registration in the LwM2M Server is not allowed.</td>
        </tr>
        <tr>
            <td>409 Conflict</td>
            <td>The Client registration conflicts with the LwM2M Server's configuration.</td>
        </tr>
        <tr>
            <td>412 Precondition Failed</td>
            <td>Supported LwM2M Versions of the Server and the Client are not compatible</td>
        </tr>
        <tr>
            <td rowspan="3">Update</td>
            <td>200 OK or 204 No Content</td>
            <td>"Update" operation is completed successfully</td>
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>The mandatory parameter is not specified or unknown parameter is specified</td>
        </tr>
        <tr>
            <td>404 Not Found</td>
            <td>URI of "Update" operation is not found</td>
        </tr>
        <tr>
            <td rowspan="3">De-register</td>
            <td>200 OK or 204 No Content</td>
            <td>"De-register" operation is completed successfully</td>
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>404 Not Found</td>
            <td>URI of "De-register" operation is not found</td>
        </tr>
    </tbody>
</table>

<table>
    <caption>Response Codes: Device Management and Service Enablement Interface</caption>
    <thead>
        <tr>
            <th>Operations</th>
            <th>Available HTTP Response Codes</th>
            <th>Reason Phrase</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="7">Create</td>
            <td>200 OK or 204 No Content</td>
            <td>"Create" operation is completed successfully</td>
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>Target (i.e., Object) already exists<br>Mandatory Resources are not specified<br>Content Format is not specified</td>
        </tr>
        <tr>
            <td>401 Unauthorized or 403 Forbidden</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>404 Not Found</td>
            <td>URI of "Create" operation is not found</td>
        </tr>
        <tr>
            <td>405 Method Not Allowed</td>
            <td>Target is not allowed for "Create" operation</td>
        </tr>
	<tr>
            <td>400 Bad Request</td>
            <td></td>
        </tr>
        <tr>
            <td>415 Unsupported Media Type </td>
            <td>The specified format is not supported</td>
        </tr>
        <tr>
            <td rowspan="6">Read</td>
            <td>200 OK</td>
            <td>"Read" operation is completed successfully</td>
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>401 Unauthorized</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>404 Not Found</td>
            <td> URI of "Read" operation is not found</td>
        </tr>
        <tr>
            <td>405 Method Not Allowed</td>
            <td>Target is not allowed for "Read" operation</td>
        </tr>
        <tr>
            <td>406 Bad Request</td>
            <td>None of the preferred Content-Formats can be returned</td>
        </tr>
	<tr>
            <td rowspan="7">Read-Composite</td>
            <td>200 OK</td>
            <td>"Read" operation is completed successfully</td>
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>401 Unauthorized or 403 Forbidden</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>404 Not Found</td>
            <td> URI of "Read" operation is not found</td>
        </tr>
        <tr>
            <td>405 Method Not Allowed</td>
            <td>Target is not allowed for "Read" operation</td>
        </tr>
        <tr>
            <td>406 Not Acceptable</td>
            <td>None of the preferred Content-Formats can be returned</td>
        </tr>
	<tr>
            <td>415 Unsupported Media Typet</td>
            <td>The specified format is not supported</td>
        </tr>
        <tr>
            <td rowspan="10">Write</td>
            <td>200 OK or 204 No Content</td>
            <td>"Write" operation is completed successfully</td>
        </tr>
        <tr>
            <td></td>
            <td></td>
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>The format of data to be written is different</td>
        </tr>
        <tr>
            <td>401 Unauthorized or 403 Forbidden</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>404 Not Found </td>
            <td>URI of "Write" operation is not found </td>
        </tr>
        <tr>
            <td>405 Method Not Allowed</td>
            <td>Target is not allowed for "Write" operation </td>
        </tr>
	<tr>
            <td>406 Not Acceptable</td>
            <td></td>
        </tr>
        <tr>
            <td></td>
            <td></td>
        </tr>
        <tr>
            <td>413 Payload Too Large</td>
            <td></td>
        </tr>
        <tr>
            <td>415 Unsupported Media Type</td>
            <td>The specified format is not supported</td>
        </tr>
        <tr>
	<tr>
            <td rowspan="6">Write-Composite</td>
            <td>200 OK or 204 No Content</td>
            <td>"Write" operation is completed successfully</td>
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>The format of data to be written is different</td>
        </tr>
        <tr>
            <td>401 Unauthorized or 403 Forbidden</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>404 Not Found </td>
            <td>URI of "Write" operation is not found </td>
        </tr>
        <tr>
            <td>405 Method Not Allowed</td>
            <td>Target is not allowed for "Write" operation </td>
        </tr>
	<tr>
            <td>406 Not Acceptable</td>
            <td></td>
        </tr>
        <tr>
            <td rowspan="5">Delete</td>
            <td>200 OK or 204 Not Modified</td>
            <td>"Delete" operation is completed successfully</td> 
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>401 Unauthorized or 403 Forbidden </td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>404 Not Found </td>
            <td>URI of "Delete" operation is not found</td>
        </tr>
        <tr>
            <td>405 Method Not Allowed</td>
            <td>Target is not allowed for "Delete" operation</td>
        </tr>
        <tr>
            <td rowspan="5">Execute</td>
            <td>200 OK</td>
            <td>"Execute" operation is completed successfully</td> 
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>The LwM2M Client doesn't understand the argument in the payload</td>
        </tr>
        <tr>
            <td>401 Unauthorized or 403 Forbidden</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>404 Not Found</td>
            <td>URI of "Execute" operation is not found</td>
        </tr>
        <tr>
            <td>405 Method Not Allowed</td>
            <td>Target is not allowed for "Execute" operation</td>
        </tr>
        <tr>
            <td rowspan="5">Write-Attributes</td>
            <td>200 OK or 204 Not Modified</td>
            <td>"Write-Attributes" operation is completed successfully</td> 
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>The format of attribute to be written is different</td>
        </tr>
        <tr>
            <td>401 Unauthorized or 403 Forbidden</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>404 Not Found </td>
            <td>URI of "Write-Attributes" operation is not found</td>
        </tr>
        <tr>
            <td>405 Method Not Allowed</td>
            <td>Target is not allowed for Write-Attributes operation</td>
        </tr>
        <tr>
            <td rowspan="5">Discover</td>
            <td>200 OK</td>
            <td>"Discover" operation is completed successfully</td> 
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>401 Unauthorized or 403 Forbidden</td>
            <td>Access Right Permission Denied</td>
        </tr>
        <tr>
            <td>404 Not Found </td>
            <td>URI of "Discover" operation is not found</td>
        </tr>
        <tr>
            <td>405 Method Not Allowed</td>
            <td>Target is not allowed for Discover operation</td>
        </tr>
    </tbody>
</table>

{:page-break}

<table>
    <caption>Response Codes: Information Reporting Interface</caption> 
    <thead>
        <tr>
            <th>Operations</th>
            <th>Available HTTP Response Codes</th>
            <th>Reason Phrase</th>
        </tr>
    </thead>
    <tbody>
	<tr>
            <td rowspan="5">Send</td>
            <td>200 OK or 204 No Content</td>
            <td>"Send" operation completed successfully</td> 
        </tr>
        <tr>
            <td>400 Bad Request</td>
            <td>Undetermined error occurred</td>
        </tr>
        <tr>
            <td>404 Not Found </td>
            <td>Reported Object was not registered to the LwM2M Server</td>
        </tr>
    </tbody>
</table>

If any operation cannot be completed in the client and the reason cannot be described by a 4xx status code, then a status code from the "Server Error" list of 5xx return codes MUST be returned.

## MQTT Transport Binding

This section defines the MQTT transport binding used by LwM2M interfaces.

To support LwM2M over MQTT this section 
* defines a mapping between the LwM2M interfaces to MQTT messages,
* standardizes a payload structure for use within MQTT messages, 
* highlights the different MQTT deployment options, 
* points to MQTT configuration objects, and
* restricts the use of LwM2M features for use with MQTT. 

The transport of LwM2M over MQTT does not use any features introduced with MQTT v5 and therefore works with both versions, MQTT 3.1.1 \[MQTT\] and \[MQTT5\].

### Communication Behavior

Message Queuing Telemetry Transport (MQTT) is a publish-subscribe-based messaging protocol. An MQTT system consists of MQTT clients communicating with an MQTT server. MQTT servers, as officially called in the MQTT specification, are also known as MQTT brokers. Message routing in MQTT is accomplished via topics. When a publisher has a new data item to distribute, it sends a control message to the connected MQTT server marked with a topic along with the data. The MQTT server then distributes the information to any MQTT clients that have subscribed to that topic. If an MQTT server receives a message addressed to a topic for which there are no current subscribers, it will discard the message unless the publisher indicates that the messages is to be retained. This allows new subscribers to a topic to receive the most current data rather than waiting for the next update from a publisher.

In some transport mappings of LwM2M, the endpoints have a direct connection. To exclude on-path adversaries from eavesdropping on the exchanged LwM2M message it was necessary to use communication security. The MQTT communication architecture also allows authorized MQTT clients to gain access to messages by subscribing to messages matching a specific topic; wildcards relax the exact matching requirement.

When LwM2M Clients belonging to different customers (tenants) are connected to the same MQTT server then the security concern is that a LwM2M Client from customer A can subscribe to messages sent and received by LwM2M Clients of another customer, B. This would allow customer A to learn about the number of devices deployed by customer B, and to see the message contents. There is also the risk that devices deployed by customer A inject messages targeted at the devices of customer B. In the worst case, any device connected to the MQTT server could impersonate a LwM2M Bootstrap-Server. 

To provide security properties of LwM2M over MQTT that are comparable with LwM2M over other transports, such as CoAP over UDP and CoAP over TCP, this specification recommends three techniques: 

* MQTT servers have to be configured to allow specific messages to be forwarded only to dedicated entities. For example, any LwM2M message addressed to a LwM2M Bootstrap-Server should only be forwarded to that server.
* MQTT servers have to be configured to isolate MQTT messages belonging to different tenants (i.e. customers).
* In deployments where multiple LwM2M Servers are used traffic to and from those servers needs to be isolated.  

There are different deployment strategies that impact the security characteristics of LwM2M over MQTT. We present two possible deployments here without restricting the deployment to other configurations. 

In [Figure Colocated MQTT Server / LwM2M Server Deployment]() the MQTT server is co-located with the LwM2M Server and this approach 
closely reassembles the architecture of LwM2M. 

<figure>
      <img src="images/mqtt_architecture.svg" alt="Colocated MQTT Server / LwM2M Server Deployment">
      <figcaption>Colocated MQTT Server / LwM2M Server Deployment</figcaption>
</figure>

In [Figure Independent MQTT Server Deployment]() a deployment is shown where the MQTT server is independently operated from the LwM2M Server and from the LwM2M Bootstrap-Server.

<figure>
      <img src="images/mqtt_architecture2.svg" alt="Independent MQTT Server Deployment">
      <figcaption>Independent MQTT Server Deployment</figcaption>
</figure>

In both deployment cases access control lists at the MQTT server can be used. Where access control lists are consider impractical, or insufficient to meet the security needs of a deployment, this specification also offers a technique for protecting the confidentiality and integrity of application data carried over MQTT.

### Topic Structure

A structured MQTT topic format is used to allow LwM2M Clients and 
LwM2M Servers to subscribe to and obtain those messages they are interested in. 

The structure of the topics, in ABNF format, is defined as follows. In MQTT 
the topic name is an UTF-8 encoded string where wildcard characters ('#', '*', and
'\$' have a special meaning). See Section 3.3.2.1 and Section 1.5.3 of \[MQTT\] 
for more details. The ABNF for UTF-8 encoded strings (and for UTF8-octets) 
is defined in RFC 3629 \[RFC3629\].

```
ENDPOINT  = UTF8-octets
PREFIX    = UTF8-octets

topic = [ PREFIX "/" ] "lwm2m/" ( "bs" / "rd" ) "/" ENDPOINT 
```

The topic structure has to be read as follows: 

* There are several static strings, namely "lwm2m", "/", "bs", and "rd". 
For the bootstrap interface the string "bs" is used; all other interfaces use the 
string "rd". "bs" and "rd" allow an MQTT server to segment traffic towards a LwM2M 
Bootstrap-Server and to a LwM2M Server, respectively. The string "lwm2M" avoids  
topic naming conflicts with non-LwM2M use of the MQTT infrastructure. 

* "/" has a special semantic in MQTT and it serves as a separator in the topic hierarchy. 

* PREFIX is a configuration-defined parameter used to indicate the tenant name, an identifier
for a LwM2M Server, and may contain a deployment-specific string. The PREFIX  
may also be omitted. Including the tenant name is useful in a multi-tenant environment to
allow access control policies to separate traffic of different tenants. Like-wise, in 
a deployment with multiple LwM2M Servers it may be useful to allow access control policies 
to separate traffic to different LwM2M Servers via the inclusion of the LwM2M Server identifier
in the PREFIX. Note that it is possible to run different instances of MQTT servers in 
different containers or virtual machines when there is a concern about misconfiguration and
information leakage between different tenants. Each tenant would therefore use a different 
MQTT server instance. 

* ENDPOINT is a parameter that indicates endpoint name of the LwM2M Client. This parameter 
has to match the endpoint client name used in the registration interface (register operation). 
When no endpoint client name is conveyed in the registration interface then the identifier 
used for the LwM2M Client of the security protocol is used instead. Re-using an existing 
identifier for the LwM2M Client helps to reduce configuration overhead, avoids 
identifier collisions, and enables filtering of messages by the MQTT server.  

### Interface Mappings

Interfaces use messages transmitted to specific topics. The structure and content of the messages depend on the interface used. 
Table 6.1 of [LwM2M-CORE] provides information about the direction of various messages. Many of the messages do, however, follow a regular pattern starting with the operation followed by a token value. The operation indicates the message type while the token is used for matching a response to a request. The value of the token needs to be chosen such that responses can be mapped unambiguously to a request. There are no randomness requirements for the token value. 

Messages are encoded in CBOR. The sections below describe what payload content is used for each interface. 
Note: The parameters {ENDPOINT} and {PREFIX} have to be replaced with the respective string used in a given deployment. 

The publish / subscribe model used by the LwM2M interfaces is defined as follows: 

* A LwM2M Server subscribes to "{PREFIX}/lwm2m/rd/#" to receive messages from LwM2M Clients. A LwM2M Server publishes responses to a specific endpoint, {ENDPOINT}, via "{PREFIX}/lwm2m/rd/{ENDPOINT}".

* A LwM2M Client subscribes to "{PREFIX}/lwm2m/rd/{ENDPOINT}" to receive messages from LwM2M Servers for processing by a respective LwM2M Client with a given endpoint name. To publish to "{PREFIX}/lwm2m/rd/{ENDPOINT}" to send a message to a LwM2M Server.  

The interaction with a LwM2M Bootstrap-Server is similar with the exception that the topic name includes the "bs" string (instead of "rd") to indicate the bootstrap interface.

#### Bootstrap Interface

The bootstrap interface uses the following operations (along with the code):

- Bootstrap-Request (0)
- Bootstrap-Write (1)
- Bootstrap-Read (2)
- Bootstrap-Delete (3)
- Bootstrap-Discover (4)
- Bootstrap-Finish (5)
- Bootstrap-Pack-Request (6)

The bootstrap interface uses the following parameters:

- Endpoint Client Name (ep)
- Preferred Content Format (pct)

The payload structure is defined as follows: 

```
Bootstrap_Payload = (
    operation : int,
    token : int,
    ? ep :  bstr,
    ? pct : bstr, 
    ? paths : bstr,
    ? payload: bstr,
    * $$extensions
)
```

The paths parameter is used to encode the URI paths in SenML CBOR format (without the value fields). 

The following example shows a Bootstrap-Request message: 

```
{
  "operation" : 0, // Bootstrap-Request
  "token" : 42
}
```

#### Registration Interface

The registration interface uses the following operations (along with the code) 

- Register (6)
- Update (7)
- De-register (8)

The registration interface uses the following operations:

- Endpoint Client Name (ep)
- Lifetime (lt)
- version (lwm2m)
- binding (b)
- MSISDN (sms)
- profile ID (pid)

The payload contains links, according to the definition of the registration interface, 
in CoRE Link format.

The payload structure is defined as follows: 

```
Registration_Payload = (
    operation : int,
    token : int,
    ? ep :  bstr,
    ? lt : int, 
    ? version : bstr, 
    ? b : bstr, 
    ? sms: bstr, 
    ? pid: bstr,
    ? payload: bstr,
    * $$extensions
)   
```

The following example in CBOR diagnostic format shows a LwM2M Client posting a Register to 
tenant-a/lwm2m/rd/b1cccdea-22ca-4448-bcf7-d07317ee0361 (whereby "tentant-a" indicates the 
name of the tenant and "b1cccdea-22ca-4448-bcf7-d07317ee0361" is the endpoint name of the LwM2M Client).

```
{
  "operation" : 0, // Register
  "token" : 34
  "version" : "1.2",
  "lt" : 3600,
  "payload" : a8393..8aa8a, // "</1/0>,</1/1>,</3/0>,</4/0>,</5/0>"
}
```

#### Device Management & Service Enablement Interface

The device management & service enablement interface uses the following operations:

- Read (9)
- Read-Composite (10)
- Discover (11)
- Write Replace (12)
- Write Partial Update (13)
- Write-Attributes (14)
- Write-Composite (15)
- Execute (16)
- Create (17)
- Write-Attributes (18)
- Delete (19)

The device management & service enablement interface uses the following parameters:

- content-type of the payload if any (ct)
- minimum period (pmin)
- maximum period (pmax)
- greater than (gt)
- less than (lt)
- step (st)
- minimum evaluation period (epmin)
- maximum evaluation period (epmax)

The payload structure is defined as follows: 

```
DMSE_Payload = (
    operation : int,
    token : int,
    ? pmin :  int,
    ? pmax : int, 
    ? gt : int, 
    ? lt : int, 
    ? st: int, 
    ? epmin : int, 
    ? epmax : int, 
    ? paths : bstr,
    ? ct : int,
    ? payload : bstr,
    * $$extensions
)
```

#### Information Reporting Interface 

The information reporting interface uses the following operations:

- Observe (19)
- Cancel-Observe (20)
- Notify (21)
- Send (22)

```
IR_Payload = (
    operation : int,
    token : int,
    ? paths : bstr,
    ? ct : int,
    ? payload : bstr,
    * $$extensions
)
```

### Generic Response Payload Structure

The response codes are contained in the result payload. 
The payload has the following structure: 

```
Generic_Response_Payload = (
    result : bstr,
    token : int,
    ? ct : int,
    ? payload : bstr,
    * $$extensions   
)
```

An example of a LwM2M Server returning a response message to 
tenant-a/lwm2m/rd/b1cccdea-22ca-4448-bcf7-d07317ee0361
response is shown below: 

```
{
  "result" : "2.05", 
  "token" : 34
}
```

### Security Protection 

MQTT exchanges are protected at the transport layer between MQTT clients and MQTT servers using TLS, as defined by the MQTT specification. 
Confidentiality protection may be used to ensure that message exchanges between a LwM2M Client and a LwM2M Server as well as between a LwM2M Client and a 
LwM2M Bootstrap-Server are not visible to the MQTT server itself. MQTT application data payloads may be encrypted also to prevent MQTT clients other than the intented recipients from reading messages.  

The following CDDL structure defines the payload structure for CBOR protection.

```
   Outer_Wrapper = {
       msg-wrapper         => bstr .cbor
                              Msg_AuthEnc_Wrapper / nil,
       lw2m2-message      => (Bootstrap_Payload /
                              Registration_Payload /
                              DMSE_Payload /                                       
                              IR_Payload /
                              Generic_Response_Payload ),
   }

   msg-wrapper = 1
   lwm2m-message = 2

   Msg_Wrapper = [ * (COSE_Encrypt / COSE_Encrypt0 )]
```

### CBOR Mapping

If CBOR is used, the parameters used in the messages 
MUST be mapped to CBOR types as specified in the table below.

<table>
    <caption>CBOR Mapping</caption> 
    <thead>
        <tr>
            <th>Name</th>
            <th>CBOR Key</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>operation</td>
            <td>1</td>
        </tr>
        <tr>
            <td>token</td>
            <td>2</td>
        </tr>
        <tr>
            <td>ep</td>
            <td>3</td>
        </tr>
        <tr>
            <td>pct</td>
            <td>4</td>
        </tr>
        <tr>
            <td>paths</td>
            <td>5</td>
        </tr>
        <tr>
            <td>payload</td>
            <td>6</td>
        </tr>
        <tr>
            <td>lt</td>
            <td>7</td>
        </tr>
        <tr>
            <td>version</td>
            <td>8</td>
        </tr>
        <tr>
            <td>b</td>
            <td>9</td>
        </tr>
        <tr>
            <td>sms</td>
            <td>10</td>
        </tr>
        <tr>
            <td>pmin</td>
            <td>11</td>
        </tr>
        <tr>
            <td>pmax</td>
            <td>12</td>
        </tr>
        <tr>
            <td>gt</td>
            <td>13</td>
        </tr>
        <tr>
            <td>st</td>
            <td>14</td>
        </tr>
        <tr>
            <td>epmin</td>
            <td>15</td>
        </tr>
        <tr>
            <td>epmax</td>
            <td>16</td>
        </tr>
        <tr>
            <td>result</td>
            <td>17</td>
        </tr>
        <tr>
            <td>ct</td>
            <td>18</td>
        </tr>	    
    </tbody>
</table>

### MQTT Configuration 

An LwM2M Client needs to have security credential and configuration information to interact with an MQTT server. The LwM2M Security Object already contains the Server URI as well as various security options for use with TLS. When LwM2M is used over MQTT then the LwM2M Server URI MUST be an mqtts: or mqtt: URI. This specification does not use the user name and the password fields in the MQTT CONNECT message with MQTT and hence those fields are not configurable. The MQTT CONNECT message may carry a Client Identifier (ClientId) and, if it does, then this value MUST be populated with the value of the endpoint client name unless further information is provided in the MQTT Server Object. Note that the MQTT specification places a restriction on the length of the string (1..23 UTF-8 encoded bytes) and the allowed character set that MQTT servers must support. They may support longer strings with other character sets though. Please refer to Section 3.1.3.1 of \[MQTT\] for more details. The use of the Client Identifier Resource of the MQTT Server Object is a safe way to know about what functionality to use with an MQTT server. 

The MQTT protocol can optionally be configured to support a certain behavior, for example different quality of service classes. These settings are provided by a newly defined MQTT Server object.

If COSE is used to protect MQTT messages between MQTT clients then security credentials need to be available. This information is available through a newly defined MQTT Client object.
