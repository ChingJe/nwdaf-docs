---
spec: TS 23.288
version: 18.13.0
release: '18'
clause: contents
title: Contents
source_archive: 23288-id0.zip
source_document: 23288-id0.docx
source_archive_sha256: 7116a1f4123dcf949818ab56feb6b242eda8819877346ab045654e1d0d8c36aa
source_document_sha256: 5e4ca6e4e350f62a5d4e4443d36ae93c9df679e1e9aea20ccaa44a4b0eb3e196
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents


Foreword 10

1 Scope 11

2 References 11

3 Definitions and abbreviations 13

3.1 Definitions 13

3.2 Abbreviations 13

4 Reference Architecture for Data Analytics 13

4.1 General 13

4.2 Non-roaming architecture 14

4.2.0 General 14

4.2.1 Analytics Data Repository Function 16

4.3 Roaming architecture 17

5 Network Data Analytics Functional Description 18

5.1 General 18

5.2 NWDAF Discovery and Selection 19

5.3 Federated Learning (FL) among multiple NWDAFs 21

5A Data Collection Coordination and Delivery Functional Description 22

5A.1 General 22

5A.2 Data Collection Coordination 23

5A.3 Data Delivery 24

5A.3.0 General 24

5A.3.1 Data Delivery via the DCCF or NWDAF 24

5A.3.2 Data Delivery via a Messaging Framework 25

5A.4 Data Formatting and Processing 26

5A.5 Historical Data Handling 28

5B Analytics Data Repository Functional Description 29

5B.1 General 29

5C Analytics/ML Model Accuracy Monitoring Functional Description 30

5C.1 General 30

6 Procedures to Support Network Data Analytics 31

6.0 General 31

6.1 Procedures for analytics exposure 32

6.1.1 Analytics Subscribe/Unsubscribe 32

6.1.1.1 Analytics subscribe/unsubscribe by NWDAF service consumer 32

6.1.1.2 Analytics subscribe/unsubscribe by AFs via NEF 33

6.1.2 Analytics Request 34

6.1.2.1 Analytics request by NWDAF service consumer 34

6.1.2.2 Analytics request by AFs via NEF 35

6.1.3 Contents of Analytics Exposure 36

6.1.4 Analytics Exposure using DCCF 40

6.1.4.1 General 40

6.1.4.2 Analytics Exposure via DCCF 40

6.1.4.3 Historical Analytics Exposure via DCCF 42

6.1.4.4 Analytics Exposure via Messaging Framework 44

6.1.4.5 Historical Analytics Exposure via Messaging Framework 46

6.1.5 Analytics Exposure in Roaming Case 48

6.1.5.1 General 48

6.1.5.2 Analytics Exposure from HPLMN to VPLMN 49

6.1.5.3 Analytics Exposure from VPLMN to HPLMN 51

6.1.5.4 Contents of Analytics Exposure in roaming case 53

6.1A Analytics aggregation from multiple NWDAFs 55

6.1A.1 General 55

6.1A.2 Analytics Aggregation 55

6.1A.3 Procedure for analytics aggregation 56

6.1A.3.1 Procedure for analytics aggregation with Provision of Area of Interest 56

6.1A.3.2 Procedure for Analytics Aggregation without Provision of Area of Interest 58

6.1B Transfer of analytics context and analytics subscription 60

6.1B.1 General 60

6.1B.2 Analytics Transfer Procedures 61

6.1B.2.1 Analytics context transfer initiated by target NWDAF selected by the NWDAF service consumer 61

6.1B.2.2 Analytics Subscription Transfer initiated by source NWDAF 62

6.1B.2.3 Prepared analytics subscription transfer 65

6.1B.3 Analytics Context Transfer 68

6.1B.4 Contents of Analytics Context 69

6.1C NWDAF Registration/Deregistration in UDM 71

6.1C.1 General 71

6.1C.2 NWDAF Registration in UDM 71

6.1C.3 NWDAF De-registration from UDM 71

