---
spec: TS 29.549
version: 20.1.0
release: '20'
clause: A.39
title: 'A.<span class="mark">39</span> SS_ADAE_AIMLEnergyConsumptionAnalytics API'
source_archive: 29549-k10.zip
source_document: '29549-k10_0_cover.docx, 29549-k10_1_Main-Body_s00_s06.docx, 29549-k10_2_Main-Body_s07_s09.docx, 29549-k10_3_Annexes_sA_sHistory.docx'
content_origin: 3gpp-source
---

# A.<span class="mark">39</span> SS_ADAE_AIMLEnergyConsumptionAnalytics API

openapi: 3.0.0

info:

title: SS_ADAE_AIMLEnergyConsumptionAnalytics

version: 1.0.0-alpha.1

description: \|

API for ADAE AIML Energy Consumption Analytics service.

© 2026, 3GPP Organizational Partners (ARIB, ATIS, CCSA, ETSI, TSDSI, TTA, TTC).

All rights reserved.

externalDocs:

description: \>

3GPP TS 29.549 V20.1.0 Service Enabler Architecture Layer for Verticals (SEAL);

Application Programming Interface (API) specification; Stage 3.

url: https://www.3gpp.org/ftp/Specs/archive/29_series/29.549/

security:

\- {}

\- oAuth2ClientCredentials: \[\]

servers:

\- url: '{apiRoot}/ss-adae-aeca/v1'

variables:

apiRoot:

default: https://example.com

description: apiRoot as defined in clause 6.5 of 3GPP TS 29.549

paths:

/subscriptions:

post:

summary: Create an Individual AIML Energy Consumption Analytics Subscription resource.

operationId: SubscribeAIMLEnergyConsumptionAnalytics

tags:

\- AIML Energy Consumption Analytics Subscriptions (Collection)

requestBody:

required: true

content:

application/json:

schema:

\$ref: '#/components/schemas/AimlEngConpSub'

responses:

'201':

description: \>

The Individual AIML Energy Consumption Analytics Subscription resource is created.

content:

application/json:

schema:

\$ref: '#/components/schemas/AimlEngConpSub'

headers:

Location:

description: Contains the URI of the newly created individual resource.

required: true

schema:

type: string

'200':

description: The requested AIML Energy Consumption Analytics information is returned.

content:

application/json:

schema:

\$ref: '#/components/schemas/AimlEngConpResp'

'400':

\$ref: 'TS29122_CommonData.yaml#/components/responses/400'

'401':

\$ref: 'TS29122_CommonData.yaml#/components/responses/401'

'403':

\$ref: 'TS29122_CommonData.yaml#/components/responses/403'

'404':

\$ref: 'TS29122_CommonData.yaml#/components/responses/404'

'411':

\$ref: 'TS29122_CommonData.yaml#/components/responses/411'

'413':

\$ref: 'TS29122_CommonData.yaml#/components/responses/413'

'415':

\$ref: 'TS29122_CommonData.yaml#/components/responses/415'

'429':

\$ref: 'TS29122_CommonData.yaml#/components/responses/429'

'500':

\$ref: 'TS29122_CommonData.yaml#/components/responses/500'

'503':

\$ref: 'TS29122_CommonData.yaml#/components/responses/503'

default:

\$ref: 'TS29122_CommonData.yaml#/components/responses/default'

callbacks:

NotifyAnalytics:

'{\$request.body#/notifUri}':

post:

requestBody:

required: true

content:

application/json:

schema:

\$ref: '#/components/schemas/AimlEngConpNotif'

responses:

'204':

description: The notification is successfully received.

'307':

\$ref: 'TS29122_CommonData.yaml#/components/responses/307'

'308':

\$ref: 'TS29122_CommonData.yaml#/components/responses/308'

'400':

\$ref: 'TS29122_CommonData.yaml#/components/responses/400'

'401':

\$ref: 'TS29122_CommonData.yaml#/components/responses/401'

'403':

\$ref: 'TS29122_CommonData.yaml#/components/responses/403'

'404':

\$ref: 'TS29122_CommonData.yaml#/components/responses/404'

