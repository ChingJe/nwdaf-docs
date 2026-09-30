---
spec: TS 23.288
version: 20.2.0
release: '20'
clause: contents
title: Contents
source_archive: 23288-k20.zip
source_document: 23288-k20.docx
content_origin: 3gpp-source
---

# Contents

Foreword 12

1 Scope 13

2 References 13

3 Definitions and abbreviations 15

3.1 Definitions 15

3.2 Abbreviations 15

4 Reference Architecture for Data Analytics 16

4.1 General 16

4.2 Non-roaming architecture 16

4.2.0 General 16

4.2.1 Analytics Data Repository Function 18

4.3 Roaming architecture 19

5 Network Data Analytics Functional Description 20

5.1 General 20

5.2 NWDAF Discovery and Selection 21

5.3 Horizontal Federated Learning (FL) among multiple NWDAFs 24

5.4 Vertical Federated Learning (VFL) 25

5.5 AF Discovery and Selection for VFL 26

5A Data Collection Coordination and Delivery Functional Description 27

5A.1 General 27

5A.2 Data Collection Coordination 27

5A.3 Data Delivery 29

5A.3.0 General 29

5A.3.1 Data Delivery via the DCCF or NWDAF 29

5A.3.2 Data Delivery via a Messaging Framework 30

5A.4 Data Formatting and Processing 31

5A.5 Historical Data Handling 33

5B Analytics Data Repository Functional Description 34

5B.1 General 34

5C Analytics/ML Model Accuracy Monitoring Functional Description 35

5C.1 General 35

6 Procedures to Support Network Data Analytics 37

6.0 General 37

6.1 Procedures for analytics exposure 37

6.1.1 Analytics Subscribe/Unsubscribe 37

6.1.1.1 Analytics subscribe/unsubscribe by NWDAF service consumer 37

6.1.1.2 Analytics subscribe/unsubscribe by AFs via NEF 38

6.1.2 Analytics Request 39

6.1.2.1 Analytics request by NWDAF service consumer 39

6.1.2.2 Analytics request by AFs via NEF 40

6.1.3 Contents of Analytics Exposure 41

6.1.4 Analytics Exposure using DCCF 46

6.1.4.1 General 46

6.1.4.2 Analytics Exposure via DCCF 46

6.1.4.3 Historical Analytics Exposure via DCCF 48

6.1.4.4 Analytics Exposure via Messaging Framework 50

6.1.4.5 Historical Analytics Exposure via Messaging Framework 52

6.1.5 Analytics Exposure in Roaming Case 54

6.1.5.1 General 54

6.1.5.2 Analytics Exposure from HPLMN to VPLMN 55

6.1.5.3 Analytics Exposure from VPLMN to HPLMN 57

6.1.5.4 Contents of Analytics Exposure in roaming case 59

6.1A Analytics aggregation from multiple NWDAFs 61

6.1A.1 General 61

6.1A.2 Analytics Aggregation 61

6.1A.3 Procedure for analytics aggregation 62

6.1A.3.1 Procedure for analytics aggregation with Provision of Area of Interest 62

6.1A.3.2 Procedure for Analytics Aggregation without Provision of Area of Interest 64

6.1B Transfer of analytics context and analytics subscription 66

6.1B.1 General 66

6.1B.2 Analytics Transfer Procedures 67

6.1B.2.1 Analytics context transfer initiated by target NWDAF selected by the NWDAF service consumer 67

6.1B.2.2 Analytics Subscription Transfer initiated by source NWDAF 68

6.1B.2.3 Prepared analytics subscription transfer 71

6.1B.3 Analytics Context Transfer 74

6.1B.4 Contents of Analytics Context 75

6.1C NWDAF Registration/Deregistration in UDM 77

6.1C.1 General 77

6.1C.2 NWDAF Registration in UDM 77

6.1C.3 NWDAF De-registration from UDM 77

6.2 Procedures for Data Collection 78

6.2.1 General 78

6.2.2 Data Collection from NFs 80

6.2.2.1 General 80

6.2.2.2 Procedure for Data Collection from NFs 83

6.2.2.3 Procedure for Data Collection from AF via NEF 85

6.2.2.4 Procedure for Data Collection from NRF 86

6.2.2.5 Usage of Exposure framework by the NWDAF for Data Collection 86

6.2.3 Data Collection from OAM 87

6.2.3.1 General 87

