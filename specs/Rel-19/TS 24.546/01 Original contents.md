---
spec: TS 24.546
version: 19.5.0
release: '19'
clause: contents
title: Contents
source_archive: 24546-j50.zip
source_document: 24546-j50.docx
content_origin: 3gpp-source
---

# Contents

Foreword 7

1 Scope 9

2 References 9

3 Definitions of terms and abbreviations 10

3.1 Terms 10

3.2 Abbreviations 11

4 General description 11

5 Functional entities 11

5.1 SEAL configuration management client (SCM-C) 11

5.2 SEAL configuration management server (SCM-S) 12

6 Configuration management procedures 13

6.1 General 13

6.2 On-network procedures 13

6.2.1 General 13

6.2.1.1 Authenticated identity in HTTP request 13

6.2.1.2 Authenticated identity in CoAP request 13

6.2.2 Common procedures 13

6.2.2.1 Management of configuration update event subscription 13

6.2.2.1.1 SIP based procedures 13

6.2.2.1.2 HTTP based procedures 15

6.2.2.1.3 CoAP based procedures 16

6.2.2.2 Notifications 17

6.2.2.2.1 SIP based procedures 17

6.2.2.2.2 HTTP based procedures 17

6.2.2.2.3 CoAP based procedures 18

6.2.3 VAL UE configuration data 18

6.2.3.1 SCM client HTTP procedure 18

6.2.3.2 SCM server HTTP procedure 18

6.2.3.3 SCM client CoAP procedure 19

6.2.3.4 SCM server CoAP procedure 19

6.2.4 VAL user profile data 20

6.2.4.1 SCM client HTTP procedure 20

6.2.4.2 SCM server HTTP procedure 20

6.2.4.3 SCM client CoAP procedure 20

6.2.4.4 SCM server CoAP procedure 20

6.2.5 Update VAL user profile data 21

6.2.5.1 SCM client HTTP procedure 21

6.2.5.2 SCM server HTTP procedure 21

6.2.5.3 SCM client CoAP procedure 22

6.2.5.4 SCM server CoAP procedure 22

6.2.6 Application satellite coverage information provisioning 22

6.2.6.1 SCM client HTTP procedure 22

6.2.6.2 SCM server HTTP procedure 23

6.2.6.3 SCM client CoAP procedure 23

6.2.6.4 SCM server CoAP procedure 24

6.2.7 UE requesting the application satellite coverage availability information 24

6.2.7.1 SCM client HTTP procedure 24

6.2.7.2 SCM server HTTP procedure 24

6.2.7.3 SCM client CoAP procedure 25

6.2.7.4 SCM server CoAP procedure 25

6.3 Off-network procedures 25

7 Coding 26

7.1 VAL user profile document 26

7.1.1 General 26

7.1.2 Application unique ID 26

7.1.3 Data structure 26

7.1.4 XML Schema 26

7.1.5 Semantics 27

7.1.6 MIME type 27

7.1.7 IANA registration template 27

7.2 VAL UE configuration document 29

7.2.1 General 29

7.2.2 Application unique ID 29

7.2.3 Data structure 29

7.2.4 XML schema 30

7.2.5 Semantics 31

7.2.6 MIME type 32

7.2.7 IANA registration template 32

7.3 VAL UE satellite information document 34

7.3.1 General 34

7.3.2 Application unique ID 34

7.3.3 Data structure 34

7.3.4 XML schema 34

7.3.5 Semantics 36

7.3.6 MIME type 37

7.3.7 IANA registration template 37

Annex A (normative): Parameters for different operations 40

A.1 Creating configuration update event subscription 40

A.1.1 General 40

A.1.2 Client side parameters 40

A.1.3 Server side parameters 40

A.2 Retrieve VAL UE configuration data 41

A.2.1 Client side parameters 41

Annex B (normative): Parameters for notifications 42

B.1 General 42

B.2 Configuration update notification 42

Annex C (normative): CoAP resource representation and encoding 43

C.1 General 43

C.1.1 Resource URI structure 43

C.1.2 Use of cache 43

C.1.3 Error handling 43

C.1.4 Data types applicable to multiple resource representations 46

C.1.4.1 General 46

C.1.4.2 Referenced structured data types 46

C.1.4.3 Referenced simple data types and enumerations 46

C.1.4.4 Common structured data types 48

C.1.4.4.1 Type: ScheduledCommunicationTime 48

C.1.4.4.2 Type: ProblemDetails 48

C.1.4.4.3 Type: GeographicalCoordinates 48

C.1.4.4.4 Type: GeographicArea 49

C.1.4.4.5 Type: Point 49

C.1.4.4.6 Type: PointUncertaintyCircle 49

C.1.4.4.7 Type: PointUncertaintyEllipse 49

C.1.4.4.8 Type: Polygon 50

C.1.4.4.9 Type: PointAltitude 50

C.1.4.4.10 Type: PointAltitudeUncertainty 50

