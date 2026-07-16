---
spec: TS 23.502
version: 18.14.0
release: '18'
clause: contents
title: Contents
source_archive: 23502-ie0.zip
source_document: 23502-ie0.docx
source_archive_sha256: 0f24045c49ddf710fb91e51ec74d070cfd13d1e9bafc07809d7e87eae6a61ce8
source_document_sha256: 89536df6095fba3b9c90f1fa06ceaec918dfc6b0fd26a769a06eae7db66bb603
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents

Foreword 26

1 Scope 27

2 References 27

3 Definitions, symbols and abbreviations 30

3.1 Definitions 30

3.2 Abbreviations 31

4 System procedures 31

4.1 General 31

4.2 Connection, Registration and Mobility Management procedures 31

4.2.1 General 31

4.2.2 Registration Management procedures 31

4.2.2.1 General 31

4.2.2.2 Registration procedures 31

4.2.2.2.1 General 31

4.2.2.2.2 General Registration 33

4.2.2.2.3 Registration with AMF re-allocation 54

4.2.2.2.4 Registration with Onboarding SNPN 58

4.2.2.3 Deregistration procedures 60

4.2.2.3.1 General 60

4.2.2.3.2 UE-initiated Deregistration 60

4.2.2.3.3 Network-initiated Deregistration 62

4.2.3 Service Request procedures 63

4.2.3.1 General 63

4.2.3.2 UE Triggered Service Request 64

4.2.3.3 Network Triggered Service Request 76

4.2.4 UE Configuration Update 85

4.2.4.1 General 85

4.2.4.2 UE Configuration Update procedure for access and mobility management related parameters 86

4.2.4.3 UE Configuration Update procedure for transparent UE Policy delivery 90

4.2.5 Reachability procedures 92

4.2.5.1 General 92

4.2.5.2 UE Reachability Notification Request procedure 92

4.2.5.3 UE Activity Notification procedure 94

4.2.6 AN Release 95

4.2.7 N2 procedures 98

4.2.7.1 N2 Configuration 98

4.2.7.2 NGAP UE-TNLA-binding related procedures 98

4.2.7.2.1 Creating NGAP UE-TNLA-bindings during Registration and Service Request 98

4.2.7.2.2 Creating NGAP UE-TNLA-bindings during handovers 99

4.2.7.2.3 Re-Creating NGAP UE-TNLA-bindings subsequent to NGAP UE-TNLA-binding release 99

4.2.7.2.4 NGAP UE-TNLA-binding update procedure 100

4.2.7.2.5 NGAP UE-TNLA-binding per UE Release procedure 100

4.2.7.3 AMF Failure or Planned Maintenance handling procedure 100

4.2.7.4 Additional User Location Information with Mobile Base Station Relay (MBSR) 101

4.2.8 Void 101

4.2.8a UE Capability Match Request procedure 101

4.2.9 Network Slice-Specific Authentication and Authorization procedure 102

4.2.9.1 General 102

4.2.9.2 Network Slice-Specific Authentication and Authorization 103

4.2.9.3 AAA Server triggered Network Slice-Specific Re-authentication and Re-authorization procedure 105

4.2.9.4 AAA Server triggered Slice-Specific Authorization Revocation 106

4.2.10 N3 data transfer establishment procedure when Control Plane CIoT 5GS Optimisation is enabled 107

4.2.10.1 UE triggered N3 data transfer establishment procedure 107

4.2.10.2 SMF triggered N3 data transfer establishment procedure 107

4.2.11 Network Slice Admission Control Function (NSACF) procedures 108

4.2.11.1 General 108

4.2.11.2 Number of UEs per network slice availability check and update procedure 108

4.2.11.2a Hierarchical NSACF-based number of UEs per network slice availability check and update procedure 111

4.2.11.3 Configuration for Early Admission Control (EAC) update procedure 113

4.2.11.4 Number of PDU Sessions per network slice availability check and update procedure 114

4.2.11.4a Hierarchical NSACF-based Number of PDU Sessions per network slice availability check and update procedure 117

4.2.11.5 Network Slice Admission Control Support for Roaming 118

4.2.11.5.1 Network Slice Admission Control Support for Roaming by VPLMN 118

4.2.11.5.2 Network Slice Admission Control Support for Roaming by HPLMN 119

4.2.11.6 Update of local maximum number in Hierarchical NSAC Architecture 124

4.3 Session Management procedures 125

4.3.1 General 125

4.3.2 PDU Session Establishment 126

4.3.2.1 General 126

4.3.2.2 UE Requested PDU Session Establishment 126

4.3.2.2.1 Non-roaming and Roaming with Local Breakout 126

4.3.2.2.2 Home-routed Roaming 142

4.3.2.2.3 SMF selection 147

4.3.2.2.4 Multiple PDU Sessions towards the same DNN and S-NSSAI 151

4.3.2.3 Secondary authorization/authentication by an DN-AAA Server during the PDU Session establishment 151

4.3.2.4 Support of L2TP 154

4.3.3 PDU Session Modification 156

4.3.3.1 General 156

4.3.3.2 UE or network requested PDU Session Modification (non-roaming and roaming with local breakout) 156

4.3.3.3 UE or network requested PDU Session Modification (home-routed roaming) 166

4.3.4 PDU Session Release 170

4.3.4.1 General 170

4.3.4.2 UE or network requested PDU Session Release for Non-Roaming and Roaming with Local Breakout 170

4.3.4.3 UE or network requested PDU Session Release for Home-routed Roaming 176

4.3.5 Session continuity, service continuity and UP path management 179

4.3.5.1 Change of SSC mode 2 PDU Session Anchor with different PDU Sessions 179

4.3.5.2 Change of SSC mode 3 PDU Session Anchor with multiple PDU Sessions 180

4.3.5.3 Change of SSC mode 3 PDU Session Anchor with IPv6 Multi-homed PDU Session 183

4.3.5.4 Addition of additional PDU Session Anchor and Branching Point or UL CL 186

4.3.5.5 Removal of additional PDU Session Anchor and Branching Point or UL CL 187

4.3.5.6 Change of additional PDU Session Anchor for IPv6 multi-homing or UL CL 189

4.3.5.7 Simultaneous change of Branching Point or UL CL and additional PSA for a PDU Session 190

4.3.5.8 Ethernet PDU Session Anchor Relocation 193

4.3.6 Application Function influence on traffic routing and service function chaining 195

4.3.6.1 General 195

4.3.6.2 Processing AF requests to influence traffic routing and/or Service Function Chaining for Sessions not identified by an UE address 198

4.3.6.3 Notification of User Plane Management Events 200

4.3.6.4 Transferring an AF request targeting an individual UE address to the relevant PCF 204

4.3.6.5 Processing AF requests to influence traffic routing for HR-SBO session 205

4.3.6.5.1 General 205

4.3.6.5.2 AF traffic influence request includes HPLMN DNN, HPLMN S-NSSAI 206

4.3.6.5.3 AF traffic influence request without HPLMN DNN, S-NSSAI information for a single UE, with private IP address or public IP address owned by VPLMN 208

4.3.6.5.4 AF traffic influence request without HPLMN DNN, S-NSSAI information for a single UE, with UE IP address owned and assigned by HPLMN 209

4.3.6.5.5 AF traffic influence request for GPSI or any UE 210

4.3.7 CN-initiated selective deactivation of UP connection of an existing PDU Session 211

4.3.8 Change of Network Slice instance for PDU Sessions 212

4.4 SMF and UPF interactions 213

4.4.1 N4 session management procedures 213

4.4.1.1 General 213

4.4.1.2 N4 Session Establishment procedure 213

4.4.1.3 N4 Session Modification procedure 214

4.4.1.4 N4 Session Release procedure 214

4.4.2 N4 Reporting Procedures 215

4.4.2.1 General 215

4.4.2.2 N4 Session Level Reporting Procedure 215

4.4.3 N4 Node Level Procedures 216

4.4.3.1 N4 Association Setup Procedure 216

4.4.3.2 N4 Association Update Procedure 217

4.4.3.3 N4 Association Release Procedure 218

4.4.3.4 N4 Report Procedure 218

4.4.3.5 N4 PFD management Procedure 219

4.4.4 SMF Pause of Charging procedure 220

4.5 User Profile management procedures 221

4.5.1 Subscriber Data Update Notification to AMF 221

4.5.2 Session Management Subscriber Data Update Notification to SMF 222

4.5.3 Purge of subscriber data in AMF 222

4.6 Security procedures 223

4.7 ME Identity check procedure 223

4.8 RAN-CN interactions 223

4.8.1 Connection Inactive and Suspend procedure 223

4.8.1.1 Connection Inactive procedure 223

4.8.1.1a Connection Inactive procedure with CN based MT communication handling 223

4.8.1.2 Connection Suspend procedure 225

4.8.2 Connection Resume procedure 226

4.8.2.1 General 226

4.8.2.2 UE Triggered Connection Resume in RRC_INACTIVE procedure 226

4.8.2.2a Network Triggered Connection Resume in RRC_INACTIVE procedure 228

4.8.2.2b Network Triggered Connection Resume in RRC_INACTIVE with CN based MT communication handling 229

4.8.2.3 Connection Resume in CM-IDLE with Suspend procedure 231

4.8.2.4 Connection Resume in CM-IDLE with Suspend and MO EDT procedure 232

4.8.3 N2 Notification procedure 234

4.9 Handover procedures 235

4.9.1 Handover procedures in 3GPP access 235

4.9.1.1 General 235

4.9.1.2 Xn based inter NG-RAN handover 235

4.9.1.2.1 General 235

4.9.1.2.2 Xn based inter NG-RAN handover without User Plane function re-allocation 236

4.9.1.2.3 Xn based inter NG-RAN handover with insertion of intermediate UPF 240

4.9.1.2.4 Xn based inter NG-RAN handover with re-allocation of intermediate UPF 241

4.9.1.3 Inter NG-RAN node N2 based handover 243

4.9.1.3.1 General 243

4.9.1.3.2 Preparation phase 244

4.9.1.3.3 Execution phase 251

4.9.1.3.3a Execution phase for DAPS handover 255

4.9.1.4 Inter NG-RAN node N2 based handover, Cancel 256

4.9.2 Handover of a PDU Session procedure between 3GPP and untrusted non-3GPP access 256

4.9.2.0 General 256

4.9.2.1 Handover of a PDU Session procedure from untrusted non-3GPP to 3GPP access (non-roaming and roaming with local breakout) 256

4.9.2.2 Handover of a PDU Session procedure from 3GPP to untrusted non-3GPP access (non-roaming and roaming with local breakout) 257

4.9.2.3 Handover of a PDU Session procedure from untrusted non-3GPP to 3GPP access (home routed roaming) 258