'411':

\$ref: 'TS29122_CommonData.yaml#/components/responses/411'

'413':

\$ref: 'TS29122_CommonData.yaml#/components/responses/413'

'415':

\$ref: 'TS29122_CommonData.yaml#/components/responses/415'

'429':

\$ref: 'TS29122_CommonData.yaml#/components/responses/429'

'500':

\$ref: 'TS29122_CommonData.yaml#/components/responses/500'

'503':

\$ref: 'TS29122_CommonData.yaml#/components/responses/503'

default:

\$ref: 'TS29122_CommonData.yaml#/components/responses/default'

/subscriptions/{subscriptionId}:

parameters:

\- name: subscriptionId

in: path

description: \>

Represents the identifier of an Individual Analytics Subscription.

required: true

schema:

type: string

get:

summary: Read the Individual AIML Energy Consumption Analytics Subscription resource.

operationId: ReadAimleEnergyConsumptionSubscription

tags:

\- Individual AIML Energy Consumption Analytics Subscription (Document)

responses:

'200':

description: \>

The requested Individual AIML Energy Consumption Analytics Subscription

resource is returned.

content:

application/json:

schema:

\$ref: '#/components/schemas/AimlEngConpSub'

'400':

\$ref: 'TS29122_CommonData.yaml#/components/responses/400'

'401':

\$ref: 'TS29122_CommonData.yaml#/components/responses/401'

'403':

\$ref: 'TS29122_CommonData.yaml#/components/responses/403'

'404':

\$ref: 'TS29122_CommonData.yaml#/components/responses/404'

'406':

\$ref: 'TS29122_CommonData.yaml#/components/responses/406'

'429':

\$ref: 'TS29122_CommonData.yaml#/components/responses/429'

'500':

\$ref: 'TS29122_CommonData.yaml#/components/responses/500'

'503':

\$ref: 'TS29122_CommonData.yaml#/components/responses/503'

default:

\$ref: 'TS29122_CommonData.yaml#/components/responses/default'

put:

description: \>

Fully update the Individual AIML Energy Consumption Analytics Subscription resource.

operationId: UpdateAIMLEnergyConsumptionAnalyticsSubscription

tags:

\- Individual AIML Energy Consumption Analytics Subscription (Document)

requestBody:

required: true

content:

application/json:

schema:

\$ref: '#/components/schemas/AimlEngConpSub'

responses:

'200':

description: \>

OK. The Individual AIML Energy Consumption Analytics Subscription resource

is successfully updated, and representation of updated resource is

returned in the response body.

content:

application/json:

schema:

\$ref: '#/components/schemas/AimlEngConpSub'

'204':

description: \>

No Content. The Individual AIML Energy Consumption Analytics Subscription resource is

successfully updated and no content is returned in the response body.

'307':

\$ref: 'TS29122_CommonData.yaml#/components/responses/307'

'308':

\$ref: 'TS29122_CommonData.yaml#/components/responses/308'

'400':

\$ref: 'TS29122_CommonData.yaml#/components/responses/400'

'401':

\$ref: 'TS29122_CommonData.yaml#/components/responses/401'

'403':

\$ref: 'TS29122_CommonData.yaml#/components/responses/403'

'404':

\$ref: 'TS29122_CommonData.yaml#/components/responses/404'

'411':

\$ref: 'TS29122_CommonData.yaml#/components/responses/411'

'413':

\$ref: 'TS29122_CommonData.yaml#/components/responses/413'

'415':

\$ref: 'TS29122_CommonData.yaml#/components/responses/415'

'429':

\$ref: 'TS29122_CommonData.yaml#/components/responses/429'

'500':

\$ref: 'TS29122_CommonData.yaml#/components/responses/500'

'503':

\$ref: 'TS29122_CommonData.yaml#/components/responses/503'

default:

\$ref: 'TS29122_CommonData.yaml#/components/responses/default'

patch:

description: \>

Partially Modify the "Individual AIML Energy Consumption Analytics Subscription" resource.

operationId: ModifyAIMLEnergyConsumptionAnalyticsSubscription