6.2 Procedures for Data Collection 72

6.2.1 General 72

6.2.2 Data Collection from NFs 74

6.2.2.1 General 74

6.2.2.2 Procedure for Data Collection from NFs 77

6.2.2.3 Procedure for Data Collection from AF via NEF 79

6.2.2.4 Procedure for Data Collection from NRF 80

6.2.2.5 Usage of Exposure framework by the NWDAF for Data Collection 80

6.2.3 Data Collection from OAM 81

6.2.3.1 General 81

6.2.3.2 Procedure for data collection from OAM 81

6.2.4 Correlation between network data and service data 82

6.2.5 Time coordination across multiple NWDAF instances 83

6.2.5.1 General 83

6.2.5.2 Procedure for time coordination across multiple NWDAFs 84

6.2.6 Enhanced Procedures for Data Collection 85

6.2.6.0 General 85

6.2.6.1 Bulked Data Collection 85

6.2.6.1.0 General 85

6.2.6.1.1 Services for Bulked Data Collection 86

6.2.6.2 Procedure for Data Collection from NWDAF 87

6.2.6.3 Data Collection using DCCF 90

6.2.6.3.1 General 90

6.2.6.3.2 Data Collection via DCCF 90

6.2.6.3.3 Historical Data Collection via DCCF 93

6.2.6.3.4 Data Collection via Messaging Framework 95

6.2.6.3.5 Historical Data Collection via Messaging Framework 97

6.2.6.3.6 Data collection profile registration 100

6.2.6.3.7 DCCF (re-)selection initiated by consumer 101

6.2.6.3.8 DCCF and MFAF relocation initiated by DCCF 102

6.2.7 Data Collection with Event Muting Mechanism 104

6.2.7.1 General 104

6.2.7.2 Procedure for Data Collection with Event Muting Mechanism 104

6.2.8 Data Collection from the UE Application 107

6.2.8.1 General 107

6.2.8.2 Procedure for data collection from the UE Application 108

6.2.8.2.1 Connection establishment between UE Application and AF 108

6.2.8.2.2 AF registration and discovery 108

6.2.8.2.3 Data Collection Procedure from UE 109

6.2.8.2.4 Correlation between UE data collection and the NWDAF data request 110

6.2.8.2.4a Void 113

6.2.9 User consent for analytics 113

6.2.10 Data collection by H-RE-NWDAF from V-RE-NWDAF for outbound roaming users 114

6.2.11 Data collection by V-RE-NWDAF from H-RE-NWDAF for inbound roaming users 116

6.2.12 Data Collection using LCS 117

6.2.12.1 General 117

6.2.12.2 Procedure for data collection using LCS 117

6.2.13 Rating untrusted AF data sources 118

6.2.13.1 General 118

6.2.13.2 Procedure for rating untrusted AF data sources 118

6.2.14 Analytics Collection from MDAF 120

6.2.14.1 General 120

6.2.14.2 Procedure for analytics collection from MDAF 121

6.2A Procedure for ML Model Provisioning 122

6.2A.0 General 122

6.2A.1 ML Model Subscribe/Unsubscribe 122

6.2A.2 Contents of ML Model Provisioning 123

6.2A.3 ML Model request 126

6.2B Analytics Data and ML Model Repository procedures 126

6.2B.1 General 126

6.2B.2 Historical Data and Analytics storage 127

6.2B.3 Historical Data and Analytics Storage via Notifications 129

6.2B.4 Data removal from an ADRF 133

6.2B.5 ML Model Storage in ADRF 133

6.2B.6 ML Model removal from ADRF 134

6.2B.7 ML Model retrieval from ADRF 134

6.2C Federated Learning among Multiple NWDAFs 136

6.2C.1 General 136

6.2C.2 Procedures 136

6.2C.2.1 Registration and Discovery procedure for Federated Learning 136

6.2C.2.2 General procedure for Federated Learning among Multiple NWDAF Instances 138

6.2C.2.3 Procedures for Maintaining Federated Learning Processes 140