4.9.2.3.1 The target AMF is in the PLMN of the N3IWF 258

4.9.2.3.2 The target AMF is not in the PLMN of the N3IWF (i.e. N3IWF in HPLMN) 259

4.9.2.4 Handover of a PDU Session procedure from 3GPP to untrusted non-3GPP access (home routed roaming) 260

4.9.2.4.1 The selected N3IWF is in the registered PLMN 260

4.9.2.4.2 The UE is roaming and the selected N3IWF is in the home PLMN 260

4.9.3 Handover of a PDU Session procedure between 3GPP and trusted non-3GPP access 261

4.9.3.0 General 261

4.10 NG-RAN Location reporting procedures 261

4.11 System interworking procedures with EPC 263

4.11.0 General 263

4.11.0a Impacts to EPS Procedures 263

4.11.0a.1 General 263

4.11.0a.2 Interaction with PCC 264

4.11.0a.2a Interaction with PCC for URSP delivery via EPS 264

4.11.0a.2a.0 General 264

4.11.0a.2a.1 SM Policy Association Establishment 265

4.11.0a.2a.1a SM Policy Association Establishment in EPS 265

4.11.0a.2a.2 SMF initiated SM Policy Association Modification 266

4.11.0a.2a.3 PCF initiated SM Policy Association Modification 267

4.11.0a.2a.4 Void 267

4.11.0a.2a.5 UE Policy Association establishment 267

4.11.0a.2a.6 UE Policy Association Modification initiated by the PCF for the UE 269

4.11.0a.2a.7 UE Policy Association Modification initiated by the PCF for the PDU Session 270

4.11.0a.2a.8 UE Policy Association Termination initiated by PCF for PDU session 271

4.11.0a.2a.9 UE Policy Association termination initiated by the PCF for the UE 271

4.11.0a.2a.10 UE Policy Container delivery via EPS 272

4.11.0a.3 Mobility Restrictions 273

4.11.0a.4 PGW Selection 273

4.11.0a.5 PDN Connection Establishment 274

4.11.0a.6 Network Configuration 276

4.11.0a.7 Interactions with DN-AAA Server 276

4.11.0a.8 5GC NAS capability (re-)enabled and disabled 276

4.11.0a.9 PDN Connection Release 276

4.11.0a.10 UE requested bearer resource modification procedure 276

4.11.0a.11 SMF+PGW-C initiated bearer modification without bearer QoS update 277

4.11.1 N26 based Interworking Procedures 277

4.11.1.1 General 277

4.11.1.2 Handover procedures 279

4.11.1.2.1 5GS to EPS handover using N26 interface 279

4.11.1.2.2 EPS to 5GS handover using N26 interface 285

4.11.1.2.3 Handover Cancel 293

4.11.1.3 Idle Mode Mobility procedures 295

4.11.1.3.1 General 295

4.11.1.3.2 5GS to EPS Idle mode mobility using N26 interface 295

4.11.1.3.2A 5GS to EPS Idle mode mobility using N26 interface with data forwarding 298

4.11.1.3.3 EPS to 5GS Mobility Registration Procedure (Idle and Connected State) using N26 interface 299

4.11.1.3.3A EPS to 5GS Idle mode mobility using N26 interface with data forwarding 305

4.11.1.3.4 EPS to 5GS Mobility Registration Procedure (Idle) using N26 interface with AMF reallocation 306

4.11.1.4 Procedures for EPS bearer ID allocation 307

4.11.1.4.1 EPS bearer ID allocation 307

4.11.1.4.2 EPS bearer ID transfer 310

4.11.1.4.3 EPS bearer ID revocation 311

4.11.1.5 Impacts to EPS Procedures 312

4.11.1.5.1 General 312

4.11.1.5.2 E-UTRAN Initial Attach 312

4.11.1.5.3 Tracking Area Update 313

4.11.1.5.4 Session Management 314

4.11.1.5.5 5GS to EPS handover using N26 interface 316

4.11.1.5.6 UE triggered Service Request 316

4.11.1.5.7 Establishment of S1-U bearer during Data Transport in Control Plane CIoT EPS Optimisation 316

4.11.1.5.8 Radio Resource Management functions and Information Storage 316

4.11.1.6 EPS interworking information storing Procedure 317

4.11.2 Interworking procedures without N26 interface 317

4.11.2.1 General 317

4.11.2.2 5GS to EPS Mobility 317

4.11.2.3 EPS to 5GS Mobility 320

4.11.2.4 Impacts to EPS Procedures 322

4.11.2.4.1 E-UTRAN Attach 322

4.11.2.4.2 Session Management 324

4.11.2.4.3 Void 324

4.11.3 Handover procedures between EPS and 5GC-N3IWF 325

4.11.3.1 Handover from EPS to 5GC-N3IWF 325

4.11.3.2 Handover from 5GC-N3IWF to EPS 325

4.11.3a Handover procedures between EPS and 5GC-TNGF 326

4.11.3a.1 Handover from EPS to 5GC-TNGF 326

4.11.3a.2 Handover from 5GC-TNGF to EPS 326

4.11.4 Handover procedures between EPC/ePDG and 5GS 327

4.11.4.1 Handover from EPC/ePDG to 5GS 327

4.11.4.2 Handover from 5GS to EPC/ePDG 328

4.11.4.3 Impacts to EPC/ePDG Procedures 328

4.11.4.3.1 General 328

4.11.4.3.2 ePDG FQDN construction 328

4.11.4.3.3 Initial Attach with GTP on S2b 329

4.11.4.3.3a Initial Attach for emergency session (GTP on S2b) 329

4.11.4.3.4 Interaction with PCC 329

4.11.4.3.5 UE initiated Connectivity to Additional PDN with GTP on S2b 330

4.11.4.3.6 Use of N10 interface instead of S6b 330

4.11.4.3.7 5GC NAS capability (re-)enabled and disabled 331

4.11.5 Impacts to 5GC Procedures 331

4.11.5.1 General 331

4.11.5.2 Registration procedure 331

4.11.5.3 UE Requested PDU Session Establishment procedure 332

4.11.5.4 UE or Network Requested PDU Session Modification procedure 334

4.11.5.5 Xn based inter NG-RAN handover 334

4.11.5.6 Inter NG-RAN node N2 based handover 335

4.11.5.7 UE or Network Requested PDU Session Release procedure 335

4.11.5.8 Network Configuration 335

4.11.5.9 Network Slice Admission Control 335

4.11.5.9a Network Slice Admission Control in 5GS for maximum number of UE with at least one PDU session and one PDN connection 336

4.11.5.10 UE Policy Association control 337

4.11.5.10.1 UE Policy Association Establishment for URSP delivery in EPS 337

4.11.6 Interworking for common network exposure 338

4.11.6.1 Subscription and Notification of availability or expected level of support of a service API 338

4.11.6.2 Unsubscribing to N11otification of availability or expected level of support of a service API 339

4.11.6.3 Configuration of monitoring events for common network exposure 340

4.12 Procedures for Untrusted non-3GPP access 341

4.12.1 General 341

4.12.2 Registration via Untrusted non-3GPP Access 341

4.12.2.1 General 341

4.12.2.2 Registration procedure for untrusted non-3GPP access 341

4.12.2.3 Emergency Registration for untrusted non-3GPP Access 345

4.12.3 Deregistration procedure for untrusted non-3gpp access 346

4.12.4 N2 procedures via Untrusted non-3GPP Access 347

4.12.4.1 Service Request procedures via Untrusted non-3GPP Access 347

4.12.4.2 Procedure for the UE context release in the N3IWF 348

4.12.4.3 CN-initiated selective deactivation of UP connection of an existing PDU Session associated with Untrusted non-3GPP Access 349

4.12.5 UE Requested PDU Session Establishment via Untrusted non-3GPP Access 349

4.12.6 UE or Network Requested PDU Session Modification via Untrusted non-3GPP access 351

4.12.7 UE or network Requested PDU Session Release via Untrusted non-3GPP access 353

4.12.8 Mobility from a non-geographically selected AMF to a geographically selected AMF 355

4.12a Procedures for Trusted non-3GPP access 356

4.12a.1 General 356

4.12a.2 Registration via Trusted non-3GPP Access 356

4.12a.2.1 General 356

4.12a.2.2 Registration procedure for trusted non-3GPP access 356

4.12a.2.3 Emergency Registration for trusted non-3GPP Access 360

4.12a.3 Deregistration procedure for Trusted non-3GPP access 360

4.12a.4 N2 procedures via Trusted non-3GPP Access 361

4.12a.4.1 Service Request procedures via Trusted non-3GPP Access 361

4.12a.4.2 Procedure for the UE context release in the TNGF 361

4.12a.4.3 CN-initiated selective deactivation of UP connection of an existing PDU Session associated with Trusted non-3GPP Access 361

4.12a.5 UE Requested PDU Session Establishment via Trusted non-3GPP Access 361

4.12a.6 UE or network Requested PDU Session Modification via Trusted non-3GPP access 362

4.12a.7 UE or network Requested PDU Session Release via Trusted non-3GPP access 363

4.12a.8 Mobility from a non-geographically selected AMF to a geographically selected AMF 363

4.12a.9 Support of mobility from source to target TNAP 363

4.12b Procedures for devices that do not support 5GC NAS over WLAN access 363

4.12b.1 General 363

4.12b.2 Initial Registration & PDU Session Establishment 363

4.12b.3 Deregistration procedure 367

4.12b.4 N2 procedures 367

4.12b.4.1 Service Request procedures 367

4.12b.4.2 Procedure for the UE context release in the TWIF 367

4.12b.4.3 CN-initiated selective deactivation of UP connection of an existing PDU Session 368

4.13 Specific services 368

4.13.1 General 368

4.13.2 Application Triggering 368

4.13.2.1 General 368

4.13.2.2 The procedure of "Application Triggering" Service 369

4.13.3 SMS over NAS procedures 371

4.13.3.1 Registration procedures for SMS over NAS 371

4.13.3.2 Deregistration procedures for SMS over NAS 372

4.13.3.3 MO SMS over NAS in CM-IDLE (baseline) 373

4.13.3.4 Void 374

4.13.3.5 MO SMS over NAS in CM-CONNECTED 374

4.13.3.6 MT SMS over NAS in CM-IDLE state and RRC_INACTIVE with CN based MT communication state via 3GPP access 375

4.13.3.7 MT SMS over NAS in CM-CONNECTED state via 3GPP access 376

4.13.3.8 MT SMS over NAS via non-3GPP access 376

4.13.3.9 Unsuccessful Mobile terminating SMS delivery re-attempt 376

4.13.4 Emergency Services 377

4.13.4.1 General 377

4.13.4.2 Emergency Services Fallback 377

4.13.5 Location Services procedures 379

4.13.5.0 General 379

