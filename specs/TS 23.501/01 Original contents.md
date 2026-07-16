---
spec: TS 23.501
version: 18.12.0
release: '18'
clause: contents
title: Contents
source_archive: 23501-ic0.zip
source_document: 23501-ic0.docx
source_archive_sha256: 800fb58a2a3a8c3eeac7084497de566fc730808923d8eea61b84d2555360debe
source_document_sha256: cdab3840e4d278d381f70806e6458f87dff8e5fd238f370e4f65872bbd5088a6
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents

Foreword 22

1 Scope 23

2 References 23

3 Definitions and abbreviations 30

3.1 Definitions 30

3.2 Abbreviations 37

4 Architecture model and concepts 41

4.1 General concepts 41

4.2 Architecture reference model 41

4.2.1 General 41

4.2.2 Network Functions and entities 42

4.2.3 Non-roaming reference architecture 43

4.2.4 Roaming reference architectures 46

4.2.5 Data Storage architectures 50

4.2.5a Radio Capabilities Signalling optimisation 52

4.2.6 Service-based interfaces 52

4.2.7 Reference points 53

4.2.8 Support of non-3GPP access 56

4.2.8.0 General 56

4.2.8.1 General Concepts to Support Trusted and Untrusted Non-3GPP Access 56

4.2.8.1A General Concepts to support Wireline Access 57

4.2.8.2 Architecture Reference Model for Trusted and Untrusted Non-3GPP Accesses 58

4.2.8.2.1 Non-roaming Architecture 58

4.2.8.2.2 LBO Roaming Architecture 59

4.2.8.2.3 Home-routed Roaming Architecture 62

4.2.8.3 Reference Points for Non-3GPP Access 64

4.2.8.3.1 Overview 64

4.2.8.3.2 Requirements on Ta 64

4.2.8.4 Architecture Reference Model for Wireline Access network 65

4.2.8.5 Access to 5GC from devices that do not support 5GC NAS over WLAN access 66

4.2.8.5.1 General 66

4.2.8.5.2 Reference Architecture 66

4.2.8.5.3 Network Functions 67

4.2.8.5.4 Reference Points 67

4.2.9 Network Analytics architecture 68

4.2.10 Architecture Reference Model for ATSSS Support 68

4.2.11 Architecture for 5G multicast-broadcast services 69

4.2.12 Architecture for Proximity based Services (ProSe) in 5GS 69

4.2.13 Architecture enhancements for Edge Computing 70

4.2.14 Architecture for Support of Uncrewed Aerial Systems connectivity, identification and tracking 70

4.2.15 Architecture to support WLAN connection using 5G credentials without 5GS registration 70

4.2.16 Architecture to support User Plane Information Exposure via a service-based interface 74

4.2.17 Architecture for Ranging based services and Sidelink Positioning 74

4.3 Interworking with EPC 75

4.3.1 Non-roaming architecture 75

4.3.2 Roaming architecture 75

4.3.3 Interworking between 5GC via non-3GPP access and E-UTRAN connected to EPC 77

4.3.3.1 Non-roaming architecture 77

4.3.3.2 Roaming architecture 78

4.3.4 Interworking between ePDG connected to EPC and 5GS 80

4.3.4.1 Non-roaming architecture 80

4.3.4.2 Roaming architectures 81

4.3.5 Service Exposure in Interworking Scenarios 83

4.3.5.1 Non-roaming architecture 83

4.3.5.2 Roaming architectures 84

4.4 Specific services 85

4.4.1 Public Warning System 85

4.4.2 SMS over NAS 85

4.4.2.0 General 85

4.4.2.1 Architecture to support SMS over NAS 85

4.4.2.2 Reference point to support SMS over NAS 87

4.4.2.3 Service based interface to support SMS over NAS 87

4.4.3 IMS support 87

4.4.4 Location services 88

4.4.4.1 Architecture to support Location Services 88

4.4.4.2 Reference point to support Location Services 88

4.4.4.3 Service Based Interfaces to support Location Services 88

4.4.5 Application Triggering Services 88

4.4.6 5G LAN-type Services 88

4.4.6.1 User plane architecture to support 5G LAN-type service 88

4.4.6.2 Reference points to support 5G LAN-type service 89

4.4.7 MSISDN-less MO SMS Service 89

4.4.8 Architecture to enable Time Sensitive Communication, Time Synchronization and Deterministic Networking 89

4.4.8.1 General 89

4.4.8.2 Architecture to support IEEE Time Sensitive Networking 90

4.4.8.3 Architecture for AF requested support of Time Sensitive Communication and Time Synchronization 91

4.4.8.4 Architecture to support IETF Deterministic Networking 91

5 High level features 92

5.1 General 92

5.2 Network Access Control 93

5.2.1 General 93

5.2.2 Network selection 93

5.2.2a Void 93

5.2.3 Identification and authentication 93

5.2.4 Authorisation 93

5.2.5 Access control and barring 93

5.2.6 Policy control 94

5.2.7 Lawful Interception 94

5.3 Registration and Connection Management 94

5.3.1 General 94

5.3.2 Registration Management 94

5.3.2.1 General 94

5.3.2.2 5GS Registration Management states 95

5.3.2.2.1 General 95

5.3.2.2.2 RM-DEREGISTERED state 95

5.3.2.2.3 RM-REGISTERED state 95

5.3.2.2.4 5GS Registration Management State models 96

5.3.2.3 Registration Area management 96

5.3.2.4 Support of a UE registered over both 3GPP and Non-3GPP access 98

5.3.3 Connection Management 99

5.3.3.1 General 99

5.3.3.2 5GS Connection Management states 100

5.3.3.2.1 General 100

5.3.3.2.2 CM-IDLE state 100

5.3.3.2.3 CM-CONNECTED state 100

5.3.3.2.4 5GS Connection Management State models 101

5.3.3.2.5 CM-CONNECTED with RRC_INACTIVE state 101

5.3.3.3 NAS signalling connection management 104

5.3.3.3.1 General 104

5.3.3.3.2 NAS signalling connection establishment 104

5.3.3.3.3 NAS signalling connection Release 104

5.3.3.4 Support of a UE connected over both 3GPP and Non-3GPP access 104

5.3.4 UE Mobility 105

5.3.4.1 Mobility Restrictions 105

5.3.4.1.1 General 105

5.3.4.1.2 Management of Service Area Restrictions 108

5.3.4.2 Mobility Pattern 109

5.3.4.3 Radio Resource Management functions 109

5.3.4.3.1 General 109

5.3.4.3.2 Preferred band(s) per data radio bearer(s) 110

5.3.4.3.3 Redirection to dedicated frequency band(s) for an S-NSSAI 110

5.3.4.3.4 Network Slice based cell reselection and Random Access 111

5.3.4.4 UE mobility event notification 112

5.3.5 Triggers for network analytics 113

5.4 3GPP access specific aspects 114

5.4.1 UE reachability in CM-IDLE 114

5.4.1.1 General 114

5.4.1.2 UE reachability allowing mobile terminated data while the UE is CM-IDLE 115

5.4.1.3 Mobile Initiated Connection Only (MICO) mode 115