C.1.4.4.11 Type: EllipsoidArc 50

C.1.4.4.12 Type: UncertaintyEllipse 51

C.1.4.4.13 Type: SatCov 51

C.1.4.5 Common enumerations 51

C.1.4.5.1 Enumeration: SupportedGADShapes 51

C.1.4.5.2 Enumeration: RatType 51

C.2 Resource representation and APIs for VAL user profile 52

C.2.1 SU_UserProfile API 52

C.2.1.1 API URI 52

C.2.1.2 Resources 52

C.2.1.2.1 Overview 52

C.2.1.2.2 Resource: User Profiles 53

C.2.1.2.2.1 Description 53

C.2.1.2.2.2 Resource Definition 53

C.2.1.2.2.3 Resource Standard Methods 53

C.2.1.2.3 Resource: Individual User Profile 54

C.2.1.2.3.1 Description 54

C.2.1.2.3.2 Resource Definition 54

C.2.1.2.3.3 Resource Standard Methods 54

C.2.1.3 Data Model 55

C.2.1.3.1 General 55

C.2.1.3.2 Structured data types 56

C.2.1.3.2.1 Type: ProfileDoc 56

C.2.1.3.2.2 Type: ProfileInfo 56

C.2.1.3.2.3 Type: ProfileConfig 56

C.2.1.3.2.4 Type: ValTargetUe 57

C.2.1.3.3 Simple data types and enumerations 57

C.2.1.3.3.1 Enumeration: ConfigType 57

C.2.1.4 Error Handling 57

C.2.1.5 CDDL Specification 57

C.2.1.5.1 Introduction 57

C.2.1.5.2 CDDL document 57

C.2.1.6 Media Type 58

C.2.1.7 Media Type registration for application/vnd.3gpp.seal-user-profile-info+cbor 58

C.3 Resource representation and APIs for UE configuration 59

C.3.1 SU_UeConfig API 59

C.3.1.1 API URI 59

C.3.1.2 Resources 59

C.3.1.2.1 Overview 59

C.3.1.2.2 Resource: UE Configurations 60

C.3.1.2.2.1 Description 60

C.3.1.2.2.2 Resource Definition 60

C.3.1.2.2.3 Resource Standard Methods 60

C.3.1.2.3 Resource: Individual UE Configuration 61

C.3.1.2.3.1 Description 61

C.3.1.2.3.2 Resource Definition 61

C.3.1.2.3.3 Resource Standard Methods 62

C.3.1.3 Data Model 63

C.3.1.3.1 General 63

C.3.1.3.2 Structured data types 64

C.3.1.3.2.1 Type: UeConfigDoc 64

C.3.1.3.2.2 Type: UeConfig 64

C.3.1.3.2.3 Type: ValUeIds 64

C.3.1.3.2.4 Type: ImeiRange 64

C.3.1.3.2.5 Type: SnrRange 65

C.3.1.3.3 Simple data types and enumerations 65

C.3.1.3.3.1 Simple data types 65

C.3.1.4 Error Handling 65

C.3.1.5 CDDL Specification 65

C.3.1.5.1 Introduction 65

C.3.1.5.2 CDDL document 65

C.3.1.6 Media Type 66

C.3.1.7 Media Type registration for application/vnd.3gpp.seal-ue-config-info+cbor 66

C.4 Resource representation and APIs for application satellite coverage information 67

C.4.1 SU_ASCI API provided by SCM-S 67

C.4.1.1 API URI 67

C.4.1.2 Resources 68

C.4.1.2.1 Overview 68

C.4.1.2.2 Resource: Provision 68

C.4.1.2.2.1 Description 68

C.4.1.2.2.2 Resource Definition 68

C.4.1.2.2.3 Resource Standard Methods 68

C.4.1.3 Data Model 69

C.4.1.3.1 General 69

C.4.1.3.2 Structured data types 69

C.4.1.3.2.1 Type: Provision 69

C.4.1.4 Error Handling 69

C.4.1.5 CDDL Specification 69

C.4.1.5.1 Introduction 69

C.4.1.5.2 CDDL document 70

C.4.2 SU_ASCI API provided by SCM-C 72

C.4.2.1 API URI 72

C.4.2.2 Resources 72

C.4.2.2.1 Overview 72

C.4.2.2.2 Resource: Request 73

C.4.2.2.2.1 Description 73

C.4.2.2.2.2 Resource Definition 73

C.4.2.2.2.3 Resource Standard Methods 73

C.4.2.3 Data Model 73

C.4.2.3.1 General 73

C.4.2.3.2 Structured data types 74

C.4.2.3.2.1 Type: Response 74

C.4.2.4 Error Handling 74

C.4.2.5 CDDL Specification 74

C.4.2.5.1 Introduction 74

C.4.2.5.2 CDDL document 74

C.4.3 Media Type 76

C.4.4 Media Type registration for application/vnd.3gpp.seal-asci+cbor 76

Annex D (informative): Change history 78