6.2D AnLF Analytics Accuracy Monitoring Procedures 142

6.2D.1 General 142

6.2D.2 Procedures for Analytics Accuracy Information Subscription 142

6.2D.3 Procedures for Analytics Accuracy Information Request 145

6.2E MTLF-based ML Model Accuracy Monitoring 146

6.2E.1 General 146

6.2E.2 Procedure for MTLF-based ML Model Accuracy Monitoring 146

6.2E.3 Procedure for AnLF-assisted MTLF ML Models Accuracy Monitoring 149

6.2E.3.1 General 149

6.2E.3.2 Procedures for registering the monitoring of the analytics accuracy of an ML Model 149

6.2E.3.3 Procedures for monitoring the analytics accuracy of an ML Model 151

6.2F Procedure for ML Model Training 153

6.2F.1 ML Model Training Subscribe/Unsubscribe 153

6.2F.2 Contents of ML Model Training 154

6.2F.3 ML Model Training Information Request 156

6.3 Slice load level related network data analytics 157

6.3.1 General 157

6.3.2 Void 158

6.3.2A Input data 158

6.3.3 Void 159

6.3.3A Output analytics 159

6.3.4 Procedures 162

6.4 Observed Service Experience related network data analytics 163

6.4.1 General 163

6.4.2 Input Data 168

6.4.3 Output Analytics 171

6.4.4 Procedures to request Service Experience for an Application 174

6.4.5 Procedures to request Service Experience for a Network Slice 176

6.4.6 Procedures to request Service Experience for a UE 176

6.5 NF load analytics 177

6.5.1 General 177

6.5.2 Input data 178

6.5.3 Output analytics 180

6.5.4 Procedures 181

6.6 Network Performance Analytics 183

6.6.1 General 183

6.6.2 Input Data 184

6.6.3 Output Analytics 184

6.6.4 Procedures 186

6.7 UE related analytics 187

6.7.1 General 187

6.7.2 UE mobility analytics 187

6.7.2.1 General 187

6.7.2.2 Input Data 188

6.7.2.3 Output Analytics 189

6.7.2.4 Procedures 191

6.7.3 UE Communication Analytics 193

6.7.3.1 General 193

6.7.3.2 Input Data 194

6.7.3.3 Output Analytics 195

6.7.3.4 Procedures 196

6.7.4 Expected UE behavioural parameters related network data analytics 198

6.7.4.1 General 198

6.7.4.2 Input Data 199

6.7.4.3 Output Analytics 199

6.7.4.4 Procedures 200

6.7.4.4.1 NWDAF-assisted expected UE behavioural analytics 200

6.7.5 Abnormal behaviour related network data analytics 201

6.7.5.1 General 201

6.7.5.2 Input Data 202

6.7.5.3 Output Analytics 203

6.7.5.4 Procedure 205

6.8 User Data Congestion Analytics 206

6.8.1 General 206

6.8.2 Input data 207

6.8.3 Output analytics 208

6.8.4 Procedures 210

6.8.4.1 Procedure for one-time or continuous reporting of analytics for user data congestion in a geographic area 210

6.8.4.2 Procedure for one-time or continuous reporting of analytics for user data congestion for a specific UE 212

6.9 QoS Sustainability Analytics 215

6.9.1 General 215

6.9.2 Input data 217

6.9.3 Output analytics 218

6.9.4 Procedures 219

6.9.4.1 Procedure for Qos Sustainability in a coarse granularity area 219

6.9.4.2 Procedure for QoS Sustainability in a fine granularity area 220

6.10 Dispersion Analytics 221

6.10.1 General 221

6.10.2 Input Data 223

6.10.3 Output Analytics 227

6.10.3.0 General 227

6.10.3.1 Data Volume Dispersion Analytics 227

6.10.3.2 Transactions Dispersion Analytics 232

6.10.4 Dispersion Analytic Procedure 236

6.11 WLAN performance analytics 238

6.11.1 General 238

6.11.2 Input Data 239