6.2.3.2 Procedure for data collection from OAM 88

6.2.4 Correlation between network data and service data 88

6.2.5 Time coordination across multiple NWDAF instances 89

6.2.5.1 General 89

6.2.5.2 Procedure for time coordination across multiple NWDAFs 90

6.2.6 Enhanced Procedures for Data Collection 91

6.2.6.0 General 91

6.2.6.1 Bulked Data Collection 91

6.2.6.1.0 General 91

6.2.6.1.1 Services for Bulked Data Collection 92

6.2.6.2 Procedure for Data Collection from NWDAF 93

6.2.6.3 Data Collection using DCCF 96

6.2.6.3.1 General 96

6.2.6.3.2 Data Collection via DCCF 96

6.2.6.3.3 Historical Data Collection via DCCF 99

6.2.6.3.4 Data Collection via Messaging Framework 101

6.2.6.3.5 Historical Data Collection via Messaging Framework 103

6.2.6.3.6 Data collection profile registration 106

6.2.6.3.7 DCCF (re-)selection initiated by consumer 107

6.2.6.3.8 DCCF and MFAF relocation initiated by DCCF 108

6.2.7 Data Collection with Event Muting Mechanism 110

6.2.7.1 General 110

6.2.7.2 Procedure for Data Collection with Event Muting Mechanism 110

6.2.8 Data Collection from the UE Application 113

6.2.8.1 General 113

6.2.8.2 Procedure for data collection from the UE Application 114

6.2.8.2.1 Connection establishment between UE Application and AF 114

6.2.8.2.2 AF registration and discovery 114

6.2.8.2.3 Data Collection Procedure from UE 115

6.2.8.2.4 Correlation between UE data collection and the NWDAF data request 116

6.2.8.2.4a Void 122

6.2.9 User consent for analytics 122

6.2.10 Data collection by H-RE-NWDAF from V-RE-NWDAF for outbound roaming users 123

6.2.11 Data collection by V-RE-NWDAF from H-RE-NWDAF for inbound roaming users 124

6.2.12 Data Collection using LCS 126

6.2.12.1 General 126

6.2.12.2 Procedure for data collection using LCS 126

6.2.13 Rating untrusted AF data sources 127

6.2.13.1 General 127

6.2.13.2 Procedure for rating untrusted AF data sources 127

6.2.14 Analytics Collection from MDAF 129

6.2.14.1 General 129

6.2.14.2 Procedure for analytics collection from MDAF 130

6.2A Procedure for ML Model Provisioning 131

6.2A.0 General 131

6.2A.1 ML Model Subscribe/Unsubscribe 131

6.2A.2 Contents of ML Model Provisioning 132

6.2A.3 ML Model request 135

6.2B Analytics Data and ML Model Repository procedures 136

6.2B.1 General 136

6.2B.2 Historical Data and Analytics storage 136

6.2B.3 Historical Data and Analytics Storage via Notifications 138

6.2B.4 Data removal from an ADRF 142

6.2B.5 ML Model Storage in ADRF 142

6.2B.6 ML Model removal from ADRF 143

6.2B.7 ML Model retrieval from ADRF 143

6.2C Horizontal Federated Learning among Multiple NWDAFs 145

6.2C.1 General 145

6.2C.2 Procedures 145

6.2C.2.1 Registration and Discovery procedure for Federated Learning 145

6.2C.2.2 General procedure for Federated Learning among Multiple NWDAF Instances 147

6.2C.2.3 Procedures for Maintaining Federated Learning Processes 149

6.2D AnLF Analytics Accuracy Monitoring Procedures 151

6.2D.1 General 151

6.2D.2 Procedures for Analytics Accuracy Information Subscription 152

6.2D.3 Procedures for Analytics Accuracy Information Request 155

6.2E MTLF-based ML Model Accuracy Monitoring 156

6.2E.1 General 156

6.2E.2 Procedure for MTLF-based ML Model Accuracy Monitoring 156

6.2E.3 Procedure for AnLF-assisted MTLF ML Models Accuracy Monitoring 159

6.2E.3.1 General 159

6.2E.3.2 Procedures for registering the monitoring of the analytics accuracy of an ML Model 159

6.2E.3.3 Procedures for monitoring the analytics accuracy of an ML Model 161

6.2E.4 Procedure for MTLF-based AI/ML model performance monitoring for LMF-based AI/ML Positioning 163