5.4.1.4 Support of Unavailability Period 116

5.4.2 UE reachability in CM-CONNECTED 118

5.4.3 Paging strategy handling 118

5.4.3.1 General 118

5.4.3.2 Paging Policy Differentiation 118

5.4.3.3 Paging Priority 119

5.4.4 UE Radio Capability handling 119

5.4.4.1 UE radio capability information storage in the AMF 119

5.4.4.1a UE radio capability signalling optimisation (RACS) 120

5.4.4.2 Void 124

5.4.4.2a UE Radio Capability Match Request 124

5.4.4.3 Paging assistance information 125

5.4.4a UE MM Core Network Capability handling 125

5.4.4b UE 5GSM Core Network Capability handling 127

5.4.5 DRX (Discontinuous Reception) framework 127

5.4.6 Core Network assistance information for RAN optimization 127

5.4.6.1 General 127

5.4.6.2 Core Network assisted RAN parameters tuning 128

5.4.6.3 Core Network assisted RAN paging information 129

5.4.7 NG-RAN location reporting 129

5.4.8 Support for identification and restriction of using unlicensed spectrum 129

5.4.9 Wake Up Signal Assistance 130

5.4.9.1 General 130

5.4.9.2 Group Wake Up Signal 131

5.4.10 Support for identification and restriction of using NR satellite access 131

5.4.11 Support for integrating NR satellite access into 5GS 132

5.4.11.1 General 132

5.4.11.2 Support of RAT types defined in 5GC for satellite access 132

5.4.11.3 Void 132

5.4.11.4 Verification of UE location 132

5.4.11.5 Network selection for NR satellite access 133

5.4.11.6 Support of Mobility Registration Update 133

5.4.11.7 Tracking Area handling for NR satellite access 133

5.4.11.8 Support for mobility Forbidden Area and Service Area Restrictions for NR satellite access 133

5.4.12 Paging Early Indication with Paging Subgrouping Assistance 134

5.4.12.1 General 134

5.4.12.2 Core Network Assistance for PEIPS 134

5.4.13 Support of discontinuous network coverage for satellite access 135

5.4.13.0 General 135

5.4.13.1 Mobility Management and Power Saving Optimization 135

5.4.13.2 Coverage availability information provisioning to the UE 136

5.4.13.3 Coverage availability information provisioning to the AMF 136

5.4.13.4 Paging 136

5.4.13.5 Overload control 136

5.5 Non-3GPP access specific aspects 137

5.5.0 General 137

5.5.1 Registration Management 137

5.5.2 Connection Management 138

5.5.3 UE Reachability 139

5.5.3.1 UE reachability in CM-IDLE 139

5.5.3.2 UE reachability in CM-CONNECTED 139

5.6 Session Management 140

5.6.1 Overview 140

5.6.2 Interaction between AMF and SMF 143

5.6.3 Roaming 145

5.6.4 Single PDU Session with multiple PDU Session Anchors 145

5.6.4.1 General 145

5.6.4.2 Usage of an UL Classifier for a PDU Session 146

5.6.4.3 Usage of IPv6 multi-homing for a PDU Session 148

5.6.5 Support for Local Area Data Network 149

5.6.5a Supporting LADN per DNN and S-NSSAI 151

5.6.6 Secondary authentication/authorization by a DN-AAA server during the establishment of a PDU Session 152

5.6.7 Application Function influence on traffic routing 154

5.6.7.1 General 154

5.6.7.2 Enhancement of UP path management based on the coordination with AFs 164

5.6.8 Selective activation and deactivation of UP connection of existing PDU Session 165

5.6.9 Session and Service Continuity 166

5.6.9.1 General 166

5.6.9.2 SSC mode 166

5.6.9.2.1 SSC Mode 1 166

5.6.9.2.2 SSC Mode 2 166

5.6.9.2.3 SSC Mode 3 167

5.6.9.3 SSC mode selection 167

5.6.10 Specific aspects of different PDU Session types 168

5.6.10.1 Support of IP PDU Session type 168

5.6.10.2 Support of Ethernet PDU Session type 168

5.6.10.3 Support of Unstructured PDU Session type 170

5.6.10.4 Maximum Transfer Unit size considerations 171

5.6.11 UE presence in Area of Interest reporting usage by SMF 171

5.6.12 Use of Network Instance 173

5.6.13 Always-on PDU session 173

5.6.14 Support of Framed Routing 174

5.6.15 Triggers for network analytics 174

5.6.16 Support for Service Function Chaining 175

5.6.16.1 General 175

5.6.16.2 Application Function influence on Service Function Chaining 175

5.7 QoS model 176

5.7.1 General Overview 176

5.7.1.1 QoS Flow 176

5.7.1.2 QoS Profile 177

5.7.1.2a Alternative QoS Profile 178

5.7.1.3 Control of QoS Flows 178

5.7.1.4 QoS Rules 178

5.7.1.5 QoS Flow mapping 179

5.7.1.6 DL traffic 181

5.7.1.7 UL Traffic 181

5.7.1.8 AMBR/MFBR enforcement and rate limitation 182

5.7.1.9 Precedence Value 182

5.7.1.10 UE-Slice-MBR enforcement and rate limitation 182

5.7.1.11 QoS aspects of home-routed roaming 183

5.7.2 5G QoS Parameters 184

5.7.2.1 5QI 184

5.7.2.2 ARP 184

5.7.2.3 RQA 184

5.7.2.4 Notification control 185

5.7.2.4.1 General 185

5.7.2.4.1a Notification Control without Alternative QoS Profiles 185

5.7.2.4.1b Notification control with Alternative QoS Profiles 186

5.7.2.4.2 Usage of Notification control with Alternative QoS Profiles at handover 186

5.7.2.4.3 Usage of Notification control with Alternative QoS Profiles during QoS Flow establishment and modification 187

5.7.2.5 Flow Bit Rates 187

5.7.2.6 Aggregate Bit Rates 188

5.7.2.7 Default values 188

5.7.2.8 Maximum Packet Loss Rate 189

5.7.2.9 Wireline access network specific 5G QoS parameters 189

5.7.3 5G QoS characteristics 189

5.7.3.1 General 189

5.7.3.2 Resource Type 190

5.7.3.3 Priority Level 190

5.7.3.4 Packet Delay Budget 190

5.7.3.5 Packet Error Rate 191

5.7.3.6 Averaging Window 191

5.7.3.7 Maximum Data Burst Volume 192

5.7.4 Standardized 5QI to QoS characteristics mapping 192

5.7.5 Reflective QoS 197

5.7.5.1 General 197

5.7.5.2 UE Derived QoS Rule 198

5.7.5.3 Reflective QoS Control 199

5.7.6 Packet Filter Set 200

5.7.6.1 General 200

5.7.6.2 IP Packet Filter Set 200

5.7.6.3 Ethernet Packet Filter Set 201

5.7.7 PDU Set QoS Parameters 201

5.7.7.1 General 201

5.7.7.2 PDU Set Delay Budget 201

5.7.7.3 PDU Set Error Rate 202

5.7.7.4 PDU Set Integrated Handling Information 202