tags:

\- Individual AIML Energy Consumption Analytics Subscription (Document)

requestBody:

required: true

content:

application/merge-patch+json:

schema:

\$ref: '#/components/schemas/AimlEngConpSubPatch'

responses:

'200':

description: \>

OK. The Individual AIML Energy Consumption Analytics Subscription resource is

successfully modified, and representation of the modified resource is returned

in the response body.

content:

application/json:

schema:

\$ref: '#/components/schemas/AimlEngConpSub'

'204':

description: \>

No Content. The Individual AIML Energy Consumption Analytics Subscription resource is

Successfully modified and no content is returned in the response body.

'307':

\$ref: 'TS29122_CommonData.yaml#/components/responses/307'

'308':

\$ref: 'TS29122_CommonData.yaml#/components/responses/308'

'400':

\$ref: 'TS29122_CommonData.yaml#/components/responses/400'

'401':

\$ref: 'TS29122_CommonData.yaml#/components/responses/401'

'403':

\$ref: 'TS29122_CommonData.yaml#/components/responses/403'

'404':

\$ref: 'TS29122_CommonData.yaml#/components/responses/404'

'411':

\$ref: 'TS29122_CommonData.yaml#/components/responses/411'

'413':

\$ref: 'TS29122_CommonData.yaml#/components/responses/413'

'415':

\$ref: 'TS29122_CommonData.yaml#/components/responses/415'

'429':

\$ref: 'TS29122_CommonData.yaml#/components/responses/429'

'500':

\$ref: 'TS29122_CommonData.yaml#/components/responses/500'

'503':

\$ref: 'TS29122_CommonData.yaml#/components/responses/503'

default:

\$ref: 'TS29122_CommonData.yaml#/components/responses/default'

delete:

summary: Remove the Individual AIML Energy Consumption Analytics Subscription resource.

operationId: RemoveAIMLEnergyConsumptionAnalyticsSubscription

tags:

\- Individual AIML Energy Consumption Analytics Subscription (Document)

responses:

'204':

description: \>

The Individual AIML Energy Consumption Analytics Subscription resource

matching the subscriptionId is deleted.

'307':

\$ref: 'TS29122_CommonData.yaml#/components/responses/307'

'308':

\$ref: 'TS29122_CommonData.yaml#/components/responses/308'

'400':

\$ref: 'TS29122_CommonData.yaml#/components/responses/400'

'401':

\$ref: 'TS29122_CommonData.yaml#/components/responses/401'

'403':

\$ref: 'TS29122_CommonData.yaml#/components/responses/403'

'404':

\$ref: 'TS29122_CommonData.yaml#/components/responses/404'

'429':

\$ref: 'TS29122_CommonData.yaml#/components/responses/429'

'500':

\$ref: 'TS29122_CommonData.yaml#/components/responses/500'

'503':

\$ref: 'TS29122_CommonData.yaml#/components/responses/503'

default:

\$ref: 'TS29122_CommonData.yaml#/components/responses/default'

components:

securitySchemes:

oAuth2ClientCredentials:

type: oauth2

flows:

clientCredentials:

tokenUrl: '{tokenUrl}'

scopes: {}

schemas:

AimlEngConpSub:

description: \>

Represents the AIML Energy Consumption Analytics subscription.

type: object

properties:

notifUri:

\$ref: 'TS29122_CommonData.yaml#/components/schemas/Uri'

analyticsType:

\$ref: 'TS29549_SS_ADAE_VALPerformanceAnalytics.yaml#/components/schemas/AnalyticsType'

mlModelId:

type: string

aimlOps:

type: array

minItems: 1

items:

\$ref: 'TS29482_MLR_MLModelManagement.yaml#/components/schemas/MLModelUsage'

calcMeth:

type: string

valUeIds:

type: array

minItems: 1

items:

\$ref: 'TS29549_SS_UserProfileRetrieval.yaml#/components/schemas/ValTargetUe'

dataProdIds:

type: array

minItems: 1

items:

type: string

profiles:

type: array

minItems: 1

items:

\$ref: 'TS29549_SS_AADRF_DataManagement.yaml#/components/schemas/DataProducerProfile'

confLevel:

\$ref: 'TS29571_CommonData.yaml#/components/schemas/Uinteger'

area:

\$ref: 'TS29122_CommonData.yaml#/components/schemas/LocationArea5G'

timeValidity:

\$ref: 'TS29122_CommonData.yaml#/components/schemas/TimeWindow'

repReq:

\$ref: 'TS29523_Npcf_EventExposure.yaml#/components/schemas/ReportingInformation'

suppFeat:

\$ref: 'TS29571_CommonData.yaml#/components/schemas/SupportedFeatures'

required:

\- notifUri

\- analyticsType

\- mlModelId

\- aimlOps

\- calcMeth

AimlEngConpSubPatch:

description: \>

Represents the AIML Energy Consumption Analytics subscription modification request.

type: object

properties:

notifUri:

\$ref: 'TS29122_CommonData.yaml#/components/schemas/Uri'

mlModelId:

type: string

aimlOps:

type: array

minItems: 1

items:

\$ref: 'TS29482_MLR_MLModelManagement.yaml#/components/schemas/MLModelUsage'

calcMeth:

type: string

valUeIds:

type: array

minItems: 1

items:

\$ref: 'TS29549_SS_UserProfileRetrieval.yaml#/components/schemas/ValTargetUe'

dataProdIds:

type: array

minItems: 1

items:

type: string

profiles:

type: array

minItems: 1

items:

\$ref: 'TS29549_SS_AADRF_DataManagement.yaml#/components/schemas/DataProducerProfile'

confLevel:

\$ref: 'TS29571_CommonData.yaml#/components/schemas/Uinteger'

area:

\$ref: 'TS29122_CommonData.yaml#/components/schemas/LocationArea5G'

timeValidity:

\$ref: 'TS29122_CommonData.yaml#/components/schemas/TimeWindow'

repReq:

\$ref: 'TS29523_Npcf_EventExposure.yaml#/components/schemas/ReportingInformation'

AimlEngConpNotif:

description: \>

Represents the AIML Energy Consumption Analytics notification.

type: object

properties:

engConpInfos:

type: array

minItems: 1

items:

\$ref: '#/components/schemas/EnergyConsumptionInfo'

timeValidity:

\$ref: 'TS29122_CommonData.yaml#/components/schemas/TimeWindow'

confLevel:

\$ref: 'TS29571_CommonData.yaml#/components/schemas/Uinteger'

required:

\- engConpInfos

EnergyConsumptionInfo:

description: \>

Represent the the AIML Energy Consumption Analytics information.

type: object

properties:

aimlOp:

\$ref: 'TS29482_MLR_MLModelManagement.yaml#/components/schemas/MLModelUsage'

valUeIds:

type: array

minItems: 1

items:

\$ref: 'TS29549_SS_UserProfileRetrieval.yaml#/components/schemas/ValTargetUe'

procCaps:

type: array

minItems: 1

items:

type: string

engConp:

\$ref: '#/components/schemas/EnergyConsumption'

required:

\- aimlOp

\- procCaps

EnergyConsumption:

description: \>

Represents the range of AIML Energy Consumption values.

type: object

properties:

maxConp:

\$ref: 'TS29122_MonitoringEvent.yaml#/components/schemas/EnergyInfo'

aveConp:

\$ref: 'TS29122_MonitoringEvent.yaml#/components/schemas/EnergyInfo'

minConp:

\$ref: 'TS29122_MonitoringEvent.yaml#/components/schemas/EnergyInfo'

anyOf:

\- required: \[maxConp\]

\- required: \[aveConp\]

\- required: \[minConp\]

AimlEngConpResp:

description: \>

Represents the AIML Energy Consumption Analytics response.

type: object

properties:

reports:

type: array

items:

\$ref: '#/components/schemas/AimlEngConpNotif'

minItems: 1

suppFeat:

\$ref: 'TS29571_CommonData.yaml#/components/schemas/SupportedFeatures'

required:

\- reports

\# Simple data types and Enumerations

\#