6.2F Procedure for ML Model Training 165

6.2F.1 ML Model Training Subscribe/Unsubscribe 165

6.2F.2 Contents of ML Model Training 166

6.2F.3 ML Model Training Information Request 168

6.2G Void 169

6.2H Vertical Federated Learning among NWDAFs and AFs 169

6.2H.1 General 169

6.2H.2 Procedures 170

6.2H.2.1 Registration and Discovery procedure for Vertical Federated Learning 170

6.2H.2.1.1 Registration and Discovery procedure for Vertical Federated Learning when NWDAF or trusted AF is acting as the VFL server 170

6.2H.2.1.2 Registration and Discovery procedure for Vertical Federated Learning when untrusted AF is acting as the VFL server 171

6.2H.2.2 Preparation procedure for Vertical Federated Learning 173

6.2H.2.2.0 General 173

6.2H.2.2.1 Preparation procedure for Vertical Federated Learning when NWDAF/trusted AF is the VFL Server 174

6.2H.2.2.2 Preparation procedure for Vertical Federated Learning when untrusted AF is the VFL server 175

6.2H.2.3 Training Procedure for Vertical Federated Learning 177

6.2H.2.3.1 Training Procedure for Vertical Federated Learning when NWDAF or trusted AF is acting as VFL server 177

6.2H.2.3.2 Training Procedure for Vertical Federated Learning untrusted AF is acting as VFL server 183

6.2H.2.4 Inference procedure for vertical federated learning 185

6.2H.2.4.1 Inference procedure for vertical federated learning when NWDAF or Trusted AF is acting as VFL server 185

6.2H.2.4.2 Inference procedure for vertical federated learning when untrusted AF is acting as VFL server 188

6.2H.2.4.3 Contents of VFL Inference service 189

6.2H.2.4.4 Contents of ML Model Inference services 190

6.2H.3 Contents of ML Model VFL Training services for Vertical Federated Learning 191

6.2H.4 Contents of ML Model Training services for Vertical Federated Learning 192

6.3 Slice load level related network data analytics 194

6.3.1 General 194

6.3.2 Void 195

6.3.2A Input data 195

6.3.3 Void 196

6.3.3A Output analytics 196

6.3.4 Procedures 199

6.4 Observed Service Experience related network data analytics 200

6.4.1 General 200

6.4.2 Input Data 205

6.4.3 Output Analytics 209

6.4.4 Procedures to request Service Experience for an Application 212

6.4.5 Procedures to request Service Experience for a Network Slice 214

6.4.6 Procedures to request Service Experience for a UE 214

6.5 NF load analytics 215

6.5.1 General 215

6.5.2 Input data 216

6.5.3 Output analytics 218

6.5.4 Procedures 219

6.6 Network Performance Analytics 221

6.6.1 General 221

6.6.2 Input Data 222

6.6.3 Output Analytics 222

6.6.4 Procedures 224

6.7 UE related analytics 225

6.7.1 General 225

6.7.2 UE mobility analytics 225

6.7.2.1 General 225

6.7.2.2 Input Data 226

6.7.2.3 Output Analytics 227

6.7.2.4 Procedures 229

6.7.3 UE Communication Analytics 231

6.7.3.1 General 231

6.7.3.2 Input Data 232

6.7.3.3 Output Analytics 233

6.7.3.4 Procedures 234

6.7.4 Expected UE behavioural parameters related network data analytics 236

6.7.4.1 General 236

6.7.4.2 Input Data 237

6.7.4.3 Output Analytics 237

6.7.4.4 Procedures 238

6.7.4.4.1 NWDAF-assisted expected UE behavioural analytics 238

6.7.5 Abnormal behaviour related network data analytics 239

6.7.5.1 General 239

6.7.5.2 Input Data 240

6.7.5.3 Output Analytics 241

6.7.5.4 Procedure 243

6.8 User Data Congestion Analytics 244

6.8.1 General 244

6.8.2 Input data 245

6.8.3 Output analytics 246

6.8.4 Procedures 248

6.8.4.1 Procedure for one-time or continuous reporting of analytics for user data congestion in a geographic area 248

6.8.4.2 Procedure for one-time or continuous reporting of analytics for user data congestion for a specific UE 250

6.9 QoS Sustainability Analytics 253

6.9.1 General 253

6.9.2 Input data 255

6.9.3 Output analytics 256

6.9.4 Procedures 257