5.8 User Plane Management 202

5.8.1 General 202

5.8.2 Functional Description 203

5.8.2.1 General 203

5.8.2.2 UE IP Address Management 203

5.8.2.2.1 General 203

5.8.2.2.2 Routing rules configuration 205

5.8.2.2.3 The procedure of Stateless IPv6 Address Autoconfiguration 206

5.8.2.2.4 IPv6 Prefix Delegation via DHCPv6 206

5.8.2.3 Management of CN Tunnel Info 207

5.8.2.3.1 General 207

5.8.2.3.2 Void 207

5.8.2.3.3 Management of CN Tunnel Info in the UPF 207

5.8.2.4 Traffic Detection 207

5.8.2.4.1 General 207

5.8.2.4.2 Traffic Detection Information 207

5.8.2.5 Control of User Plane Forwarding 208

5.8.2.5.1 General 208

5.8.2.5.2 Data forwarding between the SMF and UPF 208

5.8.2.5.3 Support of Ethernet PDU Session type 209

5.8.2.6 Charging and Usage Monitoring Handling 210

5.8.2.6.1 General 210

5.8.2.6.2 Activation of Usage Reporting in UPF 210

5.8.2.6.3 Reporting of Usage Information towards SMF 211

5.8.2.7 PDU Session and QoS Flow Policing 211

5.8.2.8 PCC Related Functions 211

5.8.2.8.1 Activation/Deactivation of predefined PCC rules 211

5.8.2.8.2 Enforcement of Dynamic PCC Rules 212

5.8.2.8.3 Redirection 212

5.8.2.8.4 Support of PFD Management 213

5.8.2.9 Functionality of Sending of "End marker" 213

5.8.2.9.0 Introduction 213

5.8.2.9.1 UPF Constructing the "End marker" Packets 213

5.8.2.9.2 SMF Constructing the "End marker" Packets 214

5.8.2.10 UP Tunnel Management 214

5.8.2.11 Parameters for N4 session management (moved) 214

5.8.2.12 Reporting of the UE MAC addresses used in a PDU Session 215

5.8.2.13 Support for 5G VN group communication 215

5.8.2.13.0 General 215

5.8.2.13.1 Support for unicast traffic forwarding of a 5G VN 216

5.8.2.13.2 Support for unicast traffic forwarding update due to UE mobility 217

5.8.2.13.3 Support for user plane traffic replication in a 5G VN 217

5.8.2.14 Inter PLMN User Plane Security functionality 219

5.8.2.15 Void 220

5.8.2.16 Support for L2TP tunnelling on N6 220

5.8.2.17 Data exposure via Service Based interface 220

5.8.2.18 QoS Flow related QoS monitoring and reporting 221

5.8.2.19 Explicit Buffer Management 222

5.8.2.19.1 General 222

5.8.2.19.2 Buffering at UPF 222

5.8.2.19.3 Buffering at SMF 223

5.8.2.20 SMF Pause of Charging 223

5.8.3 Explicit Buffer Management (moved) 223

5.8.4 SMF Pause of Charging (moved) 223

5.8.5 Parameters for N4 session management 223

5.8.5.1 General 223

5.8.5.2 N4 Session Context 224

5.8.5.3 Packet Detection Rule 225

5.8.5.4 QoS Enforcement Rule 228

5.8.5.5 Usage Reporting Rule 231

5.8.5.6 Forwarding Action Rule 234

5.8.5.7 Usage Report generated by UPF 237

5.8.5.8 Multi-Access Rule 238

5.8.5.9 Bridge/Router Information 239

5.8.5.10 Void 239

5.8.5.11 Session Reporting Rule 240

5.8.5.12 Session reporting generated by UPF 240

5.8.5.13 Void 240

5.8.5.14 TSC Management Information 241

5.8.5.15 Downlink Data Report generated by UPF 241

5.9 Identifiers 241

5.9.1 General 241

5.9.2 Subscription Permanent Identifier 242

5.9.2a Subscription Concealed Identifier 242

5.9.3 Permanent Equipment Identifier 242

5.9.4 5G Globally Unique Temporary Identifier 243

5.9.5 AMF Name 243

5.9.6 Data Network Name (DNN) 243

5.9.7 Internal-Group Identifier 244

5.9.8 Generic Public Subscription Identifier 244

5.9.9 AMF UE NGAP ID 244

5.9.10 UE Radio Capability ID 244

5.10 Security aspects 245

5.10.1 General 245

5.10.2 Security Model for non-3GPP access 245

5.10.2.1 Signalling Security 245

5.10.3 PDU Session User Plane Security 245

5.11 Support for Dual Connectivity, Multi-Connectivity 248

5.11.1 Support for Dual Connectivity 248

5.12 Charging 249

5.12.1 General 249

5.12.2 Usage Data Reporting for Secondary RAT 249

5.12.3 Secondary RAT Periodic Usage Data Reporting Procedure 249

5.13 Support for Edge Computing 250

5.14 Policy Control 251

5.15 Network slicing 251

5.15.1 General 251

5.15.2 Identification and selection of a Network Slice: the S-NSSAI and the NSSAI 252

5.15.2.1 General 252

5.15.2.2 Standardised SST values 253

5.15.3 Subscription aspects 254

5.15.4 UE NSSAI configuration and NSSAI storage aspects 255

5.15.4.1 General 255

5.15.4.1.1 UE Network Slice configuration 255

5.15.4.1.2 Mapping of S-NSSAIs values in the Allowed NSSAI and in the Requested NSSAI to the S-NSSAIs values used in the HPLMN 257

5.15.4.2 Update of UE Network Slice configuration 258

5.15.5 Detailed Operation Overview 259

5.15.5.1 General 259

5.15.5.2 Selection of a Serving AMF supporting the Network Slices 259

5.15.5.2.1 Registration to a set of Network Slices 259

5.15.5.2.2 Modification of the Set of Network Slice(s) for a UE 264

5.15.5.2.3 AMF Re-allocation due to Network Slice(s) Support 266

5.15.5.3 Establishing a PDU Session in a Network Slice 266

5.15.6 Network Slicing Support for Roaming 268

5.15.7 Network slicing and Interworking with EPS 269

5.15.7.1 General 269

5.15.7.2 Idle mode aspects 269

5.15.7.3 Connected mode aspects 270

5.15.7.4 Support of Network Slice usage control and Interworking with EPC 270

5.15.8 Configuration of Network Slice support and availability in a PLMN 270

5.15.9 Operator-controlled inclusion of NSSAI in Access Stratum Connection Establishment 271

5.15.10 Network Slice-Specific Authentication and Authorization 272

5.15.11 Network Slice Admission Control 273

5.15.11.0 General 273

5.15.11.1 Network Slice Admission Control for maximum number of UEs 275

5.15.11.1.1 Non-Hierarchical NSAC architecture 275

5.15.11.1.2 Hierarchical NSAC architecture 275

5.15.11.1.3 Centralized NSAC architecture 277

5.15.11.2 Network Slice Admission Control for maximum number of PDU sessions 277

5.15.11.2.1 Non- Hierarchical NSAC architecture 277