6.11.3 Output Analytics 240

6.11.4 Procedures 242

6.12 Session Management Congestion Control Experience Analytics 243

6.12.1 General 243

6.12.2 Input Data 243

6.12.3 Output Analytics 244

6.12.4 Procedures 244

6.13 Redundant Transmission Experience related analytics 245

6.13.1 General 245

6.13.2 Input Data 246

6.13.3 Output Analytics 247

6.13.4 Procedures 248

6.13.4.1 Analytics Procedure 248

6.14 DN Performance Analytics 250

6.14.1 General 250

6.14.2 Input Data 251

6.14.3 Output Analytics 252

6.14.4 Procedures to request DN Performance Analytics for an Application 255

6.15 Void 256

6.16 PFD Determination Analytics 256

6.16.1 General 256

6.16.2 Input Data 256

6.16.3 Output Analytics 257

6.16.4 Procedures 258

6.17 Location Accuracy Analytics 259

6.17.1 General 259

6.17.2 Input Data 260

6.17.3 Output Analytics 260

6.17.4 Procedures to request Location Accuracy Analytics 263

6.18 End-to-end data volume transfer time analytics 264

6.18.1 General 264

6.18.2 Input Data 265

6.18.3 Output Analytics 266

6.18.4 Procedures 268

6.19 Relative Proximity Analytics 270

6.19.1 General 270

6.19.2 Input data 271

6.19.3 Output analytics 272

6.19.4 Procedures 273

6.20 PDU Session traffic analytics 274

6.20.1 General 274

6.20.2 Input Data 275

6.20.3 Output Analytics 276

6.20.4 Procedures 276

6.21 Movement Behaviour Analytics 278

6.21.1 General 278

6.21.2 Input data 278

6.21.3 Output analytics 278

6.21.4 Procedures 279

7 Nnwdaf Services Description 281

7.1 General 281

7.2 Nnwdaf_AnalyticsSubscription Service 285

7.2.1 General 285

7.2.2 Nnwdaf_AnalyticsSubscription_Subscribe service operation 285

7.2.3 Nnwdaf_AnalyticsSubscription_Unsubscribe service operation 286

7.2.4 Nnwdaf_AnalyticsSubscription_Notify service operation 286

7.2.5 Nnwdaf_AnalyticsSubscription_Transfer service operation 287

7.3 Nnwdaf_AnalyticsInfo service 288

7.3.1 General 288

7.3.2 Nnwdaf_AnalyticsInfo_Request service operation 288

7.3.3 Nnwdaf_AnalyticsInfo_ContextTransfer service operation 289

7.4 Nnwdaf_DataManagement Service 289

7.4.1 General 289

7.4.2 Nnwdaf_DataManagement_Subscribe service operation 289

7.4.3 Nnwdaf_DataManagement_Unsubscribe service operation 290

7.4.4 Nnwdaf_DataManagement_Notify service operation 290

7.4.5 Nnwdaf_DataManagement_Fetch service operation 291

7.5 Nnwdaf_MLModelProvision services 291

7.5.1 General 291

7.5.2 Nnwdaf_MLModelProvision_Subscribe service operation 291

7.5.3 Nnwdaf_MLModelProvision_Unsubscribe service operation 292

7.5.4 Nnwdaf_MLModelProvision_Notify service operation 292

7.6 Nnwdaf_MLModelInfo service 292

7.6.1 General 292

7.6.2 Nnwdaf_MLModelInfo_Request service operation 292

7.7 Nnwdaf_RoamingAnalytics Service 293

7.7.1 General 293

7.7.2 Nnwdaf_RoamingAnalytics_Subscribe service operation 293

7.7.3 Nnwdaf_RoamingAnalytics_Unsubscribe service operation 294

7.7.4 Nnwdaf_RoamingAnalytics_Notify service operation 294

7.7.5 Nnwdaf_RoamingAnalytics_Request service operation 295

7.8 Nnwdaf_RoamingData Service 295

7.8.1 General 295