6.9.4.1 Procedure for QoS Sustainability in a coarse granularity area 257

6.9.4.2 Procedure for QoS Sustainability in a fine granularity area 258

6.10 Dispersion Analytics 260

6.10.1 General 260

6.10.2 Input Data 261

6.10.3 Output Analytics 265

6.10.3.0 General 265

6.10.3.1 Data Volume Dispersion Analytics 265

6.10.3.2 Transactions Dispersion Analytics 270

6.10.4 Dispersion Analytic Procedure 274

6.11 WLAN performance analytics 276

6.11.1 General 276

6.11.2 Input Data 277

6.11.3 Output Analytics 278

6.11.4 Procedures 280

6.12 Session Management Congestion Control Experience Analytics 281

6.12.1 General 281

6.12.2 Input Data 281

6.12.3 Output Analytics 282

6.12.4 Procedures 282

6.13 Redundant Transmission Experience related analytics 283

6.13.1 General 283

6.13.2 Input Data 284

6.13.3 Output Analytics 285

6.13.4 Procedures 286

6.13.4.1 Analytics Procedure 286

6.14 DN Performance Analytics 288

6.14.1 General 288

6.14.2 Input Data 289

6.14.3 Output Analytics 290

6.14.4 Procedures to request DN Performance Analytics for an Application 295

6.15 Void 296

6.16 PFD Determination Analytics 296

6.16.1 General 296

6.16.2 Input Data 296

6.16.3 Output Analytics 297

6.16.4 Procedures 298

6.17 Location Accuracy Analytics 299

6.17.1 General 299

6.17.2 Input Data 300

6.17.3 Output Analytics 300

6.17.4 Procedures to request Location Accuracy Analytics 303

6.18 End-to-end data volume transfer time analytics 304

6.18.1 General 304

6.18.2 Input Data 305

6.18.3 Output Analytics 306

6.18.4 Procedures 308

6.19 Relative Proximity Analytics 310

6.19.1 General 310

6.19.2 Input data 311

6.19.3 Output analytics 312

6.19.4 Procedures 313

6.20 PDU Session traffic analytics 314

6.20.1 General 314

6.20.2 Input Data 315

6.20.3 Output Analytics 316

6.20.4 Procedures 316

6.21 Movement Behaviour Analytics 318

6.21.1 General 318

6.21.2 Input data 318

6.21.3 Output analytics 319

6.21.4 Procedures 320

6.22 Signalling Storm Analytics 321

6.22.1 General 321

6.22.2 Input data 322

6.22.3 Output analytics 327

6.22.4 Procedures 330

6.23 QoS and Policy Assistance Analytics 332

6.23.1 General 332

6.23.2 Input Data 333

6.23.3 Output Analytics 335

6.23.4 Procedures 338

6.24 Abnormal User Plane Traffic Analytics 339

6.24.1 General 339

6.24.2 Input Data 340

6.24.3 Output Analytics 343

6.24.4 Procedures 344

6.24.4.1 Procedure applicable for all consumers except for the UPF 344

6.24.4.2 Procedure for SMF subscription for abnormal traffic analytics on behalf of UPF 345

6.25 Traffic pattern analytics 346

6.25.1 General 346

6.25.2 Input data 347

6.25.3 Output analytics 347

6.25.4 Procedures 349

7 Nnwdaf Services Description 349

7.1 General 349

7.2 Nnwdaf_AnalyticsSubscription Service 354

7.2.1 General 354

7.2.2 Nnwdaf_AnalyticsSubscription_Subscribe service operation 354

7.2.3 Nnwdaf_AnalyticsSubscription_Unsubscribe service operation 355

7.2.4 Nnwdaf_AnalyticsSubscription_Notify service operation 356

7.2.5 Nnwdaf_AnalyticsSubscription_Transfer service operation 356

7.3 Nnwdaf_AnalyticsInfo service 357

7.3.1 General 357

7.3.2 Nnwdaf_AnalyticsInfo_Request service operation 358

7.3.3 Nnwdaf_AnalyticsInfo_ContextTransfer service operation 358

7.4 Nnwdaf_DataManagement Service 359

7.4.1 General 359

7.4.2 Nnwdaf_DataManagement_Subscribe service operation 359

7.4.3 Nnwdaf_DataManagement_Unsubscribe service operation 359

7.4.4 Nnwdaf_DataManagement_Notify service operation 359