5.15.11.2.2 Hierarchical NSAC architecture 277

5.15.11.2.3 Centralized NSAC architecture 278

5.15.11.3 Network Slice Admission Control for Roaming 278

5.15.11.3.0 General 278

5.15.11.3.1 VPLMN NSAC Admission Mode 278

5.15.11.3.2 VPLMN with HPLMN assistance NSAC Admission 279

5.15.11.3.3 HPLMN NSAC Admission Mode 279

5.15.11.4 Network Slice status notifications and reports to a consumer NF 279

5.15.11.5 Support of Network Slice Admission Control and Interworking with EPC 280

5.15.11.5a Support of Network Slice Admission Control in 5GS for maximum number of UEs with at least one PDU Session/PDN Connection 281

5.15.12 Support of subscription-based restrictions to simultaneous registration of network slices 283

5.15.12.1 General 283

5.15.12.2 UE and UE configuration aspects 283

5.15.13 Support of data rate limitation per Network Slice for a UE 284

5.15.14 Network Slice AS Groups support 285

5.15.15 Support of Network Slice usage control 285

5.15.15.1 General 285

5.15.15.2 UE Configuration of network-controlled Slice Usage Policy 286

5.15.15.3 Network-based per UE Network Slice usage behaviour control 287

5.15.16 Optimized handling of temporarily available network slices 287

5.15.17 Partial Network Slice support in a Registration Area 289

5.15.18 Support for Network Slices with Network Slice Area of Service not matching deployed Tracking Areas 292

5.15.18.1 General 292

5.15.18.2 S-NSSAI location availability information 292

5.15.18.3 Network based monitoring and enforcement of Network Slice Area of Service not matching deployed Tracking Areas 293

5.15.19 Support of Network Slice Replacement 294

5.15.20 Support of Network Slice Instance Replacement 297

5.16 Support for specific services 297

5.16.1 Public Warning System 297

5.16.2 SMS over NAS 297

5.16.2.1 General 297

5.16.2.2 SMS over NAS transport 298

5.16.3 IMS support 298

5.16.3.1 General 298

5.16.3.2 IMS voice over PS Session Supported Indication over 3GPP access 298

5.16.3.2a IMS voice over PS Session Supported Indication over non-3GPP access 299

5.16.3.3 Homogeneous support for IMS voice over PS Session supported indication 299

5.16.3.4 P-CSCF address delivery 300

5.16.3.5 Domain selection for UE originating sessions / calls 300

5.16.3.6 Terminating domain selection for IMS voice 301

5.16.3.7 UE's usage setting 301

5.16.3.8 Domain and Access Selection for UE originating SMS 301

5.16.3.8.1 UE originating SMS for IMS Capable UEs supporting SMS over IP 301

5.16.3.8.2 Access Selection for SMS over NAS 301

5.16.3.9 SMF support for P-CSCF restoration procedure 302

5.16.3.10 IMS Voice Service via EPS Fallback or RAT fallback in 5GS 302

5.16.3.11 P-CSCF discovery and selection 302

5.16.3.12 HSS discovery and selection 303

5.16.4 Emergency Services 303

5.16.4.1 Introduction 303

5.16.4.2 Architecture Reference Model for Emergency Services 306

5.16.4.3 Mobility Restrictions and Access Restrictions for Emergency Services 306

5.16.4.4 Reachability Management 307

5.16.4.5 SMF and UPF selection function for Emergency Services 307

5.16.4.6 QoS for Emergency Services 307

5.16.4.7 PCC for Emergency Services 307

5.16.4.8 IP Address Allocation 307

5.16.4.9 Handling of PDU Sessions for Emergency Services 307

5.16.4.9a Handling of PDU Sessions for normal services for Emergency Registered UEs 308

5.16.4.10 Support of eCall Only Mode 308

5.16.4.11 Emergency Services Fallback 308

5.16.5 Multimedia Priority Services 309

5.16.6 Mission Critical Services 311

5.17 Interworking and Migration 312

5.17.1 Support for Migration from EPC to 5GC 312

5.17.1.1 General 312

5.17.1.2 User Plane management to support interworking with EPS 314

5.17.1.3 QoS handling for home routed roaming 314

5.17.2 Interworking with EPC 314

5.17.2.1 General 314

5.17.2.2 Interworking Procedures with N26 interface 318

5.17.2.2.1 General 318

5.17.2.2.2 Mobility for UEs in single-registration mode 319

5.17.2.3 Interworking Procedures without N26 interface 320

5.17.2.3.1 General 320

5.17.2.3.2 Mobility for UEs in single-registration mode 321

5.17.2.3.3 Mobility for UEs in dual-registration mode 321

5.17.2.3.4 Redirection for UEs in connected state 323

5.17.2.4 Mobility between 5GS and GERAN/UTRAN 323

5.17.2.5 Secondary DN authentication and authorization in EPS Interworking case 324

5.17.3 Interworking with EPC in presence of Non-3GPP PDU Sessions 324

5.17.4 Network sharing support and interworking between EPS and 5GS 325

5.17.5 Service Exposure in Interworking Scenarios 325

5.17.5.1 General 325

5.17.5.2 Support of interworking for Monitoring Events 326

5.17.5.2.1 Interworking with N26 interface 326

5.17.5.2.2 Interworking without N26 interface 326

5.17.5.3 Availability or expected level of a service API 326

5.17.6 Void 327

5.17.7 Configuration Transfer Procedure between NG-RAN and E-UTRAN 327

5.17.7.1 Architecture Principles for Configuration Transfer between NG-RAN and E-UTRAN 327

5.17.7.2 Addressing, routing and relaying 328

5.17.7.2.1 Addressing 328

5.17.7.2.2 Routing 328

5.17.7.2.3 Relaying 329

5.17.8 URSP Provisioning in EPS 329

5.18 Network Sharing 331

5.18.1 General concepts 331

5.18.2 Broadcast system information for network sharing 332

5.18.2a PLMN list and SNPN list handling for network sharing 332

5.18.3 Network selection by the UE 332

5.18.4 Network selection by the network 333

5.18.5 Network Sharing and Network Slicing 334

5.19 Control Plane Load Control, Congestion and Overload Control 334

5.19.1 General 334

5.19.2 TNLA Load Balancing and TNLA Load Re-Balancing 334

5.19.3 AMF Load Balancing 334

5.19.4 AMF Load Re-Balancing 334

5.19.5 AMF Control Of Overload 335

5.19.5.1 General 335

5.19.5.2 AMF Overload Control 335

5.19.6 SMF Overload Control 336

5.19.7 NAS level congestion control 336

5.19.7.1 General 336

5.19.7.2 General NAS level congestion control 337

5.19.7.3 DNN based congestion control 338

5.19.7.4 S-NSSAI based congestion control 339

5.19.7.5 Group specific NAS level congestion control 341

5.19.7.6 Control Plane data specific NAS level congestion control 341

5.20 External Exposure of Network Capability 342

5.20a Data Collection from an AF 343

5.20b Support exposure of DNN and S-NSSAI specific Group Parameters 344

5.20b.1 Group attribute provisioning 344