7.8.2 Nnwdaf_RoamingData_Subscribe service operation 295

7.8.3 Nnwdaf_RoamingData_Unsubscribe service operation 296

7.8.4 Nnwdaf_RoamingData_Notify service operation 296

7.9 Nnwdaf_MLModelMonitor Service 297

7.9.1 General 297

7.9.2 Nnwdaf_MLModelMonitor_Subscribe service operation 297

7.9.3 Nnwdaf_MLModelMonitor_Unsubscribe service operation 297

7.9.4 Nnwdaf_MLModelMonitor_Notify service operation 297

7.9.5 Nnwdaf_MLModelMonitor_Register 298

7.9.6 Nnwdaf_MLModelMonitor_Deregister 299

7.10 Nnwdaf_MLModelTraining Service 299

7.10.1 General 299

7.10.2 Nnwdaf_MLModelTraining_Subscribe service operation 299

7.10.3 Nnwdaf_MLModelTraining_Unsubscribe service operation 300

7.10.4 Nnwdaf_MLModelTraining_Notify service operation 300

7.11 Nnwdaf_MLModelTrainingInfo Service 301

7.11.1 General 301

7.11.2 Nnwdaf_MLModelTrainingInfo_Request service operation 301

8 DCCF Services 302

8.1 General 302

8.2 Ndccf_DataManagement service 303

8.2.1 General 303

8.2.2 Ndccf_DataManagement_Subscribe service operation 303

8.2.3 Ndccf_DataManagement_Unsubscribe service operation 304

8.2.4 Ndccf_DataManagement_Notify service operation 304

8.2.5 Ndccf_DataManagement_Fetch service operation 305

8.2.6 Ndccf_DataManagement_Transfer service operation 305

8.3 Ndccf_ContextManagement service 305

8.3.1 General 305

8.3.2 Ndccf_ContextManagement_Register service operation 305

8.3.3 Ndccf_ContextManagement_Update service operation 306

8.3.4 Ndccf_ContextManagement_Deregister service operation 306

9 MFAF Services 306

9.1 General 306

9.2 Nmfaf_3daDataManagement service 307

9.2.1 General 307

9.2.2 Nmfaf_3daDataManagement_Configure service operation 307

9.2.3 Nmfaf_3daDataManagement_Deconfigure service operation 308

9.3 Nmfaf_3caDataManagement service 308

9.3.1 General 308

9.3.2 Nmfaf_3caDataManagement_Notify service operation 308

9.3.3 Nmfaf_3caDataManagement_Fetch service operation 308

9.4 Nmfaf_ContextManagement service 309

9.4.1 General 309

9.4.2 Nmfaf_ContextManagement_Transfer service operation 309

10 ADRF Services 309

10.1 General 309

10.2 Nadrf_DataManagement service 310

10.2.1 General 310

10.2.2 Nadrf_DataManagement_StorageRequest service operation 310

10.2.3 Nadrf_DataManagement_StorageSubscriptionRequest service operation 310

10.2.4 Nadrf_DataManagement_StorageSubscriptionRemoval service operation 311

10.2.5 Nadrf_DataManagement_RetrievalRequest service operation 311

10.2.6 Nadrf_DataManagement_RetrievalSubscribe service operation 312

10.2.7 Nadrf_DataManagement_RetrievalUnsubscribe service operation 312

10.2.8 Nadrf_DataManagement_RetrievalNotify service operation 312

10.2.9 Nadrf_DataManagement_Delete 313

10.3 Nadrf_MLModelManagement service 313

10.3.1 General 313

10.3.2 Nadrf_MLModelManagement_StorageRequest service operation 313

10.3.3 Nadrf_MLModelManagement_Delete service operation 314

10.3.4 Nadrf_MLModelManagement_RetrievalRequest service operation 314

Annex A (informative): Methods to handle NAT on IPv4 between UE and AF 316

A.1 Methods to handle NAT on IPv4 between UE and AF 316

Annex B (informative): Change history 317