4.13.5.1 5GC-NI-LR Procedure 379

4.13.5.2 5GC-MT-LR Procedure without UDM Query 379

4.13.5.3 5GC-MT-LR Procedure 379

4.13.5.4 UE Assisted and UE Based Positioning Procedure 379

4.13.5.5 Network Assisted Positioning Procedure 379

4.13.5.6 Obtaining Non-UE Associated Network Assistance Data 379

4.13.5.7 Location continuity for Handover of an Emergency session from NG-RAN 379

4.13.6 Support of IMS Voice 379

4.13.6.1 EPS fallback for IMS voice 379

4.13.6.2 Inter RAT Fallback in 5GC for IMS voice 381

4.13.6.3 Transfer of PDU session used for IMS voice from non-3GPP access to 5GS 382

4.13.7 MSISDN-less MO SMS 383

4.13.7.1 General 383

4.13.7.2 The procedure of MSISDN-less MO SMS Service 384

4.13.8 Support of 5G LAN-type service 385

4.13.8.1 Support of 5G VN group management 385

4.13.8.2 Support of 5G VN group communication 385

4.13.8.2.1 General 385

4.13.8.2.2 group-level N4 session management procedures 385

4.14 Support for Dual Connectivity 385

4.14.1 RAN Initiated QoS Flow Mobility 385

4.15 Network Exposure 387

4.15.1 General 387

4.15.2 External Exposure of Network Capabilities 391

4.15.2a Data Collection from an AF 391

4.15.3 Event Exposure using NEF 392

4.15.3.1 Monitoring Events 392

4.15.3.2 Information flows 398

4.15.3.2.1 AMF service operations information flow 398

4.15.3.2.2 UDM service operations information flow 398

4.15.3.2.3 NEF service operations information flow 401

4.15.3.2.3a Void 405

4.15.3.2.3b Specific NEF service operations information flow for loss of connectivity and UE reachability 405

4.15.3.2.3c Specific NEF service operations information flow for group member list change 407

4.15.3.2.4 Exposure with bulk subscription 407

4.15.3.2.5 Information flow for downlink data delivery status with SMF buffering 409

4.15.3.2.6 GMLC service operations information flow 410

4.15.3.2.7 Information flow for Availability after DDN Failure with SMF buffering 410

4.15.3.2.8 Information flow for downlink data delivery status with UPF buffering 412

4.15.3.2.9 Information flow for Availability after DDN Failure with UPF buffering 413

4.15.3.2.10 Number of UEs and PDU Sessions per network slice notification procedure 415

4.15.3.2.11 Network-initiated explicit event notification subscription cancel procedure 419

4.15.3.2.12 Void 420

4.15.3.2.13 Handling AF requests when the UE is identified via UE addressing information 420

4.15.3.3 Void 420

4.15.4 Core Network Internal Event Exposure 420

4.15.4.1 General 420

4.15.4.2 Exposure of Mobility Events from AMF 421

4.15.4.3 Exposure of Communication trends from SMF 422

4.15.4.4 Internal Event Exposure Subscription/Unsubscription via UDM 423

4.15.4.5 Exposure of Events from UPF for UPF Data Collection 424

4.15.4.5.1 General 424

4.15.4.5.2 Information flow for subscription to UPF event exposure service for certain UE(s) via SMF 426

4.15.4.5.3 Information flow for UPF event exposure service for any UE 427

4.15.4.5.4 Information flow for subscription via SMF to UPF event exposure service related with AOI 428

4.15.5 Void 429

4.15.6 External Parameter Provisioning 429

4.15.6.1 General 429

4.15.6.2 NEF service operations information flow 430

4.15.6.3 Expected UE Behaviour parameters 433

4.15.6.3a Network Configuration parameters 434

4.15.6.3b 5G VN group data 435

4.15.6.3c 5G VN Group membership management parameters 436

4.15.6.3d ECS Address Configuration Information Parameters 436

4.15.6.3e DNN and S-NSSAI specific Group Parameters 437

4.15.6.3f Application-Specific Expected UE Behaviour parameters 437

4.15.6.3g Network Slice Usage Control parameters for dedicated S-NSSAI to a single AF 438

4.15.6.4 Set a chargeable party at AF session setup 438

4.15.6.5 Change the chargeable party during the session 439

4.15.6.6 Setting up an AF session with required QoS procedure 440

4.15.6.6a AF session with required QoS update procedure 444

4.15.6.7 Service specific parameter provisioning 446

4.15.6.7.1 General 446

4.15.6.7.2 Service specific parameter provisioning by AF to HPLMN 446

4.15.6.7.3 Service specific parameter provisioning by AF to VPLMN 448

4.15.6.7a Authorization of service specific parameter provisioning 450

4.15.6.8 Set a policy for a future AF session 452

4.15.6.9 Procedures for AF-triggered dynamically changing access and mobility management policies 452

4.15.6.9.1 General 452

4.15.6.9.2 Processing AF requests to influence access and mobility management policies targeting an individual UE 453

4.15.6.9.3 Processing AF requests to influence access and mobility management policies 455

4.15.6.10 Application guidance for URSP determination 456

4.15.6.11 Void 458

4.15.6.12 Parameter Provisioning when the UE is identified via UE addressing information 458

4.15.6.13 Multi-member AF session with required QoS 458

4.15.6.13.1 General Descriptions 458

4.15.6.13.2 Procedures for Creating a Multi-member AF session with required QoS 460

4.15.6.13.3 Procedure for updating a Multi-member AF session with required QoS 462

4.15.6.13.4 Procedure for Revoking a Multi-member AF session with required QoS 464

4.15.6.14 Procedures for AF requested QoS for a UE or group of UEs not identified by a UE address 465

4.15.7 Network status reporting 466

4.15.8 Event exposure controlled by a DCCF 467

4.15.9 Time Synchronization exposure 467

4.15.9.1 General 467

4.15.9.2 Exposure of UE availability for Time Synchronization service 467

4.15.9.3 Time Synchronization activation, modification and deactivation 470

4.15.9.3.1 General 470

4.15.9.3.2 Time synchronization service activation 475

4.15.9.3.3 Time synchronization service modification 479

4.15.9.3.4 Time synchronization service deactivation 481

4.15.9.4 Procedures for management of 5G access stratum time distribution 481

4.15.9.5 Procedures for subscription and management of network timing synchronization status monitoring 488

4.15.9.5.1 Network timing synchronization status 488

4.15.9.5.2 5G access stratum time distribution status reporting to the UE 490

4.15.10 AF specific UE ID retrieval 491

4.15.10A MSISDN retrieval 492

4.15.11 Void 493

4.15.12 Event Exposure using Local NEF 493

4.15.13 Assistance for Member UE selection 494

4.15.13.0 General 494

4.15.13.1 Member UE selection assistance subscribe and update procedure 494

4.15.13.2 Member UE Filtering Criteria for 5GS assistance to Member UE selection 496

4.15.13.3 Specific procedure for QoS Member UE filtering criteria 499

4.15.13.3.1 General 499

4.15.13.3.2 Void 499

4.15.13.3.3 Member UE Selection Assistance with QoS filtering criteria for real-time QoS Monitoring 499

4.15.13.4 Specific procedure for the 5GC assistance to member UE selection based on the UE's current location, historical location and direction and UE separation distance 502

4.15.13.5 Specific procedure for Service Experience Member UE filtering criteria 503

4.15.13.5.1 General 503

4.15.13.5.2 Member UE Selection Assistance with Service Experience filtering criteria 504

4.15.13.6 Specific procedure for end-to-end data volume transfer time related member UE filtering criteria 505

4.15.13.6.1 General 505

4.15.13.6.2 Member UE Selection assistance with end-to-end data volume transfer time related filtering criteria 506

4.15.13.7 Member UE selection assistance unsubscribe procedure 507

4.16 Procedures and flows for Policy Framework 507

4.16.1 AM Policy Association Establishment 507

4.16.1.1 General 507

4.16.1.2 AM Policy Association Establishment with new Selected PCF 508

4.16.1.3 Void 509

4.16.2 AM Policy Association Modification 509

4.16.2.0 General 509

4.16.2.1 AM Policy Association Modification initiated by the AMF 509

4.16.2.1.1 AM Policy Association Modification initiated by the AMF without AMF relocation 509

4.16.2.1.2 AM Policy Association Modification with old PCF during AMF relocation 510

4.16.2.2 AM Policy Association Modification initiated by the PCF 512

4.16.3 AM Policy Association Termination 513

4.16.3.1 General 513

4.16.3.2 AMF-initiated AM Policy Association Termination 513

4.16.3.3 Void 514

4.16.4 SM Policy Association Establishment 514

4.16.5 SM Policy Association Modification 516

4.16.5.0 General 516

4.16.5.1 SMF initiated SM Policy Association Modification 516

4.16.5.2 PCF initiated SM Policy Association Modification 518

4.16.6 SM Policy Association Termination 520

4.16.7 Negotiations for future background data transfer 521

4.16.7.1 General 521

4.16.7.2 Procedures for future background data transfer 522

4.16.7.3 Procedure for BDT warning notification 524

4.16.8 Procedures on interaction between PCF and CHF 525

4.16.8.1 General 525

4.16.8.2 Initial Spending Limit Retrieval 526

4.16.8.3 Intermediate Spending Limit Report Retrieval 526

4.16.8.4 Final Spending Limit Report Retrieval 527

4.16.8.5 Spending Limit Report 528

4.16.8.6 CHF report the removal of the subscriber 528

4.16.9 Update of the subscription information in the PCF 529

4.16.10 Void 530

4.16.11 UE Policy Association Establishment 530

4.16.11.1 General 530

4.16.12 UE Policy Association Modification 532

4.16.12.1 UE Policy Association Modification initiated by the AMF 532

4.16.12.1.1 UE Policy Association Modification initiated by the AMF without AMF relocation 532

4.16.12.1.2 UE Policy Association Modification with old PCF during AMF relocation 534

4.16.12.2 UE Policy Association Modification initiated by the PCF 535

4.16.13 UE Policy Association Termination 536

4.16.13.1 AMF-initiated UE Policy Association Termination 536

4.16.13.2 PCF-initiated UE Policy Association Termination 538

4.16.14 Management of access and mobility related policy information depending on the application in use 539

4.16.14.1 General 539

4.16.14.2 Procedures for management of access and mobility related policy information 539

4.16.14.2.1 Management of access and mobility related policy information at start and stop of application traffic 539

4.16.14.2.2 Management of access and mobility related policy information at SM Policy Association establishment and termination with the notification sent by the BSF 542

4.16.15 Negotiations for planned data transfer with QoS requirements 543

4.16.15.1 General 543

4.16.15.2 Procedures 543

4.16.15.2.1 Procedures for negotiation of planned data transfer with QoS requirements 543