5.20b.2 Support LADN service area for a group 344

5.20b.3 Support QoS for a group 344

5.20b.4 Void 344

5.20b.5 Void 344

5.20c Provisioning of traffic characteristics and monitoring of performance characteristics for a group 345

5.20d User Plane Direct 5GS Information Exposure 345

5.20d.1 General 345

5.21 Architectural support for virtualized deployments 346

5.21.0 General 346

5.21.1 Architectural support for N2 346

5.21.1.1 TNL associations 346

5.21.1.2 NGAP UE-TNLA-binding 347

5.21.1.3 N2 TNL association selection 347

5.21.2 AMF Management 347

5.21.2.1 AMF Addition/Update 347

5.21.2.2 AMF planned removal procedure 348

5.21.2.2.1 AMF planned removal procedure with UDSF deployed 348

5.21.2.2.2 AMF planned removal procedure without UDSF 349

5.21.2.3 Procedure for AMF Auto-recovery 351

5.21.3 Network Reliability support with Sets 352

5.21.3.1 General 352

5.21.3.2 NF Set and NF Service Set 353

5.21.3.3 Reliability of NF instances within the same NF Set 353

5.21.3.4 Reliability of NF Services 353

5.21.4 Network Function/NF Service Context Transfer 353

5.21.4.1 General 353

5.22 System Enablers for priority mechanism 354

5.22.1 General 354

5.22.2 Subscription-related Priority Mechanisms 354

5.22.3 Invocation-related Priority Mechanisms 355

5.22.4 QoS Mechanisms applied to established QoS Flows 356

5.23 Supporting for Asynchronous Type Communication 357

5.24 3GPP PS Data Off 357

5.25 Support of OAM Features 358

5.25.1 Support of Tracing: Signalling Based Activation/Deactivation of Tracing 358

5.25.2 Support of OAM-based 5G VN group management 359

5.25.3 Signalling Based Activation of QoE Measurement Collection 359

5.26 Configuration Transfer Procedure 359

5.26.1 Architecture Principles for Configuration Transfer 359

5.26.2 Addressing, routing and relaying 360

5.26.2.1 Addressing 360

5.26.2.2 Routing 360

5.26.2.3 Relaying 360

5.27 Enablers for Time Sensitive Communications, Time Synchronization and Deterministic Networking 360

5.27.0 General 360

5.27.1 Time Synchronization 361

5.27.1.1 General 361

5.27.1.2 Distribution of timing information 362

5.27.1.2.1 Distribution of 5G internal system clock 362

5.27.1.2.2 Distribution of grandmaster clock and time-stamping 362

5.27.1.3 Support for multiple (g)PTP domains 365

5.27.1.4 DS-TT and NW-TT Time Synchronization functionality 366

5.27.1.5 Detection of (g)PTP Sync and Announce timeouts 367

5.27.1.6 Distribution of Announce messages and best master clock selection 367

5.27.1.7 Support for PTP grandmaster function in 5GS 368

5.27.1.8 Exposure of Time Synchronization 368

5.27.1.9 Support for derivation of Uu time synchronization error budget 370

5.27.1.10 Support for coverage area filters for time synchronization service 370

5.27.1.11 Controlling time synchronization service based on the Subscription 371

5.27.1.12 Support for network timing synchronization status monitoring 374

5.27.1a Periodic deterministic communication 377

5.27.2 TSC Assistance Information (TSCAI) and TSC Assistance Container (TSCAC) 378

5.27.2.1 General 378

5.27.2.2 TSC Assistance Container determination based on PSFP 379

5.27.2.3 TSC Assistance Container determination by TSCTSF 380

5.27.2.4 TSCAI determination based on TSC Assistance Container 381

5.27.2.5 RAN feedback for Burst Arrival Time offset and adjusted Periodicity 382

5.27.2.5.1 Overview 382

5.27.2.5.2 Proactive RAN feedback for adaptation of Burst Arrival Time and Periodicity 382

5.27.2.5.3 Reactive RAN feedback for Burst Arrival Time adaptation 383

5.27.3 Support for TSC QoS Flows 383

5.27.4 Hold and Forward Buffering mechanism 384

5.27.5 5G System Bridge delay 384

5.28 Support of integration with TSN, Time Sensitive Communications, Time Synchronization and Deterministic Networking 385

5.28.0 General 385

5.28.1 5GS bridge management for TSN 385

5.28.2 5GS Bridge configuration for TSN 387

5.28.3 Port and user plane node management information exchange in 5GS 389

5.28.3.1 General 389

5.28.3.2 Transfer of port or user plane node management information 390

5.28.3.3 VLAN Configuration Information for TSN 391

5.28.4 QoS mapping tables for TSN 391

5.28.5 Support of integration with IETF Deterministic Networking 393

5.28.5.1 General 393

5.28.5.2 5GS DetNet node reporting 393

5.28.5.3 DetNet node configuration mapping in 5GS 394

5.28a Support for TSN enabled Transport Network 395

5.28a.1 General 395

5.28a.2 Transfer of TL-Container between SMF/CUC and AN-TL and CN-TL 395

5.28a.3 Topology Information for TSN TN 396

5.29 Support for 5G LAN-type service 396

5.29.1 General 396

5.29.2 5G VN group management 397

5.29.3 PDU Session management 398

5.29.4 User Plane handling 399

5.30 Support for non-public networks 400

5.30.1 General 400

5.30.2 Stand-alone Non-Public Networks 401

5.30.2.0 General 401

5.30.2.1 Identifiers 401

5.30.2.2 Broadcast system information 402

5.30.2.3 UE configuration and subscription aspects 403

5.30.2.4 Network selection in SNPN access mode 405

5.30.2.4.1 General 405

5.30.2.4.2 Automatic network selection 406

5.30.2.4.3 Manual network selection 408

5.30.2.5 Network access control 408

5.30.2.6 Cell (re-)selection in SNPN access mode 408

5.30.2.7 Access to PLMN services via stand-alone non-public networks 408

5.30.2.8 Access to stand-alone non-public network services via PLMN 409

5.30.2.9 SNPN connectivity for UEs with credentials owned by Credentials Holder 409

5.30.2.9.1 General 409

5.30.2.9.2 Credentials Holder using AAA Server for primary authentication and authorization 410

5.30.2.9.3 Credentials Holder using AUSF and UDM for primary authentication and authorization 411

5.30.2.10 Onboarding of UEs for SNPNs 412

5.30.2.10.1 General 412

5.30.2.10.2 Onboarding Network is an SNPN 412

5.30.2.10.3 Onboarding Network is a PLMN 417

5.30.2.10.4 Remote Provisioning of UEs in Onboarding Network 417

5.30.2.11 UE Mobility support for SNPN 419

5.30.2.12 Access to SNPN services via Untrusted non-3GPP access 420

5.30.2.13 Access to SNPN services via Trusted non-3GPP access 421

5.30.2.14 Access to SNPN services via wireline access network 422

5.30.2.15 Access to SNPN services for N5CW devices 422

5.30.3 Public Network Integrated NPN 422

5.30.3.1 General 422