7.4.5 Nnwdaf_DataManagement_Fetch service operation 360

7.5 Nnwdaf_MLModelProvision services 360

7.5.1 General 360

7.5.2 Nnwdaf_MLModelProvision_Subscribe service operation 361

7.5.3 Nnwdaf_MLModelProvision_Unsubscribe service operation 361

7.5.4 Nnwdaf_MLModelProvision_Notify service operation 361

7.6 Nnwdaf_MLModelInfo service 362

7.6.1 General 362

7.6.2 Nnwdaf_MLModelInfo_Request service operation 362

7.7 Nnwdaf_RoamingAnalytics Service 363

7.7.1 General 363

7.7.2 Nnwdaf_RoamingAnalytics_Subscribe service operation 363

7.7.3 Nnwdaf_RoamingAnalytics_Unsubscribe service operation 364

7.7.4 Nnwdaf_RoamingAnalytics_Notify service operation 364

7.7.5 Nnwdaf_RoamingAnalytics_Request service operation 364

7.8 Nnwdaf_RoamingData Service 365

7.8.1 General 365

7.8.2 Nnwdaf_RoamingData_Subscribe service operation 365

7.8.3 Nnwdaf_RoamingData_Unsubscribe service operation 366

7.8.4 Nnwdaf_RoamingData_Notify service operation 366

7.9 Nnwdaf_MLModelMonitor Service 367

7.9.1 General 367

7.9.2 Nnwdaf_MLModelMonitor_Subscribe service operation 367

7.9.3 Nnwdaf_MLModelMonitor_Unsubscribe service operation 367

7.9.4 Nnwdaf_MLModelMonitor_Notify service operation 367

7.9.5 Nnwdaf_MLModelMonitor_Register 368

7.9.6 Nnwdaf_MLModelMonitor_Deregister 368

7.10 Nnwdaf_MLModelTraining Service 369

7.10.1 General 369

7.10.2 Nnwdaf_MLModelTraining_Subscribe service operation 369

7.10.3 Nnwdaf_MLModelTraining_Unsubscribe service operation 370

7.10.4 Nnwdaf_MLModelTraining_Notify service operation 370

7.11 Nnwdaf_MLModelTrainingInfo Service 371

7.11.1 General 371

7.11.2 Nnwdaf_MLModelTrainingInfo_Request service operation 371

7.12 Nnwdaf_VFLTraining Service 372

7.12.1 General 372

7.12.2 Nnwdaf_VFLTraining_Subscribe service operation 372

7.12.3 Nnwdaf_VFLTraining_Unsubscribe service operation 373

7.12.4 Nnwdaf_VFLTraining_Notify service operation 373

7.12.5 Nnwdaf_VFLTraining_Request service operation 373

7.13 Nnwdaf_VFLInference Service 374

7.13.1 General 374

7.13.2 Nnwdaf_VFLInference_Subscribe service operation 374

7.13.3 Nnwdaf_VFLInference_Unsubscribe service operation 375

7.13.4 Nnwdaf_VFLInference_Notify service operation 375

7.13.5 Nnwdaf_VFLInference_Request service operation 375

8 DCCF Services 376

8.1 General 376

8.2 Ndccf_DataManagement service 376

8.2.1 General 376

8.2.2 Ndccf_DataManagement_Subscribe service operation 376

8.2.3 Ndccf_DataManagement_Unsubscribe service operation 377

8.2.4 Ndccf_DataManagement_Notify service operation 377

8.2.5 Ndccf_DataManagement_Fetch service operation 378

8.2.6 Ndccf_DataManagement_Transfer service operation 378

8.3 Ndccf_ContextManagement service 379

8.3.1 General 379

8.3.2 Ndccf_ContextManagement_Register service operation 379

8.3.3 Ndccf_ContextManagement_Update service operation 379

8.3.4 Ndccf_ContextManagement_Deregister service operation 379

9 MFAF Services 380

9.1 General 380

9.2 Nmfaf_3daDataManagement service 380

9.2.1 General 380

9.2.2 Nmfaf_3daDataManagement_Configure service operation 380

9.2.3 Nmfaf_3daDataManagement_Deconfigure service operation 381

9.3 Nmfaf_3caDataManagement service 381

9.3.1 General 381

9.3.2 Nmfaf_3caDataManagement_Notify service operation 381

9.3.3 Nmfaf_3caDataManagement_Fetch service operation 382