4.16.15.2.2 Procedure for PDTQ warning notification 546

4.16.16 Awareness of URSP Rule Enforcement 547

4.16.16.1 General 547

4.16.16.2 Forwarding of URSP Rule Enforcement Information (for non-roaming and HR roaming) 547

4.16.16.3 Forwarding of URSP Rule Enforcement Information (for LBO roaming) 549

4.17 Network Function Service Framework Procedure 551

4.17.1 NF service Registration 551

4.17.2 NF service update 552

4.17.3 NF service deregistration 552

4.17.4 NF/NF service discovery by NF service consumer in the same PLMN 553

4.17.4a NF/NF service discovery by NF service consumer in the same SNPN 554

4.17.5 NF/NF service discovery across PLMNs in the case of discovery made by NF service consumer 554

4.17.5a NF/NF service discovery between SNPN and Credentials Holder hosting AUSF/UDM or between SNPN and DCS hosting AUSF/UDM 555

4.17.6 SMF Provisioning of available UPFs using the NRF 556

4.17.6.1 General 556

4.17.6.2 SMF provisioning of UPF instances using NRF 556

4.17.7 NF/NF service status subscribe/notify in the same PLMN 558

4.17.8 NF/NF service status subscribe/notify across PLMNs 558

4.17.9 Delegated service discovery when NF service consumer and NF service producer are in same PLMN 559

4.17.10 Delegated service discovery when NF service consumer and NF service producer are in different PLMNs 561

4.17.11 Indirect Communication without delegated discovery Procedure 561

4.17.12 Binding between NF service consumer and NF service producer 562

4.17.12.1 General 562

4.17.12.2 Binding created as part of service response 562

4.17.12.3 Binding created as part of service request 563

4.17.12.4 Binding for subscription requests 564

4.17.13 NRF bootstrapping procedure 566

4.18 Procedures for Management of PFDs 566

4.18.1 General 566

4.18.2 PFD management via NEF (PFDF) 566

4.18.2.1 PFD management triggered by AF 566

4.18.2.2 PFD management based on PFD Determination analytics 567

4.18.3 PFD management in the SMF 568

4.18.3.1 PFD Retrieval by the SMF 568

4.18.3.2 Management of PFDs in the SMF 568

4.19 Network Data Analytics 569

4.19.1 Void 569

4.19.2 Void 569

4.20 UE Parameters Update via UDM Control Plane Procedure 569

4.20.1 General 569

4.20.2 UE Parameters Update via UDM Control Plane Procedure 570

4.20.3 Void 571

4.21 Secondary RAT Usage Data Reporting Procedure 571

4.22 ATSSS Procedures 572

4.22.1 General 572

4.22.2 UE Requested MA PDU Session Establishment 573

4.22.2.0 Overview 573

4.22.2.1 Non-roaming and Roaming with Local Breakout 573

4.22.2.2 Home-routed Roaming 575

4.22.2.2.1 Home-routed Roaming - UE registered to the same PLMN 575

4.22.2.2.2 Home-routed Roaming - UE registered to different PLMNs 576

4.22.2.3 MA PDU Session establishment with 3GPP access connected to EPC and non-3GPP access connected to 5GC 577

4.22.2.3.1 General 577

4.22.2.3.2 PDN Connections and Multi Access PDU Sessions 578

4.22.2.3.3 QoS Support 579

4.22.2.4 MA PDU Session establishment with non-3GPP access connected to EPC and 3GPP access connected to 5GC 579

4.22.2.4.1 General 579

4.22.2.4.2 PDN Connections and Multi Access PDU Sessions 580

4.22.2.4.3 QoS Support 581

4.22.3 UE Requested PDU Session Establishment with Network Modification to MA PDU Session 581

4.22.3.1 Overview 581

4.22.3.2 Non-roaming or roaming with local breakout 582

4.22.3.3 Home-routed, the UE registered to the same VPLMN over both access 582

4.22.3.4 Home-routed, the UE registered to different PLMNs over both access 583

4.22.4 Access Network Performance Measurements 584

4.22.5 Reporting of Access Availability 584

4.22.5A Suspend and Resume Traffic Duplication 584

4.22.6 EPS Interworking 585

4.22.6.1 General 585

4.22.6.2 Impacts to EPS interworking procedures 585

4.22.6.2.1 5GS to EPS handover using N26 interface 585

4.22.6.2.2 5GS to EPS idle mode mobility using N26 interface 585

4.22.6.2.3 EPS bearer ID allocation 586

4.22.6.2.4 EPS bearer ID revocation 586

4.22.6.2.5 5GS to EPS mobility without N26 interface 586

4.22.6.3 Network Modification to MA PDU Session after a UE moving from EPC 587

4.22.7 Adding / Re-activating / De-activating User-Plane Resources 588

4.22.8 UE or network requested MA PDU Session Modification 588

4.22.8.1 General 588

4.22.8.2 UE or network requested MA PDU Session Modification (non-roaming and roaming with local breakout) 588

4.22.8.3 UE or network requested MA PDU Session Modification (home-routed roaming) 589

4.22.9 Connection, Registration and Mobility Management procedures 590

4.22.9.1 Registration procedures 590

4.22.9.2 UE Triggered Service Request 591

4.22.9.3 N2 based handover 591

4.22.9.4 Network Triggered Service Request 591

4.22.9.5 Registration procedures for non-3GPP access path switching 592

4.22.10 MA PDU Session Release 594

4.22.10.1 General 594

4.22.10.2 UE or network requested MA PDU Session Release (non-roaming and roaming with local breakout) 595

4.22.10.3 UE or network requested MA PDU Session Release (home-routed roaming) 595

4.23 Support of deployments topologies with specific SMF Service Areas 596

4.23.1 General 596

4.23.2 I-SMF selection 597

4.23.3 Registration procedure 597

4.23.3a Deregistration procedure 598

4.23.4 Service Request procedures 599

4.23.4.1 General 599

4.23.4.2 UE Triggered Service Request without I-SMF change/removal 599

4.23.4.3 UE Triggered Service Request with I-SMF insertion/change/removal 599

4.23.4.4 Network Triggered Service Request 606

4.23.5 PDU Session Management procedure 606

4.23.5.1 PDU Session establishment procedure 606

4.23.5.2 PDU Session Release procedure 607

4.23.5.3 PDU Session modification procedure 607

4.23.5.4 SMF triggered I-SMF selection or removal 608

4.23.6 I-SMF Related Procedures with PCF 609

4.23.6.1 General 609

4.23.6.2 Policy Update Procedures with I-SMF 610

4.23.6.3 Reporting UP path change to the AF 610

4.23.7 Inter NG-RAN node N2 based handover 611

4.23.7.1 General 611

4.23.7.2 Inter NG-RAN node N2 based handover without I-SMF change/removal 611

4.23.7.2.1 General 611

4.23.7.2.2 Preparation phase 611

4.23.7.2.3 Execution phase 611

4.23.7.2.4 Handover Cancel 611

4.23.7.3 Inter NG-RAN node N2 based handover with I-SMF insertion/change/removal 612

4.23.7.3.1 General 612

4.23.7.3.2 Preparation phase 613

4.23.7.3.3 Execution phase 619

4.23.7.3.4 Handover Cancel 623

4.23.8 AN Release procedure involving I-SMF 625

4.23.9 Branching Point or UL CL controlled by I-SMF 625

4.23.9.0 Overview 625

4.23.9.1 Addition of PDU Session Anchor and Branching Point or UL CL controlled by I-SMF 626

4.23.9.2 Removal of PDU Session Anchor and Branching Point or UL CL controlled by I-SMF 629

4.23.9.3 Change of PDU Session Anchor for IPv6 multi-homing or UL CL controlled by I-SMF 630

4.23.9.4 Simultaneous change of Branching Point or UL CL and additional PSA controlled by I-SMF 632

4.23.9.5 Simultaneous change of Branching Points or UL CLs controlled by different I-SMFs 634

4.23.9a Void 637

4.23.10 CN-initiated selective deactivation of UP connection of an existing PDU Session involving I-SMF 637

4.23.11 Xn based handover 637

4.23.11.1 General 637

4.23.11.2 Xn based handover with insertion of intermediate SMF 637

4.23.11.3 Xn based handover with re-allocation of intermediate SMF 639

4.23.11.4 Xn based handover with removal of intermediate SMF 641

4.23.11.5 Xn based handover without change of intermediate SMF 643

4.23.12 N26 based Interworking Procedures with I-SMF 643

4.23.12.1 General 643

4.23.12.2 5GS to EPS Idle mode mobility using N26 interface with I-SMF removal 643

4.23.12.2A 5GS to EPS Idle mode mobility using N26 interface with Data forwarding and I-SMF removal 643

4.23.12.3 EPS to 5GS mobility registration procedure (Idle and Connected State) using N26 interface with I-SMF insertion 643

4.23.12.3A EPS to 5GS Idle mode mobility using N26 interface with Data forwarding and I-SMF insertion 644

4.23.12.4 Procedures for EPS bearer ID allocation 644

4.23.12.5 EPS to 5GS mobility registration procedure (Idle) using N26 interface with AMF reallocation and I-SMF insertion 644

4.23.12.6 5GS to EPS handover using N26 interface with I-SMF removal 644

4.23.12.7 EPS to 5GS handover using N26 interface with I-SMF insertion 644

4.23.12.7.1 Preparation phase 644

4.23.12.7.2 Execution phase 645

4.23.12.8 Impact to 5GC procedures 645

4.23.12.8.1 General 645

4.23.12.8.2 UE Triggered Service Request with I-SMF insertion/change/removal 645

4.23.12.8.3 PDU Session establishment procedure 645

4.23.12.8.4 PDU Session modification procedure 645

4.23.12.8.5 Inter NG-RAN node N2 based handover with I-SMF insertion/change/removal 645

4.23.12.8.6 Xn based handover with insertion of I-SMF 646

4.23.13 Non N26 based Interworking Procedures with I-SMF 646

4.23.13.1 General 646

4.23.13.2 Mobility procedure without N26 interface from EPS to 5GS 646

4.23.13.3 Mobility procedure without N26 interface from EPC/ePDG to 5GS 646

4.23.13.4 Mobility procedure without N26 interface from 5GS to EPS 646

4.23.13.5 Mobility procedure without N26 interface from 5GS to EPC/ePDG 646

4.23.14 Pause of charging 647

4.23.15 PDU Session mobility between 3GPP and non-3GPP access 647

4.23.16 Handover of a PDU Session procedure between 3GPP and untrusted non-3GPP access 647

4.23.16.1 Handover of a PDU Session procedure from untrusted non-3GPP to 3GPP access (non-roaming and roaming with local breakout) 647