5.30.3.2 Identifiers 423

5.30.3.3 UE configuration, subscription aspects and storage 423

5.30.3.4 Network and cell (re-)selection and access control 424

5.30.3.5 Support of emergency services in CAG cells 426

5.31 Support for Cellular IoT 426

5.31.1 General 426

5.31.2 Preferred and Supported Network Behaviour 426

5.31.3 Selection, steering and redirection between EPS and 5GS 427

5.31.4 Control Plane CIoT 5GS Optimisation 428

5.31.4.1 General 428

5.31.4.2 Establishment of N3 data transfer during Data Transport in Control Plane CIoT 5GS Optimisation 429

5.31.4.3 Control Plane Relocation Indication procedure 429

5.31.5 Non-IP Data Delivery (NIDD) 429

5.31.6 Reliable Data Service 430

5.31.7 Power Saving Enhancements 431

5.31.7.1 General 431

5.31.7.2 Extended Discontinuous Reception (DRX) for CM-IDLE and CM-CONNECTED with RRC-INACTIVE 431

5.31.7.2.1 Overview 431

5.31.7.2.2 Paging for extended idle mode DRX in E-UTRA and NR connected to 5GC 433

5.31.7.2.3 Paging for a UE registered in a tracking area with heterogeneous support of extended idle mode DRX 434

5.31.7.2.4 Paging for extended DRX for RRC_INACTIVE in NR connected to 5GC 434

5.31.7.3 MICO mode with Extended Connected Time 435

5.31.7.4 MICO mode with Active Time 435

5.31.7.5 MICO mode and Periodic Registration Timer Control 435

5.31.8 High latency communication 436

5.31.9 Support for Monitoring Events 437

5.31.10 NB-IoT UE Radio Capability Handling 437

5.31.11 Inter-RAT idle mode mobility to and from NB-IoT 437

5.31.12 Restriction of use of Enhanced Coverage 438

5.31.13 Paging for Enhanced Coverage 439

5.31.14 Support of rate control of user data 439

5.31.14.1 General 439

5.31.14.2 Serving PLMN Rate Control 439

5.31.14.3 Small Data Rate Control 440

5.31.15 Control Plane Data Transfer Congestion Control 441

5.31.16 Service Gap Control 441

5.31.17 Inter-UE QoS for NB-IoT 443

5.31.18 User Plane CIoT 5GS Optimisation 443

5.31.19 QoS model for NB-IoT 444

5.31.20 Category M UEs differentiation 444

5.32 Support for ATSSS 445

5.32.1 General 445

5.32.2 Multi Access PDU Sessions 446

5.32.3 Policy for ATSSS Control 451

5.32.4 QoS Support 452

5.32.5 Access Network Performance Measurements 453

5.32.5.1 General principles 453

5.32.5.1a Address of PMF messages 456

5.32.5.2 Round Trip Time Measurements 457

5.32.5.2a Packet Loss Rate Measurements 457

5.32.5.3 Access Availability/Unavailability Report 458

5.32.5.4 Protocol stack for user plane measurements and measurement reports 459

5.32.5.5 UE Assistance Operation 460

5.32.5.6 Suspend and Resume Traffic Duplication 460

5.32.6 Support of Steering Functionalities 461

5.32.6.1 General 461

5.32.6.2 High-Layer Steering Functionalities 463

5.32.6.2.1 MPTCP Functionality 463

5.32.6.2.2 MPQUIC Functionality 464

5.32.6.3 Low-Layer Steering Functionalities 467

5.32.6.3.1 ATSSS-LL Functionality 467

5.32.7 Interworking with EPS 467

5.32.7.1 General 467

5.32.7.2 Interworking with N26 Interface 468

5.32.7.3 Interworking without N26 Interface 469

5.32.8 ATSSS Rules 469

5.33 Support for Ultra Reliable Low Latency Communication 473

5.33.1 General 473

5.33.2 Redundant transmission for high reliability communication 474

5.33.2.1 Dual Connectivity based end to end Redundant User Plane Paths 474

5.33.2.2 Support of redundant transmission on N3/N9 interfaces 476

5.33.2.3 Support for redundant transmission at transport layer 478

5.33.3 QoS Monitoring for packet delay 478

5.33.3.1 General 478

5.33.3.2 Per QoS Flow per UE QoS Measurement 478

5.33.3.3 GTP-U Path Measurement 479

5.34 Support of deployments topologies with specific SMF Service Areas 481

5.34.1 General 481

5.34.2 Architecture 482

5.34.2.1 SBA architecture 482

5.34.2.2 Non-roaming architecture 482

5.34.2.3 Roaming architecture 483

5.34.3 I-SMF selection, V-SMF reselection 483

5.34.4 Usage of an UL Classifier for a PDU Session controlled by I-SMF 484

5.34.5 Usage of IPv6 multi-homing for a PDU Session controlled by I-SMF 485

5.34.6 Interaction between I-SMF and SMF for the support of traffic offload by UPF controlled by the I-SMF 485

5.34.6.1 General 485

5.34.6.2 N4 information sent from SMF to I-SMF for local traffic offload 486

5.34.7 Event Management 487

5.34.7.1 UE's Mobility Event Management 487

5.34.7.2 SMF event exposure service 487

5.34.7.3 AMF implicit subscription about events related with the PDU Session 487

5.34.8 Support for Cellular IoT 488

5.34.9 Support of the Deployment Topologies with specific SMF Service Areas feature within and between PLMN(s) 488

5.34.10 Support for 5G LAN-type service 488

5.35 Support for Integrated access and backhaul (IAB) 488

5.35.1 IAB architecture and functional entities 488

5.35.2 5G System enhancements to support IAB 490

5.35.3 Data handling and QoS support with IAB 490

5.35.4 Mobility support with IAB 491

5.35.5 Charging support with IAB 491

5.35.6 IAB operation involving EPC 491

5.35A Support for Mobile Base Station Relay (MBSR) 491

5.35A.1 General 491

5.35A.2 Configuration of the MBSR 492

5.35A.3 Mobility support of UEs served by MBSR 492

5.35A.3.1 UE mobility between a fixed cell and MBSR cell 492

5.35A.3.2 UE mobility between MBSR cells 492

5.35A.3.3 UE mobility when moving together with a MBSR cell 492

5.35A.3.4 MBSR mobility 492

5.35A.4 MBSR authorization 493

5.35A.5 Location Service Support of UEs served by MBSR 494

5.35A.6 Providing cell ID/TAC of MBSR for services 494

5.35A.7 Control of UE access to MBSR 495

5.36 RIM Information Transfer 495

5.37 Support for high data rate low latency services, eXtended Reality (XR) and interactive media services 495

5.37.1 General 495

5.37.2 Policy control enhancements to support multi-modal services 496

5.37.3 Support of ECN marking for L4S to expose the congestion information 496

5.37.3.1 General 496

5.37.3.2 Support of ECN marking for L4S in NG-RAN 497

5.37.3.3 Support of ECN marking for L4S in PSA UPF 497

5.37.4 Network Exposure of 5GS information 498

5.37.5 PDU Set based Handling 499