9.4 Nmfaf_ContextManagement service 382

9.4.1 General 382

9.4.2 Nmfaf_ContextManagement_Transfer service operation 382

10 ADRF Services 382

10.1 General 382

10.2 Nadrf_DataManagement service 383

10.2.1 General 383

10.2.2 Nadrf_DataManagement_StorageRequest service operation 383

10.2.3 Nadrf_DataManagement_StorageSubscriptionRequest service operation 383

10.2.4 Nadrf_DataManagement_StorageSubscriptionRemoval service operation 384

10.2.5 Nadrf_DataManagement_RetrievalRequest service operation 384

10.2.6 Nadrf_DataManagement_RetrievalSubscribe service operation 385

10.2.7 Nadrf_DataManagement_RetrievalUnsubscribe service operation 385

10.2.8 Nadrf_DataManagement_RetrievalNotify service operation 385

10.2.9 Nadrf_DataManagement_Delete 386

10.3 Nadrf_MLModelManagement service 386

10.3.1 General 386

10.3.2 Nadrf_MLModelManagement_StorageRequest service operation 386

10.3.3 Nadrf_MLModelManagement_Delete service operation 387

10.3.4 Nadrf_MLModelManagement_RetrievalRequest service operation 387

11 AF Services to support network data analytics 388

11.1 General 388

11.2 Naf_VFLTraining Service 388

11.2.1 General 388

11.2.2 Naf_VFLTraining_Subscribe service operation 389

11.2.3 Naf_VFLTraining_Unsubscribe service operation 389

11.2.4 Naf_VFLTraining_Notify service operation 389

11.2.5 Naf_VFLTraining_Request service operation 390

11.3 Naf_VFLInference Service 390

11.3.1 General 390

11.3.2 Naf_VFLInference_Subscribe service operation 390

11.3.3 Naf_VFLInference_Unsubscribe service operation 391

11.3.4 Naf_VFLInference_Notify service operation 391

11.3.5 Naf_VFLInference_Request service operation 391

11.4 Naf_Inference Service 392

11.4.1 General 392

11.4.2 Naf_Inference_Subscribe service operation 392

11.4.3 Naf_Inference_Unsubscribe service operation 393

11.4.4 Naf_Inference_Notify service operation 393

11.4.5 Naf_Inference_Request service operation 393

11.5 Naf_Training Service 394

11.5.1 General 394

11.5.2 Naf_Training_Subscribe service operation 394

11.5.3 Naf_Training_Unsubscribe service operation 394

11.5.4 Naf_Training_Notify service operation 395

12 NEF Services to support network data analytics 395

12.1 General 395

12.2 Nnef_VFLTraining Service 396

12.2.1 General 396

12.2.2 Nnef_VFLTraining_Subscribe service operation 396

12.2.3 Nnef_VFLTraining_Unsubscribe service operation 396

12.2.4 Nnef_VFLTraining_Notify service operation 396

12.2.5 Nnef_VFLTraining_Request service operation 397

12.3 Nnef_VFLInference Service 398

12.3.1 General 398

12.3.2 Nnef_VFLInference_Subscribe service operation 398

12.3.3 Nnef_VFLInference_Unsubscribe service operation 398

12.3.4 Nnef_VFLInference_Notify service operation 399

12.3.5 Nnef_VFLInference_Request service operation 399

12.4 Nnef_VFLNFDiscovery Service 399

12.4.1 General 399

12.4.2 Nnef_VFLNFDiscovery_NwdafDiscovery service operation 400

12.4.3 Nnef_VFLNFDiscovery_NwdafRelease service operation 400

12.5 Nnef_Inference Service 400

12.5.1 General 400

12.5.2 Nnef_Inference_Subscribe service operation 400

12.5.3 Nnef_Inference_Unsubscribe service operation 401

12.5.4 Nnef_Inference_Notify service operation 401

12.5.5 Nnef_Inference_Request service operation 402

12.6 Nnef_Training Service 402

12.6.1 General 402

12.6.2 Nnef_Training_Subscribe service operation 402

12.6.3 Nnef_Training_Unsubscribe service operation 403

12.6.4 Nnef_Training_Notify service operation 403

Annex A (informative): Methods to handle NAT on IPv4 between UE and AF 404

A.1 Methods to handle NAT on IPv4 between UE and AF 404

Annex B (informative): Change history 405