4.23.16.2 Handover of a PDU Session procedure from 3GPP to untrusted non-3GPP access (non-roaming and roaming with local breakout) 647

4.23.16.3 Handover of a PDU Session procedure from untrusted non-3GPP to 3GPP access (home routed roaming) 648

4.23.16.3.1 The target AMF is in the PLMN of the N3IWF 648

4.23.16a Handover of a PDU Session procedure between 3GPP and trusted non-3GPP access 648

4.23.16a.1 General 648

4.23.17 Additional considerations for Home-routed roaming 648

4.24 Procedures for UPF Anchored Data Transport in Control Plane CIoT 5GS Optimisation 649

4.24.1 UPF anchored Mobile Originated Data Transport in Control Plane CIoT 5GS Optimisation 649

4.24.2 UPF anchored Mobile Terminated Data Transport in Control Plane CIoT 5GS Optimisation 652

4.25 Procedures for NEF based Non-IP Data Delivery 657

4.25.1 General 657

4.25.2 SMF-NEF Connection Establishment 657

4.25.3 NIDD Configuration 658

4.25.4 NEF Anchored Mobile Originated Data Transport 660

4.25.5 NEF Anchored Mobile Terminated Data Transport 661

4.25.6 NIDD Authorization Update 664

4.25.7 SMF Initiated SMF-NEF Connection Release procedure 665

4.25.8 NEF Initiated SMF-NEF Connection Release procedure 665

4.25.9 NEF Anchored Group NIDD via NEF anchored unicast MT data 667

4.26 Network Function/NF Service Context Transfer Procedures 667

4.26.1 General 667

4.26.2 NF/NF Service Context Transfer Push Procedure 668

4.26.3 NF/NF Service Context Transfer Pull procedure 668

4.26.4 Context Transfer due to decommissioning 669

4.26.5 SMF Service Context Transfer procedures 669

4.26.5.1 General 669

4.26.5.2 I-SMF Context Transfer procedure 669

4.26.5.3 SMF Context Transfer procedure, LBO or no Roaming, no I-SMF 669

4.27 Procedures for Enhanced Coverage Restriction Control via NEF 672

4.27.1 General 672

4.28 Subscription-based distribution of timing information 673

4.28.1 General 673

4.28.2 5G Access Stratum-based Time Distribution 673

4.28.2.1 Control of access stratum time synchronization service without AF request 673

4.28.3 (g)PTP-based Time Distribution 674

4.28.3.1 Control of (g)PTP time synchronization service without AF request 674

5 Network Function Service procedures 676

5.1 Network Function Service framework procedures 676

5.1.1 Network Function Service Discovery 676

5.2 Network Function services 676

5.2.1 General 676

5.2.2 AMF Services 676

5.2.2.1 General 676

5.2.2.2 Namf_Communication service 677

5.2.2.2.1 General 677

5.2.2.2.2 Namf_Communication_UEContextTransfer service operation 678

5.2.2.2.3 Namf_Communication_RegistrationStatusUpdate service operation 683

5.2.2.2.4 Namf_Communication_N1MessageNotify service operation 684

5.2.2.2.5 Namf_Communication_N1N2MessageSubscribe service operation 684

5.2.2.2.6 Namf_Communication_N1N2MessageUnSubscribe service operation 684

5.2.2.2.7 Namf_Communication_N1N2MessageTransfer service operation 685

5.2.2.2.7A Namf_Communication_N1N2TransferFailureNotification service operation 686

5.2.2.2.8 Namf_Communication_N2InfoSubscribe service operation 686

5.2.2.2.9 Namf_Communication_N2InfoUnsubscribe service operation 686

5.2.2.2.10 Namf_Communication_N2InfoNotify service operation 686

5.2.2.2.11 Namf_Communication_CreateUEContext service operation 687

5.2.2.2.12 Namf_Communication_ReleaseUEContext service operation 687

5.2.2.2.13 Namf_Communication_EBIAssignment service operation 687

5.2.2.2.14 Namf_Communication_AMFStatusChangeSubscribe service operation 688

5.2.2.2.15 Namf_Communication_AMFStatusChangeUnSubscribe service operation 688

5.2.2.2.16 Namf_Communication_AMFStatusChangeNotify service operation 688

5.2.2.2.17 Namf_Communication_NonUeN2MessageTransfer service operation 688

5.2.2.2.18 Namf_Communication_NonUeN2InfoSubscribe service operation 689

5.2.2.2.19 Namf_Communication_NonUeN2InfoUnSubscribe service operation 689

5.2.2.2.20 Namf_Communication_NonUeN2InfoNotify service operation 689

5.2.2.2.21 Namf_Communication_RelocateUEContext service operation 689

5.2.2.2.22 Namf_Communication_CancelRelocateUEContext service operation 690

5.2.2.3 Namf_EventExposure service 690

5.2.2.3.1 General 690

5.2.2.3.2 Namf_EventExposure_Subscribe service operation 692

5.2.2.3.3 Namf_EventExposure_UnSubscribe service operation 693

5.2.2.3.4 Namf_EventExposure_Notify service operation 693

5.2.2.4 Namf_MT service 694

5.2.2.4.1 General 694

5.2.2.4.2 Namf_MT_EnableUEReachability service operation 694

5.2.2.4.3 Namf_MT_ProvideDomainSelectionInfo 695

5.2.2.4.4 Namf_MT_EnableGroupReachability service operation 695

5.2.2.4.5 Namf_MT_UEReachabilityInfoNotify 695

5.2.2.5 Namf_Location service 695

5.2.2.5.1 General 695

5.2.2.5.2 Namf_Location_ProvidePositioningInfo service operation 696

5.2.2.5.3 Namf_Location_EventNotify service operation 696

5.2.2.5.4 Namf_Location_ProvideLocationInfo service operation 696

5.2.2.5.5 Namf_Location_CancelLocation service operation 697

5.2.3 UDM Services 697

5.2.3.1 General 697

5.2.3.2 Nudm_UECM (UECM) service 698

5.2.3.2.1 Nudm_UECM_Registration service operation 698

5.2.3.2.2 Nudm_UECM_DeregistrationNotification service operation 699

5.2.3.2.3 Nudm_UECM_Deregistration service operation 700

5.2.3.2.4 Nudm_UECM_Get service operation 700

5.2.3.2.5 Nudm_UECM_Update service operation 700

5.2.3.2.6 Nudm_UECM_PCscfRestoration service operation 701

5.2.3.2.7 Nudm_UECM_SendRoutingInfoForSM service operation 701

5.2.3.3 Nudm_SubscriberDataManagement (SDM) Service 701

5.2.3.3.1 General 701

5.2.3.3.2 Nudm_SDM_Get service operation 714

5.2.3.3.3 Nudm_SDM_Notification service operation 714

5.2.3.3.4 Nudm_SDM_Subscribe service operation 715

5.2.3.3.5 Nudm_SDM_Unsubscribe service operation 715

5.2.3.3.6 Nudm_SDM_Info service operation 715

5.2.3.3.7 Void 716

5.2.3.3.8 Nudm_SDM_ModifySubscription service operation 716

5.2.3.4 Nudm_UEAuthentication Service 716

5.2.3.4.1 General 716

5.2.3.4.2 Nudm_UEAuthentication_Get service operation 716

5.2.3.4.3 Nudm_UEAuthentication_ResultConfirmation service operation 716

5.2.3.5 Nudm_EventExposure service 716

5.2.3.5.1 General 716

5.2.3.5.2 Nudm_EventExposure_Subscribe service operation 716

5.2.3.5.3 Nudm_EventExposure_Unsubscribe service operation 717

5.2.3.5.4 Nudm_EventExposure_Notify service operation 717

5.2.3.5.5 Nudm_EventExposure_ModifySubscription service operation 717

5.2.3.6 Nudm_ParameterProvision service 717

5.2.3.6.1 General 717

5.2.3.6.2 Nudm_ParameterProvision_Update service operation 719

5.2.3.6.3 Nudm_ParameterProvision_Create service operation 719

5.2.3.6.4 Nudm_ParameterProvision_Delete service operation 720

5.2.3.6.5 Nudm_ParameterProvision_Get service operation 720

5.2.3.7 Nudm_NIDDAuthorisation service 720

5.2.3.7.1 General 720

5.2.3.7.2 Nudm_NIDDAuthorisation_Get service operation 720

5.2.3.7.3 Nudm_NIDDAuthorisation_UpdateNotify service operation 721

5.2.3.7.4 Void 721

5.2.3.7.5 Void 721

5.2.3.8 Nudm_ServiceSpecificAuthorisation service 721

5.2.3.8.1 General 721

5.2.3.8.2 Nudm_ServiceSpecificAuthorisation_Create service operation 721

5.2.3.8.3 Nudm_ServiceSpecificAuthorisation_UpdateNotify service operation 721

5.2.3.8.4 Nudm_ServiceSpecificAuthorisation_Remove service operation 722

5.2.3.9 Nudm_ReportSMDeliveryStatus service 722

5.2.3.9.1 General 722

5.2.3.9.2 Nudm_ReportSMDeliveryStatus_Request service operation 722

5.2.4 5G-EIR Services 722

5.2.4.1 General 722

5.2.4.2 N5g-eir_EquipmentIdentityCheck service 722

5.2.4.2.1 General 722

5.2.4.2.2 N5g-eir_EquipmentIdentityCheck_Get service operation 722

5.2.5 PCF Services 723

5.2.5.1 General 723

5.2.5.2 Npcf_AMPolicyControl service 724

5.2.5.2.1 General 724

5.2.5.2.2 Npcf_AMPolicyControl_Create service operation 725

5.2.5.2.3 Npcf_AMPolicyControl_UpdateNotify service operation 725

5.2.5.2.4 Npcf_AMPolicyControl_Delete service operation 726

5.2.5.2.5 Npcf_AMPolicyControl_Update service operation 726

5.2.5.3 Npcf_PolicyAuthorization Service 726

5.2.5.3.1 General 726

5.2.5.3.2 Npcf_PolicyAuthorization_Create service operation 726

5.2.5.3.3 Npcf_PolicyAuthorization_Update service operation 727

5.2.5.3.4 Npcf_PolicyAuthorization_Delete service operation 728

5.2.5.3.5 Npcf_PolicyAuthorization_Notify service operation 728

5.2.5.3.6 Npcf_PolicyAuthorization_Subscribe service operation 729

5.2.5.3.7 Npcf_PolicyAuthorization_Unsubscribe service operation 729

5.2.5.4 Npcf_SMPolicyControl service 729

5.2.5.4.1 General 729

5.2.5.4.2 Npcf_SMPolicyControl_Create service operation 730

5.2.5.4.3 Npcf_SMPolicyControl_UpdateNotify service operation 730

5.2.5.4.4 Npcf_SMPolicyControl_Delete service operation 731