5.37.5.1 General 499

5.37.5.2 PDU Set Information and Identification 500

5.37.5.3 Non-homogenous support of PDU set based handling in NG-RAN 500

5.37.6 UL/DL policy control based on round-trip latency requirement 501

5.37.7 5GS Packet Delay Variation monitoring and reporting 501

5.37.7.1 General 501

5.37.8 UE power saving management 502

5.37.8.1 General 502

5.37.8.2 Periodicity and N6 Jitter Information associated with Periodicity 502

5.37.8.3 End of Data Burst Indication 503

5.38 Support for Multi-USIM UE 503

5.38.1 General 503

5.38.2 Connection Release 503

5.38.3 Paging Cause Indication for Voice Service 504

5.38.4 Reject Paging Request 504

5.38.5 Paging Restriction 505

5.38.6 Paging Timing Collision Control 505

5.39 Remote provisioning of credentials for NSSAA or secondary authentication/authorization 506

5.39.1 General 506

5.39.2 Configuration for the UE 506

5.40 Support of Disaster Roaming with Minimization of Service Interruption 506

5.40.1 General 506

5.40.2 UE configuration and provisioning for Disaster Roaming 507

5.40.3 Disaster Condition Notification and Determination 507

5.40.4 Registration for Disaster Roaming service 508

5.40.5 Handling when a Disaster Condition is no longer applicable 508

5.40.6 Prevention of signalling overload related to Disaster Condition and Disaster Roaming service 509

5.41 NR RedCap and NR eRedCap UEs differentiation 509

5.42 Support of Non-seamless WLAN offload 510

5.43 Support for 5G Satellite Backhaul 511

5.43.1 General 511

5.43.2 Edge Computing via UPF deployed on satellite 511

5.43.3 Local switch for UE-to-UE communications via UPF deployed on GEO satellite 512

5.43.3.1 General 512

5.43.3.2 Local switch with PSA UPF deployed on satellite 512

5.43.3.3 Local switching with UL CL/BP and local PSA UPF deployed on satellite 513

5.43.4 Reporting of satellite backhaul to SMF 513

5.43.5 QoS monitoring when dynamic Satellite Backhaul is used 513

5.44 Support of Personal IoT network service 514

5.44.1 General 514

5.44.2 UE policy delivery for PIN 514

5.44.3 Session management enhancement for PIN service support 514

5.44.3.1 PDU Session Establishment for PIN 514

5.44.3.2 Session management related policy control 515

5.44.3.3 Non-3GPP QoS Assistance Information 515

5.44.3.4 Non-3GPP delay budget between PINE and PEGC 515

5.44.4 Identifiers for PIN 516

5.45 QoS Monitoring 516

5.45.1 General 516

5.45.2 Packet delay monitoring 517

5.45.3 Congestion information monitoring 517

5.45.4 Data rate monitoring 517

5.45.5 Void 517

5.46 Assistance to AI/ML Operations in the Application Layer 518

5.46.1 General 518

5.46.2 Member UE selection assistance functionality for application operation 519

5.47 Support for Network Controlled Repeater (NCR) 520

6 Network Functions 520

6.1 General 520

6.2 Network Function Functional description 520

6.2.1 AMF 520

6.2.2 SMF 522

6.2.3 UPF 524

6.2.4 PCF 525

6.2.5 NEF 525

6.2.5.0 NEF functionality 525

6.2.5.1 Support for CAPIF 527

6.2.5a Void 527

6.2.6 NRF 527

6.2.6.1 General 527

6.2.6.2 NF profile 528

6.2.6.3 SCP profile 530

6.2.7 UDM 531

6.2.8 AUSF 531

6.2.9 N3IWF 531

6.2.9A TNGF 532

6.2.10 AF 532

6.2.11 UDR 533

6.2.12 UDSF 533

6.2.13 SMSF 533

6.2.14 NSSF 533

6.2.15 5G-EIR 534

6.2.16 LMF 534

6.2.16A GMLC 534

6.2.17 SEPP 534

6.2.18 Network Data Analytics Function (NWDAF) 534

6.2.19 SCP 535

6.2.20 W-AGF 535

6.2.21 UE radio Capability Management Function (UCMF) 536

6.2.22 TWIF 536

6.2.23 NSSAAF 536

6.2.24 DCCF 536

6.2.25 MFAF 537

6.2.26 ADRF 537

6.2.27 MB-SMF 537

6.2.27a MB-UPF 537

6.2.27b MBSF 537

6.2.27c MBSTF 537

6.2.28 NSACF 538

6.2.29 TSCTSF 538

6.2.30 5G DDNMF 539

6.2.31 EASDF 539

6.2.32 TSN AF 539

6.2.33 NSWOF 539

6.3 Principles for Network Function and Network Function Service discovery and selection 539

6.3.1 General 539

6.3.1.0 Principles for Binding, Selection and Reselection 541

6.3.1.1 NF Discovery and Selection aspects relevant with indirect communication 543

6.3.1.2 Location information 543

6.3.2 SMF discovery and selection 543

6.3.3 User Plane Function Selection 546

6.3.3.1 Overview 546

6.3.3.2 SMF Provisioning of available UPF(s) 547

6.3.3.3 Selection of an UPF for a particular PDU Session 547

6.3.4 AUSF discovery and selection 548

6.3.5 AMF discovery and selection 549

6.3.6 N3IWF selection 552

6.3.6.1 General 552

6.3.6.2 Stand-alone N3IWF selection 554

6.3.6.2a SNPN N3IWF selection 555

6.3.6.3 Combined N3IWF/ePDG Selection 556

6.3.6.4 PLMN and non-3GPP access node Selection for emergency services 558

6.3.6.4.1 General 558

6.3.6.4.2 Stand-alone N3IWF selection 558

6.3.6.3.3 Combined N3IWF/ePDG Selection 559

6.3.7 PCF discovery and selection 560

6.3.7.0 General principles 560

6.3.7.1 PCF discovery and selection for a UE or a PDU Session 560

6.3.7.2 Providing policy requirements that apply to multiple UE and hence to multiple PCF 563

6.3.7.3 Binding an AF request targeting a UE address to the relevant PCF 563

6.3.7.4 Binding an AF request targeting a UE to the relevant PCF 563

6.3.8 UDM discovery and selection 563

6.3.9 UDR discovery and selection 564

6.3.10 SMSF discovery and selection 565

6.3.11 CHF discovery and selection 565

6.3.12 Trusted Non-3GPP Access Network selection 567

6.3.12.1 General 567

6.3.12.2 Access Network Selection Procedure 568

6.3.12a Access Network selection for devices that do not support 5GC NAS over WLAN 571

6.3.12a.1 General 571

6.3.12a.2 Access Network Selection Procedure 571

6.3.12b Access Network selection for 5G NSWO 572

6.3.13 NWDAF discovery and selection 573

6.3.14 NEF Discovery 574

6.3.15 UCMF Discovery and Selection 575

6.3.16 SCP discovery and selection 575

6.3.17 NSSAAF discovery and selection 576

6.3.18 5G-EIR discovery and selection 576

6.3.19 DCCF discovery and selection 576