5.2.5.4.5 Npcf_SMPolicyControl_Update service operation 731

5.2.5.5 Npcf_BDTPolicyControl Service 731

5.2.5.5.1 General 731

5.2.5.5.2 Npcf_BDTPolicyControl_Create service operation 731

5.2.5.5.3 Npcf_BDTPolicyControl_Update service operation 732

5.2.5.5.4 Npcf_BDTPolicyControl_Notify service operation 732

5.2.5.6 Npcf_UEPolicyControl Service 732

5.2.5.6.1 General 732

5.2.5.6.2 Npcf_UEPolicyControl_Create service operation 733

5.2.5.6.3 Npcf_UEPolicyControl_UpdateNotify service operation 733

5.2.5.6.4 Npcf_UEPolicyControl_Delete service operation 734

5.2.5.6.5 Npcf_UEPolicyControl_Update service operation 734

5.2.5.7 Npcf_EventExposure service 734

5.2.5.7.1 General 734

5.2.5.7.2 Npcf_EventExposure_Subscribe service operation 734

5.2.5.7.3 Npcf_EventExposure_Unsubscribe service operation 735

5.2.5.7.4 Npcf_EventExposure_Notify service operation 735

5.2.5.8 Npcf_AMPolicyAuthorization Service 736

5.2.5.8.1 General 736

5.2.5.8.2 Npcf_AMPolicyAuthorization_Create service operation 736

5.2.5.8.3 Npcf_AMPolicyAuthorization_Update service operation 736

5.2.5.8.4 Npcf_AMPolicyAuthorization_Delete service operation 736

5.2.5.8.5 Npcf_AMPolicyAuthorization_Notify service operation 737

5.2.5.8.6 Npcf_AMPolicyAuthorization_Subscribe service operation 737

5.2.5.8.7 Npcf_AMPolicyAuthorization_Unsubscribe service operation 737

5.2.5.9 Npcf_PDTQPolicyControl Service 737

5.2.5.9.1 General 737

5.2.5.9.2 Npcf_PDTQPolicyControl_Create service operation 737

5.2.5.9.3 Npcf_PDTQPolicyControl_Update service operation 738

5.2.5.9.4 Npcf_PDTQPolicyControl_Notify service operation 738

5.2.6 NEF Services 738

5.2.6.1 General 738

5.2.6.2 Nnef_EventExposure service 742

5.2.6.2.1 General 742

5.2.6.2.2 Nnef_EventExposure_Subscribe operation 742

5.2.6.2.3 Nnef_EventExposure_Unsubscribe service operation 743

5.2.6.2.4 Nnef_EventExposure_Notify service operation 743

5.2.6.3 Nnef_PFDManagement service 743

5.2.6.3.1 General 743

5.2.6.3.2 Nnef_PFDManagement_Fetch service operation 743

5.2.6.3.3 Nnef_PFDManagement_Subscribe service operation 743

5.2.6.3.4 Nnef_PFDManagement_Notify service operation 744

5.2.6.3.5 Nnef_PFDManagement_Unsubscribe service operation 744

5.2.6.3.6 Nnef_PFDManagement_Create service operation 744

5.2.6.3.7 Nnef_PFDManagement_Update service operation 744

5.2.6.3.8 Nnef_PFDManagement_Delete service operation 744

5.2.6.4 Nnef_ParameterProvision service 745

5.2.6.4.1 General 745

5.2.6.4.2 Nnef_ParameterProvision_Update service operation 745

5.2.6.4.3 Nnef_ParameterProvision_Create service operation 745

5.2.6.4.4 Nnef_ParameterProvision_Delete service operation 745

5.2.6.4.5 Nnef_ParameterProvision_Get service operation 746

5.2.6.5 Nnef_Trigger service 746

5.2.6.5.1 General 746

5.2.6.5.2 Nnef_Trigger_Delivery service operation 746

5.2.6.5.3 Nnef_Trigger_DeliveryNotify service operation 746

5.2.6.6 Nnef_BDTPNegotiation service 746

5.2.6.6.1 General 746

5.2.6.6.2 Nnef_BDTPNegotiation_Create service operation 747

5.2.6.6.3 Nnef_BDTPNegotiation_Update service operation 747

5.2.6.6.4 Nnef_BDTPNegotiation_Notify service operation 747

5.2.6.7 Nnef_TrafficInfluence service 747

5.2.6.7.1 General 747

5.2.6.7.2 Nnef_TrafficInfluence_Create operation 747

5.2.6.7.3 Nnef_TrafficInfluence_Update operation 748

5.2.6.7.4 Nnef_TrafficInfluence_Delete operation 748

5.2.6.7.4A Nnef_TrafficInfluence_Get operation 749

5.2.6.7.5 Nnef_TrafficInfluence_Notify operation 749

5.2.6.7.6 Nnef_TrafficInfluence_AppRelocationInfo operation 749

5.2.6.8 Nnef_ChargeableParty service 749

5.2.6.8.1 General 749

5.2.6.8.2 Nnef_ChargeableParty_Create service operation 750

5.2.6.8.3 Nnef_ChargeableParty_Update service operation 750

5.2.6.8.4 Nnef_ChargeableParty_Notify service operation 750

5.2.6.9 Nnef_AFsessionWithQoS service 750

5.2.6.9.1 General 750

5.2.6.9.2 Nnef_AFsessionWithQoS_Create service operation 750

5.2.6.9.3 Nnef_AFsessionWithQoS_Notify service operation 751

5.2.6.9.4 Nnef_AFsessionWithQoS_Revoke service operation 752

5.2.6.9.5 Nnef_AFsessionWithQoS_Update service operation 752

5.2.6.10 Nnef_MSISDN-less_MO_SMS service 753

5.2.6.10.1 General 753

5.2.6.10.2 Nnef_MSISDN-less_MO_SMSNotify service operation 753

5.2.6.11 Nnef_ServiceParameter service 753

5.2.6.11.1 General 753

5.2.6.11.2 Nnef_ServiceParameter_Create operation 753

5.2.6.11.3 Nnef_ServiceParameter_Update operation 753

5.2.6.11.4 Nnef_ServiceParameter_Delete operation 754

5.2.6.11.5 Nnef_ServiceParameter_Get operation 754

5.2.6.11.6 Nnef_ServiceParameter_Notify operation 754

5.2.6.12 Nnef_APISupportCapability service 755

5.2.6.12.1 General 755

5.2.6.12.2 Nnef_APISupportCapability_Subscribe service operation 755

5.2.6.12.3 Nnef_APISupportCapability_Notify service operation 755

5.2.6.12.4 Nnef_APISupportCapability_Unsubscribe service operation 755

5.2.6.13 Nnef_NIDDConfiguration service 755

5.2.6.13.1 General 755

5.2.6.13.2 Nnef_NIDDConfiguration_Create service operation 756

5.2.6.13.3 Nnef_NIDDConfiguration_TriggerNotify service operation 756

5.2.6.13.4 Nnef_NIDDConfiguration_UpdateNotify service operation 756

5.2.6.13.5 Nnef_NIDDConfiguration_Delete service operation 756

5.2.6.14 Nnef_NIDD service 756

5.2.6.14.1 General 756

5.2.6.14.2 Nnef_NIDD_Delivery service operation 756

5.2.6.14.3 Nnef_NIDD_DeliveryNotify service operation 757

5.2.6.14.4 Nnef_NIDD_GroupDeliveryNotify service operation 757

5.2.6.15 Nnef_SMContext service 757

5.2.6.15.1 General 757

5.2.6.15.2 Nnef_SMContext_Create service operation 757

5.2.6.15.3 Nnef_SMContext_Delete service operation 757

5.2.6.15.4 Nnef_SMContext_DeleteNotify service operation 758

5.2.6.15.5 Nnef_SMContext_Delivery service operation 758

5.2.6.16 Nnef_AnalyticsExposure service 758

5.2.6.16.1 General 758

5.2.6.16.2 Nnef_AnalyticsExposure_Subscribe operation 758

5.2.6.16.3 Nnef_AnalyticsExposure_Unsubscribe service operation 758

5.2.6.16.4 Nnef_AnalyticsExposure_Notify service operation 759

5.2.6.16.5 Nnef_AnalyticsExposure_Fetch service operation 759

5.2.6.17 Nnef_UCMFProvisioning service 759

5.2.6.17.1 General 759

5.2.6.17.2 Nnef_UCMFProvisioning_Create operation 759

5.2.6.17.3 Nnef_UCMFProvisioning_Delete operation 760

5.2.6.17.4 Nnef_UCMFProvisioning_Update operation 760

5.2.6.18 Nnef_ECRestriction service 760

5.2.6.18.1 General 760

5.2.6.18.2 Nnef_ECRestriction_Get service operation 760

5.2.6.18.3 Nnef_ECRestriction_Update service operation 761

5.2.6.19 Nnef_ApplyPolicy service 761

5.2.6.19.1 General 761

5.2.6.19.2 Nnef_ApplyPolicy_Create service operation 761

5.2.6.19.3 Nnef_ApplyPolicy_Update service operation 761

5.2.6.19.4 Nnef_ApplyPolicy_Delete service operation 761

5.2.6.20 Void 762

5.2.6.21 Nnef_Location service 762

5.2.6.21.1 General 762

5.2.6.21.2 Nnef_Location_LocationUpdateNotify service operation 762

5.2.6.22 Nnef_AMPolicyAuthorization Service 762

5.2.6.22.1 General 762

5.2.6.22.2 Nnef_AMPolicyAuthorization_Create service operation 762

5.2.6.22.3 Nnef_AMPolicyAuthorization_Update service operation 762

5.2.6.22.4 Nnef_AMPolicyAuthorization_Delete service operation 763

5.2.6.22.5 Nnef_AMPolicyAuthorization_Notify service operation 763

5.2.6.22.6 Nnef_AMPolicyAuthorization_Subscribe service operation 763

5.2.6.22.7 Nnef_AMPolicyAuthorization_Unsubscribe service operation 763

5.2.6.23 Nnef_AMInfluence service 764

5.2.6.23.1 General 764

5.2.6.23.2 Nnef_AMInfluence_Create operation 764

5.2.6.23.3 Nnef_AMInfluence_Update operation 764

5.2.6.23.4 Nnef_AMInfluence_Delete operation 764

5.2.6.23.6 Nnef_AMInfluence_Notify operation 765

5.2.6.24 Void 765

5.2.6.25 Nnef_TimeSynchronization service 765

5.2.6.25.1 General 765

5.2.6.25.2 Nnef_TimeSynchronization_ConfigCreate operation 765

5.2.6.25.3 Nnef_TimeSynchronization_ConfigUpdate operation 766

5.2.6.25.4 Nnef_TimeSynchronization_ConfigDelete operation 766

5.2.6.25.5 Nnef_TimeSynchronization_ConfigUpdateNotify operation 766

5.2.6.25.6 Nnef_TimeSynchronization_CapsSubscribe operation 766

5.2.6.25.7 Nnef_TimeSynchronization_CapsUnsubscribe operation 767

5.2.6.25.8 Nnef_TimeSynchronization_CapsNotify operation 767

5.2.6.25.9 Void 767

5.2.6.25.10 Void 767

5.2.6.25.11 Void 767

5.2.6.25.12 Void 767

5.2.6.26 Nnef_EASDeployment service 768

5.2.6.26.1 General 768

5.2.6.26.2 Nnef_EASDeployment_Create service operation 768

5.2.6.26.3 Nnef_EASDeployment_Update service operation 768

5.2.6.26.4 Nnef_EASDeployment_Delete service operation 768

5.2.6.26.5 Void 768

5.2.6.26.6 Nnef_EASDeployment_Subscribe service operation 768

5.2.6.26.7 Nnef_EASDeployment_Unsubscribe service operation 769

5.2.6.26.8 Nnef_EASDeployment_Notify service operation 769

5.2.6.27 Nnef_UEId service 769

5.2.6.27.1 General 769

5.2.6.27.2 Nnef_UEId_Get operation 769

5.2.6.27.3 Nnef_UEId_UeIdMappingGet operation 770

5.2.6.27.4 Nnef_UEId_UeIdMappingCreate operation 770

5.2.6.27.5 Nnef_UEId_UeIdMappingUpdate operation 770

5.2.6.27.6 Nnef_UEId_UeIdMappingDelete operation 770

5.2.6.28 Nnef_ASTI service 771

5.2.6.28.1 General 771

5.2.6.28.2 Nnef_ASTI_Create operation 771

5.2.6.28.3 Nnef_ASTI_Update operation 771

5.2.6.28.4 Nnef_ASTI_Delete operation 771

5.2.6.28.5 Nnef_ASTI_Get operation 771

5.2.6.28.6 Nnef_ASTI_UpdateNotify operation 772

5.2.6.28.7 Void 772

5.2.6.28.8 Void 772

5.2.6.29 Nnef_SMService service 772

5.2.6.29.1 General 772

5.2.6.29.2 Nnef_SMService_MoForwardSm service operation 772

5.2.6.30 Nnef_PDTQPolicyNegotiation service 772

5.2.6.30.1 General 772

5.2.6.30.2 Nnef_PDTQPolicyNegotiation_Create service operation 773

5.2.6.30.3 Nnef_PDTQPolicyNegotiation_Update service operation 773

5.2.6.30.4 Nnef_PDTQPolicyNegotiation_Notify service operation 773

5.2.6.31 Void 773

5.2.6.32 Nnef_MemberUESelectionAssistance service 773

5.2.6.32.1 General 773

5.2.6.32.2 Nnef_MemberUESelectionAssistance_Subscribe service operation 773

5.2.6.32.3 Nnef_MemberUESelectionAssistance_Unsubscribe service operation 774

5.2.6.32.4 Nnef_MemberUESelectionAssistance_Notify service operation 774

5.2.6.33 Nnef_AF_request_for_QoS service 774

5.2.6.33.1 General 774

5.2.6.33.2 Nnef_AF_request_for_QoS_Create service operation 775

5.2.6.33.3 Nnef_AF_request_for_QoS_Notify service operation 775

5.2.6.33.4 Nnef_AF_request_for_QoS_Revoke service operation 775

5.2.6.33.5 Nnef_AF_request_for_QoS_Update service operation 775

5.2.6.34 Nnef_DNAIMapping service 776

5.2.6.34.1 General 776

5.2.6.34.2 Nnef_DNAIMapping_Subscribe service operation 776

5.2.6.34.3 Nnef_DNAIMapping_Unsubscribe service operation 776

5.2.6.34.4 Nnef_DNAIMapping_UpdateNotify service operation 776

5.2.6.35 Nnef TrafficInfluenceData service 777

5.2.6.35.1 General 777

5.2.6.35.2 Nnef_TrafficInfluenceData_Subscribe operation 777

5.2.6.35.3 Nnef_TrafficInfluenceData_Unsubscribe service operation 777

5.2.6.35.4 Nnef_TrafficInfluenceData_Notify service operation 777

5.2.6.36 Nnef_UEAddress service 777

5.2.6.36.1 General 777

5.2.6.36.2 Nnef_UEAddress_Get operation 777

5.2.6.37 Nnef_ECSAddress service 778

5.2.6.37.1 General 778

5.2.6.37.2 Nnef_ECSAddress_Create service operation 778

5.2.6.37.3 Nnef_ECSAddress_Update service operation 778

5.2.6.37.4 Nnef_ECSAddress_Delete service operation 778

5.2.6.37.5 Nnef_ECSAddress_Subscribe service operation 779

5.2.6.37.6 Nnef_ECSAddress_Unsubscribe service operation 779

5.2.6.37.7 Nnef_ECSAddress_Notify service operation 779

5.2.6.38 Void 779

5.2.6A Void 779

5.2.7 NRF Services 780

5.2.7.1 General 780

5.2.7.2 Nnrf_NFManagement service 780

5.2.7.2.1 General 780

5.2.7.2.2 Nnrf_NFManagement_NFRegister service operation 781

5.2.7.2.3 Nnrf_NFManagement_NFUpdate service operation 784

5.2.7.2.4 Nnrf_NFManagement_NFDeregister service operation 784

5.2.7.2.5 Nnrf_NFManagement_NFStatusSubscribe service operation 784

5.2.7.2.6 Nnrf_NFManagement_NFStatusNotify service operation 786

5.2.7.2.7 Nnrf_NFManagement_NFStatusUnsubscribe service operation 786

5.2.7.3 Nnrf_NFDiscovery service 786

5.2.7.3.1 General 786

5.2.7.3.2 Nnrf_NFDiscovery_Request service operation 786

5.2.7.4 Nnrf_AccessToken_service 792

5.2.7.4.1 General 792

5.2.7.4.2 Nnrf_AccessToken_Get Service Operation 792

5.2.7.5 Nnrf_Bootstrapping service 792

5.2.7.5.1 General 792

5.2.7.5.2 Nnrf_Bootstrapping_Get service operation 792

5.2.8 SMF Services 792

5.2.8.1 General 792

5.2.8.2 Nsmf_PDUSession Service 793

5.2.8.2.1 General 793

5.2.8.2.2 Nsmf_PDUSession_Create service operation 794

5.2.8.2.3 Nsmf_PDUSession_Update service operation 795

5.2.8.2.4 Nsmf_PDUSession_Release service operation 796

5.2.8.2.5 Nsmf_PDUSession_CreateSMContext service operation 796

5.2.8.2.6 Nsmf_PDUSession_UpdateSMContext service operation 797

5.2.8.2.7 Nsmf_PDUSession_ReleaseSMContext service operation 798

5.2.8.2.8 Nsmf_PDUSession_SMContextStatusNotify service operation 799

5.2.8.2.9 Nsmf_PDUSession_StatusNotify service operation 799

5.2.8.2.10 Nsmf_PDUSession_ContextRequest service operation 799

5.2.8.2.11 Nsmf_PDUSession_ContextPush service operation 803

5.2.8.2.12 Nsmf_PDUSession_SendMOData service operation 803

5.2.8.2.13 Nsmf_PDUSession_TransferMOData service operation 804

5.2.8.2.14 Nsmf_PDUSession_TransferMTData service operation 804

5.2.8.3 Nsmf_EventExposure Service 804

5.2.8.3.1 General 804

5.2.8.3.2 Nsmf_EventExposure_Notify service operation 808

5.2.8.3.2A Nsmf_EventExposure_AppRelocationInfo service operation 808

5.2.8.3.3 Nsmf_EventExposure_Subscribe service operation 809

5.2.8.3.4 Nsmf_EventExposure_UnSubscribe service operation 809

5.2.8.4 Nsmf_NIDD Service 809

5.2.8.4.1 General 809

5.2.8.4.2 Nsmf_NIDD_Delivery service operation 809

5.2.8.5 Nsmf_TrafficCorrelation service 810

5.2.8.5.1 General 810

5.2.8.5.2 Nsmf_TrafficCorrelation_Notify service operation 810

5.2.9 SMSF Services 810

5.2.9.1 General 810

5.2.9.2 Nsmsf_SMService service 810

5.2.9.2.1 General 810

5.2.9.2.2 Nsmsf_SMService_Activate service operation 810

5.2.9.2.3 Nsmsf_SMService_Deactivate service operation 811

5.2.9.2.4 Nsmsf_SMService_UplinkSMS service operation 811

5.2.9.2.5 Nsmsf_SMService_MtForwardSm service operation 811

5.2.10 AUSF Services 811

5.2.10.1 General 811

5.2.10.2 Nausf_UEAuthentication service 812

5.2.10.2.1 General 812

5.2.10.2.2 Nausf_UEAuthentication_Authenticate service operation 812

5.2.10.2.3 Void 812

5.2.10.3 Nausf_SoRProtection service 812

5.2.10.3.1 General 812

5.2.10.3.2 Nausf_SoRProtection Protect service operation 812

5.2.10.4 Nausf_UPUProtection service 812

5.2.10.4.1 General 812

5.2.10.4.2 Nausf_UPUProtection Protect service operation 812

5.2.10.5 Void 812

5.2.11 NWDAF Services 812

5.2.11.1 Void 812

5.2.11.2 Void 813

5.2.11.3 Void 813

5.2.12 UDR Services 813

5.2.12.1 General 813

5.2.12.2 Nudr_DataManagement (DM) service 814

5.2.12.2.1 General 814

5.2.12.2.2 Nudr_DM_Query service operation 820

5.2.12.2.3 Nudr_DM_Create service operation 820

5.2.12.2.4 Nudr_DM_Delete service operation 820

5.2.12.2.5 Nudr_DM_Update service operation 820

5.2.12.2.6 Nudr_DM_Subscribe service operation 821

5.2.12.2.7 Nudr_DM_Unsubscribe service operation 821

5.2.12.2.8 Nudr_DM_Notify service operation 821

5.2.12.3 Nudr_GroupIDmap service 821

5.2.12.3.1 General 821

5.2.12.3.2 Nudr_GroupIDmap_query service operation 821

5.2.13 BSF Services 822

5.2.13.1 General 822

5.2.13.2 Nbsf_Management service 822

5.2.13.2.1 General 822

5.2.13.2.2 Nbsf_Management_Register service operation 822

5.2.13.2.3 Nbsf_Management_Deregister service operation 823

5.2.13.2.4 Nbsf_Management_Discovery service operation 823

5.2.13.2.5 Nbsf_Management_Update service operation 823