6.3.20 ADRF discovery and selection 577

6.3.21 MFAF discovery and selection 577

6.3.22 NSACF discovery and selection 577

6.3.23 EASDF discovery and selection 578

6.3.24 TSCTSF Discovery 578

6.3.25 AF Discovery and Selection 579

6.3.26 NRF discovery and selection 579

7 Network Function Services and descriptions 579

7.1 Network Function Service Framework 579

7.1.1 General 579

7.1.2 NF Service Consumer - NF Service Producer interactions 580

7.1.3 Network Function Service discovery 582

7.1.4 Network Function Service Authorization 582

7.1.5 Network Function and Network Function Service registration and de-registration 583

7.2 Network Function Services 583

7.2.1 General 583

7.2.2 AMF Services 584

7.2.3 SMF Services 585

7.2.4 PCF Services 585

7.2.5 UDM Services 586

7.2.6 NRF Services 587

7.2.7 AUSF Services 588

7.2.8 NEF Services 588

7.2.8A Void 591

7.2.9 SMSF Services 591

7.2.10 UDR Services 591

7.2.11 5G-EIR Services 592

7.2.12 NWDAF Services 592

7.2.13 UDSF Services 593

7.2.14 NSSF Services 593

7.2.15 BSF Services 594

7.2.16 LMF Services 594

7.2.16A GMLC Services 594

7.2.17 CHF Services 594

7.2.18 UCMF Services 595

7.2.19 AF Services 595

7.2.20 NSSAAF Services 596

7.2.21 DCCF Services 596

7.2.22 MFAF Services 596

7.2.23 ADRF Services 597

7.2.24 5G DDNMF Services 597

7.2.25 EASDF Services 597

7.2.26 TSCTSF Services 597

7.2.27 NSACF Services 598

7.2.28 MB-SMF Services 598

7.2.29 UPF Services 599

7.3 Exposure 599

8 Control and User Plane Protocol Stacks 599

8.1 General 599

8.2 Control Plane Protocol Stacks 599

8.2.1 Control Plane Protocol Stacks between the 5G-AN and the 5G Core: N2 599

8.2.1.1 General 599

8.2.1.2 5G-AN - AMF 600

8.2.1.3 5G-AN - SMF 601

8.2.2 Control Plane Protocol Stacks between the UE and the 5GC 601

8.2.2.1 General 601

8.2.2.2 UE - AMF 603

8.2.2.3 UE – SMF 603

8.2.3 Control Plane Protocol Stacks between the network functions in 5GC 604

8.2.3.1 The Control Plane Protocol Stack for the service based interface 604

8.2.3.2 The Control Plane protocol stack for the N4 interface between SMF and UPF 604

8.2.4 Control Plane for untrusted non 3GPP Access 604

8.2.5 Control Plane for trusted non-3GPP Access 605

8.2.6 Control Plane for W-5GAN Access 606

8.2.7 Control Plane for Trusted WLAN Access for N5CW Device 606

8.3 User Plane Protocol Stacks 606

8.3.1 User Plane Protocol Stack for a PDU Session 606

8.3.2 User Plane for untrusted non-3GPP Access 608

8.3.3 User Plane for trusted non-3GPP Access 608

8.3.4 User Plane for W-5GAN Access 608

8.3.5 User Plane for N19-based forwarding of a 5G VN group 609

8.3.6 User Plane for Trusted WLAN Access for N5CW Device 609

Annex A (informative): Relationship between Service-Based Interfaces and Reference Points 610

Annex B (normative): Mapping between temporary identities 612

Annex C (informative): Guidelines and Principles for Compute-Storage Separation 613

Annex D (informative): 5GS support for Non-Public Network deployment options 614

D.1 Introduction 614

D.2 Support of Non-Public Network as a network slice of a PLMN 614

D.3 Support for access to PLMN services via Stand-alone Non-Public Network and access to Stand-alone Non Public Network services via PLMN 615

D.4 Support for UE capable of simultaneously connecting to an SNPN and a PLMN 616

D.5 Support for keeping UE in CM-CONNECTED state in overlay network when accessing services via NWu 616

D.6 Support for session/service continuity between SNPN and PLMN when using N3IWF 617

D.7 Guidance for underlay network to support QoS differentiation for User Plane IPsec Child SA 618

D.7.1 Network initiated QoS 618

D.7.2 UE initiated QoS 619

Annex E (informative): Communication models for NF/NF services interaction 621

E.1 General 621

Annex F (informative): Redundant user plane paths based on multiple UEs per device 623

Annex G (informative): SCP Deployment Examples 626

G.1 General 626

G.2 An SCP based on service mesh 626

G.2.1 Introduction 626

G.2.2 Communication across service mesh boundaries 627

G.3 An SCP based on independent deployment units 628

G.4 An SCP deployment example based on name-based routing 629

G.4.0 General Information 629

G.4.1 Service Registration and Service Discovery 630

G.4.2 Overview of Deployment Scenario 631

G.4.3 References 631

Annex H (normative): PTP usage guidelines 632

H.1 General 632

H.2 Signalling of ingress time for time synchronization 632

H.3 Void 632

H.4 Path and Link delay measurements 632

Annex I (normative): TSN usage guidelines 634

I.1 Determination of traffic pattern information 634

Annex J (informative): Link MTU considerations 636

Annex K (normative): Port and user plane node management information exchange 638

K.1 Standardized port and user plane node management information 638

K.2 Port and user plane node management information exchange for time synchronization 654

K.2.1 Capability exchange 654

K.2.2 PTP Instance configuration 655

K.2.2.1 General 655

K.2.2.2 Configuration for Sync and Announce reception timeouts 656

K.2.2.3 Configuration for PTP port states 656

K.2.2.4 Configuration for PTP grandmaster function 657

K.2.2.5 Configuration for Sync and Announce intervals 657

K.2.2.6 Configuration for transport protocols 658

Annex L (normative): Support of GERAN/UTRAN access 659

Annex M (normative): Interworking with TSN deployed in the Transport Network 660

M.1 Mapping of the parameters between 5GS and TSN UNI 660

M.2 TL-Container Information 664

Annex N (informative): Support for access to Localized Services 667

N.1 General 667

N.2 Enabling access to Localized Services 667

N.2.1 General 667

N.2.2 Configuration of network to provide access to Localized Services 668

N.2.3 Session Management aspects 668

N.3 Selection of network providing access to Localized Services 668

N.4 Enabling the UE access to Localized Services 668

N.5 Support for leaving network that provides access to Localized Services 668

N.6 Configuration of Credentials Holder for determining SNPN selection information 669

Annex O (informative): Allowing UE to simultaneously send data to different groups with different QoS policy 671

O.1 A PDU Session with multiple QoS Flows for different groups 671

O.2 Multiple PDU Sessions for different groups 672

O.3 A PDU Session targeting a predefined group formed of multiple sub-groups 673

Annex P (informative): Personal IoT Networks 675

P.1 PIN Reference Architecture 675

P.2 Session management and traffic routing for PIN 675

Annex Q (informative): Satellite coverage availability information 677

Annex R (informative): Change history 678