5.2.13.2.6 Nbsf_Management_Subscribe service operation 824

5.2.13.2.7 Nbsf_Management_Unsubscribe service operation 824

5.2.13.2.8 Nbsf_Management_Notify service operation 824

5.2.14 UDSF Services 825

5.2.14.1 General 825

5.2.14.2 Nudsf_UnstructuredDataManagement service 825

5.2.14.2.1 General 825

5.2.14.2.2 Nudsf_UnstructuredDataManagement_Query service operation 825

5.2.14.2.3 Nudsf_UnstructuredDataManagement_Create service operation 825

5.2.14.2.4 Nudsf_UnstructuredDataManagement_Delete service operation 826

5.2.14.2.5 Nudsf_UnstructuredDataManagement_Update service operation 826

5.2.14.2.6 Nudsf_UnstructuredDataManagement_Subscribe 826

5.2.14.2.7 Nudsf_UnstructuredDataManagement_Unsubscribe 827

5.2.14.2.8 Nudsf_UnstructuredDataManagement_Notify 827

5.2.14.3 Nudsf_Timer service 827

5.2.14.3.1 General 827

5.2.14.3.2 Nudsf_Timer_Start service operation 827

5.2.14.3.3 Nudsf_Timer_Update service operation 827

5.2.14.3.4 Nudsf_Timer_Notify service operation 828

5.2.14.3.5 Nudsf_Timer_Stop service operation 828

5.2.14.3.6 Nudsf_Timer_Search service operation 828

5.2.15 LMF Services 828

5.2.16 NSSF Services 828

5.2.16.1 General 828

5.2.16.2 Nnssf_NSSelection service 829

5.2.16.2.1 Nnssf_NSSelection_Get service operation 829

5.2.16.3 Nnssf_NSSAIAvailability service 831

5.2.16.3.1 General 831

5.2.16.3.2 Nnssf_NSSAIAvailability_Update service operation 831

5.2.16.3.3 Nnssf_NSSAIAvailability_Notify service operation 831

5.2.16.3.4 Nnssf_NSSAIAvailability_Subscribe service operation 832

5.2.16.3.5 Nnssf_NSSAIAvailability_Unsubscribe service operation 833

5.2.16.3.6 Nnssf_NSSAIAvailability_Delete service operation 834

5.2.16.4 Void 834

5.2.17 CHF Spending Limit Control Service 834

5.2.17.1 General 834

5.2.17.2 Nchf_SpendingLimitControl service 834

5.2.17.2.1 General 834

5.2.17.2.2 Nchf_SpendingLimitControl Subscribe service operation 834

5.2.17.2.3 Nchf_SpendingLimitControl Unsubscribe service operation 835

5.2.17.2.4 Nchf_SpendingLimitControl Notify service operation 835

5.2.18 UCMF Services 835

5.2.18.1 General 835

5.2.18.2 Nucmf_Provisioning service 835

5.2.18.2.1 Nucmf_Provisioning_Create service operation 835

5.2.18.2.2 Nucmf_Provisioning_Delete service operation 836

5.2.18.2.3 Nucmf_Provisioning_Update service operation 836

5.2.18.3 Nucmf_UECapabilityManagement Service 836

5.2.18.3.1 Nucmf_UECapabilityManagement Resolve service operation 836

5.2.18.3.2 Nucmf_UECapabilityManagement_Assign service operation 837

5.2.18.3.3 Nucmf_UECapabilityManagement_Subscribe service operation 837

5.2.18.3.4 Nucmf_UECapabilityManagement_Unsubscribe service operation 837

5.2.18.3.5 Nucmf_UECapabilityManagement_Notify service operation 838

5.2.19 AF Services 838

5.2.19.1 General 838

5.2.19.2 Naf_EventExposure service 838

5.2.19.2.1 General 838

5.2.19.2.2 Naf_EventExposure_Subscribe service operation 839

5.2.19.2.3 Naf_EventExposure_Unsubscribe service operation 840

5.2.19.2.4 Naf_EventExposure_Notify service operation 840

5.2.19.3 Void 840

5.2.20 NSSAAF services 840

5.2.20.1 General 840

5.2.20.2 Nnssaaf_NSSAA service 841

5.2.20.2.1 General 841

5.2.20.2.2 Nnssaaf_NSSAA_Authenticate service operation 841

5.2.20.2.3 Nnssaaf_NSSAA_Re-AuthenticationNotification service operation 841

5.2.20.2.4 Nnssaaf_NSSAA_RevocationNotification service operation 841

5.2.20.3 Nnssaaf_AIW service 841

5.2.20.3.1 General 841

5.2.20.3.2 Nnssaaf_AIW_Authenticate service operation 841

5.2.21 NSACF services 841

5.2.21.1 General 841

5.2.21.2 Nnsacf_NSAC services 842

5.2.21.2.1 General 842

5.2.21.2.2 Nnsacf_NSAC_NumOfUEsUpdate service operation 842

5.2.21.2.3 Nnsacf_NSAC_EACNotify service operation 843

5.2.21.2.4 Nnsacf_NSAC_NumOfPDUsUpdate service operation 844

5.2.21.2.5 Nnsacf_NSAC_QuotaUpdate service operation 845

5.2.21.2.6 Nnsacf_NSAC_LocalNumberUpdate service operation 845

5.2.21.3 Void 845

5.2.21.4 Nnsacf_SliceEventExposure services 845

5.2.21.4.1 General 845

5.2.21.4.2 Nnsacf_SliceEventExposure_Subscribe service operation 845

5.2.21.4.3 Nnsacf_SliceEventExposure_Unsubscribe service operation 846

5.2.21.4.4 Nnsacf_SliceEventExposure_Notify service operation 846

5.2.21.5 Void 847

5.2.22 DCCF Services 847

5.2.23 MFAF Services 847

5.2.24 ADRF Services 847

5.2.25 EASDF Services 847

5.2.26 UPF Services 847

5.2.26.1 General 847

5.2.26.2 Nupf_EventExposure Service 847

5.2.26.2.1 General 847

5.2.26.2.2 Nupf_EventExposure_Notify service operation 849

5.2.26.2.3 Nupf_EventExposure_Subscribe service operation 849

5.2.26.2.4 Nupf_EventExposure_UnSubscribe service operation 849

5.2.26.3 Nupf_GetUEPrivateIPaddrAndIdentifiers service 849

5.2.26.3.1 General 849

5.2.26.3.2 Nupf_GetPrivateUEIPaddrAndIdentifiers_Get service operation 850

5.2.27 TSCTSF Services 850

5.2.27.1 General 850

5.2.27.2 Ntsctsf_TimeSynchronization service 850

5.2.27.2.1 General 850

5.2.27.2.2 Ntsctsf_TimeSynchronization_ConfigCreate operation 850

5.2.27.2.3 Ntsctsf_TimeSynchronization_ConfigUpdate operation 851

5.2.27.2.4 Ntsctsf_TimeSynchronization_ConfigDelete operation 851

5.2.27.2.5 Ntsctsf_TimeSynchronization_ConfigUpdateNotify operation 851

5.2.27.2.6 Ntsctsf_TimeSynchronization_CapsSubscribe operation 852

5.2.27.2.7 Ntsctsf_TimeSynchronization_CapsUnsubscribe operation 853

5.2.27.2.8 Ntsctsf_TimeSynchronization_CapsNotify operation 853

5.2.27.2.9 Void 854

5.2.27.2.10 Void 854

5.2.27.2.11 Void 854

5.2.27.2.12 Void 854

5.2.27.3 Ntsctsf_QoSandTSCAssistance 854

5.2.27.3.1 General 854

5.2.27.3.2 Ntsctsf_QoSandTSCAssistance_Create operation 854

5.2.27.3.3 Ntsctsf_QoSandTSCAssistance_Update operation 854

5.2.27.3.4 Ntsctsf_QoSandTSCAssistance_Delete operation 855

5.2.27.3.5 Ntsctsf_QoSandTSCAssistance_Notify operation 855

5.2.27.3.6 Ntsctsf_QoSandTSCAssistance_Subscribe operation 855

5.2.27.3.7 Ntsctsf_QoSandTSCAssistance_Unsubscribe operation 855

5.2.27.4 Ntsctsf_ASTI service 856

5.2.27.4.1 General 856

5.2.27.4.2 Ntsctsf_ASTI_Create operation 856

5.2.27.4.3 Ntsctsf_ASTI_Update operation 856

5.2.27.4.4 Ntsctsf_ASTI_Delete operation 856

5.2.27.4.5 Ntsctsf_ASTI_Get operation 857

5.2.27.4.6 Ntsctsf_ASTI_UpdateNotify operation 857

5.2.27.4.7 Void 857

5.2.27.4.8 Void 857

Annex A (informative): Drafting rules and conventions for NF services 858

A.1 General 858

A.2 Naming 858

A.2.1 Service naming 858

A.2.2 Service operation naming 858

A.3 Representation in an information flow 859

A.4 Reference to services and service operations in procedures 860

A.5 Service and service operation description template 860

A.6 Design Guidelines for NF services 860

A.6.1 Self-Containment 860

A.6.2 Reusability 861

A.6.3 Use Independent Management Schemes 861

Annex B (informative): Drafting Rules for Information flows 862

Annex C (informative): Generating EPS PDN Connection parameters from 5G PDU Session parameters 863

Annex D (normative): UE Presence in Area of Interest 864

D.1 Determination of UE presence in Area of Interest by AMF 864

D.2 Determination of UE presence in Area of Interest by NG-RAN 865

Annex E (normative): Delegated SMF and PCF discovery in the Home Routed and specific SMF Service Areas scenarios 866

E.0 Overview 866

E.1 Delegated SMF discovery in the Home Routed scenario 866

E.2 Delegated I-SMF discovery 868

E.3 Delegated PCF discovery in the Roaming scenario 870

Annex F (informative): Information flows for 5GS integration with TSN or with Deterministic Networking 872

F.1 5GS Bridge information reporting 872

F.2 5GS Bridge configuration 874

F.3 BMCA procedure 875

F.4 5GS interworking with TSN deployed in the transport network 876

F.5 5GS DetNet node information reporting 879

F.6 5GS DetNet node configuration 879

Annex G (normative): Support of GERAN/UTRAN access by SMF+PGW-C 881

Annex H (normative): Support of EAP-based secondary authentication and authorization by DN-AAA over EPC 883

H.1 Introduction 883

H.2 Procedures 883

H.2.1 Secondary authentication and authorization by DN-AAA at PDN Connection Establishment 883

Annex I (informative): Member UE selection without the NEF assistance at the AF 886

Annex J (informative): Support for Personal IoT Networks 887

J.1 Procedure for PIN service 887

Annex K (informative): Change history 889
