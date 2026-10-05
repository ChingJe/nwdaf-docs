---
spec: TS 29.482
version: 20.1.0
release: '20'
clause: A.17
title: A.17 MLR_FLEvents API
source_archive: 29482-k10.zip
source_document: 29482-k10.docx
content_origin: 3gpp-source
---

# A.17 MLR_FLEvents API

openapi: 3.0.0

info:

title: Machine Learning Federated Learning Events Service

version: 1.1.0-alpha.1

description: \|

ML Repository provides a service consumer to create/update/delete an MLR FL Events

Subscription and receive MLR FL Events Notifications.

© 2026, 3GPP Organizational Partners (ARIB, ATIS, CCSA, ETSI, TSDSI, TTA, TTC).

All rights reserved.

externalDocs:

description: \>

3GPP TS 29.482 v20.1.0; Service Enabler Architecture Layer for Verticals (SEAL); Artificial

Intelligence Machine Learning Enablement (AIMLE) Services; Stage 3.

url: https://www.3gpp.org/ftp/Specs/archive/29_series/29.482/

servers:

\- url: '{apiRoot}/mlr-fle/v1'

variables:

apiRoot:

default: https://example.com

description: apiRoot as defined in clause 6.5 of 3GPP TS 29.549

security:

\- {}

\- oAuth2ClientCredentials: \[\]

paths:

/subscriptions:

post:

summary: Request the creation of an MLR FL Events Subscription resource.

operationId: SubFLEvents

tags:

\- FL Events (Collection)

requestBody:

required: true

content:

application/json:

schema:

\$ref: '#/components/schemas/FlEvtsSub'

responses:

'201':

description: \>

FL Events Subscription is Created.

content:

application/json:

schema:

\$ref: '#/components/schemas/FlEvtsSub'

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

FlEvtsNotif:

'{\$request.body#/notifUri}':

post:

requestBody:

required: true

content:

application/json:

schema:

\$ref: '#/components/schemas/FlEvtsNotif'

responses:

'204':

description: \>

The MLR FL Events Event Notification is successfully received and acknowledged.

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

description: String identifying an individual FL Events resource.

required: true

schema:

type: string

get:

summary: Retrieve an existing "Individual MLR FL Events Event Subscription" resource.

operationId: RetrieveFLEvents

tags:

\- Individual FL Events (Document)

responses:

'200':

description: The requested "Individual FL Events” is retrieved.

content:

application/json:

schema:

\$ref: '#/components/schemas/FlEvtsSub'

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

summary: Update completely an existing Individual FL Events resource.

operationId: UpdateFLEvents

tags:

\- Individual FL Events (Document)

requestBody:

description: Subscription information to be completely updated.

required: true

content:

application/json:

schema:

\$ref: '#/components/schemas/FlEvtsSub'

responses:

'200':

description: OK. The configuration is updated successfully.

content:

application/json:

schema:

\$ref: '#/components/schemas/FlEvtsSub'

'204':

description: successful case but without any Content

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

summary: Modify partially an existing Individual FL Events resource.

operationId: ModifyPartiallyFLEvents

tags:

\- Individual FL Events (Document)

requestBody:

description: Subscription information to be partially modified.

required: true

content:

application/json:

schema:

\$ref: '#/components/schemas/FlEvtsSubPatch'

responses:

'200':

description: OK. The individual FL Events is partially modified.

content:

application/json:

schema:

\$ref: '#/components/schemas/FlEvtsSub'

'204':

description: successful case but without any Content

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

description: Deletes an individual FL Events.

operationId: DeleteFLEvents

tags:

\- Individual FL Events (Document)

responses:

'204':

description: The individual subscription matching subscription Id is deleted.

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

FlEvtsSub:

description: Represents the FL Events subscription information.

type: object

properties:

flMbrInfo:

\$ref: '#/components/schemas/FlMbrInfo'

notifUri:

\$ref: 'TS29122_CommonData.yaml#/components/schemas/Uri'

mlMdlInfo:

\$ref: 'TS29482_MLR_MLModelManagement.yaml#/components/schemas/MLModel'

evtInfo:

\$ref: '#/components/schemas/EvtInfo'

timeValidity:

\$ref: 'TS29122_CommonData.yaml#/components/schemas/TimeWindow'

energyReqs:

\$ref: '#/components/schemas/RenewableEnergyReqs'

suppFeat:

\$ref: 'TS29571_CommonData.yaml#/components/schemas/SupportedFeatures'

required:

\- flMbrInfo

\- notifUri

\- mlMdlInfo

\- evtInfo

\- timeValidity

FlEvtsSubPatch:

description: Represents requested partial update to the FL Events subscription information.

type: object

properties:

flMbrInfo:

\$ref: '#/components/schemas/FlMbrInfo'

notifUri:

\$ref: 'TS29122_CommonData.yaml#/components/schemas/Uri'

mlMdlInfo:

\$ref: 'TS29482_MLR_MLModelManagement.yaml#/components/schemas/MLModel'

evtInfo:

\$ref: '#/components/schemas/EvtInfo'

timeValidity:

\$ref: 'TS29122_CommonData.yaml#/components/schemas/TimeWindow'

energyReqs:

\$ref: '#/components/schemas/RenewableEnergyReqs'

FlEvtsNotif:

description: Represents the FL Events notification information.

type: object

properties:

evtId:

description: Identity of the event

type: string

evtTime:

\$ref: 'TS29122_CommonData.yaml#/components/schemas/DateTime'

evtArea:

\$ref: 'TS29122_CommonData.yaml#/components/schemas/LocationArea'

evtContentList:

type: array

items:

\$ref: '#/components/schemas/EvtContent'

minItems: 1

energyInfo:

\$ref: '#/components/schemas/RenewableEnergyInfo'

required:

\- evtId

\- evtTime

\- evtContentList

FlMbrInfo:

description: Represents the FL member information.

type: object

properties:

flMbrId:

description: Identity of the FL member

type: string

flMbrType:

\$ref: '#/components/schemas/FlMbrType'

anyOf:

\- required: \[flMbrType\]

\- required: \[flMbrId\]

EvtInfo:

description: Represents the event information.

type: object

properties:

evtId:

description: Identity of the event

type: string

evtType:

\$ref: '#/components/schemas/EvtType'

required:

\- evtType

EvtType:

description: Represents the event type information.

type: object

properties:

availFlMbr:

\$ref: '#/components/schemas/AvailFlMbr'

flMdlInfo:

\$ref: '#/components/schemas/FlMdlInfo'

flMbrLdInfo:

\$ref: 'TS29571_CommonData.yaml#/components/schemas/Uinteger'

required:

\- availFlMbr

\- flMdlInfo

\- flMbrLdInfo

FlMdlInfo:

description: Represents the FL model information.

type: object

properties:

accuracy:

\$ref: 'TS29571_CommonData.yaml#/components/schemas/Uinteger'

timeSchedule:

type: array

items:

\$ref: 'TS29122_CpProvisioning.yaml#/components/schemas/ScheduledCommunicationTime'

minItems: 1

latency:

\$ref: 'TS29571_CommonData.yaml#/components/schemas/Uinteger'

required:

\- accuracy

\- timeSchedule

\- latency

EvtContent:

description: Represents the event content information.

type: object

properties:

flMbrInfo:

\$ref: '#/components/schemas/FlMbrInfo'

flMbrEnterLeave:

\$ref: '#/components/schemas/EnterLeave'

flMbrCapabilityUpdated:

\$ref: 'TS24560_Aimles_AIMLEClientRegistration.yaml#/components/schemas/ClientCapability'

flMbrTimeAvail:

\$ref: 'TS29122_CommonData.yaml#/components/schemas/TimeWindow'

areaOfInterest:

\$ref: 'TS29122_CommonData.yaml#/components/schemas/LocationArea'

required:

\- flMbrInfo

RenewableEnergyReqs:

description: Represents the requirements for renewable eneregy.

type: object

properties:

energySource:

\$ref: '#/components/schemas/RenewableEnergySource'

energyTarget:

\$ref: '#/components/schemas/RenewableEnergyTarget'

energyFormula:

\$ref: '#/components/schemas/RenewableEnergyFormula'

energyChange:

\$ref: '#/components/schemas/RenewableEnergyChange'

required:

\- energySource

\- energyTarget

RenewableEnergyInfo:

description: Represents the information for renewable eneregy.

type: object

properties:

energySource:

\$ref: '#/components/schemas/RenewableEnergySource'

energyRatio:

\$ref: 'TS29571_CommonData.yaml#/components/schemas/Uinteger'

timeValidity:

\$ref: 'TS29122_CommonData.yaml#/components/schemas/TimeWindow'

cause:

\$ref: '#/components/schemas/RenewableEnergyCause'

energyGranularity:

\$ref: '#/components/schemas/RenewableEnergyGranularity'

required:

\- energySource

\- energyRatio

\- energyGranularity

RenewableEnergyTarget:

description: Represents the target for renewable eneregy.

type: object

properties:

targetRatio:

\$ref: 'TS29571_CommonData.yaml#/components/schemas/Uinteger'

minRatio:

\$ref: 'TS29571_CommonData.yaml#/components/schemas/Uinteger'

oneOf:

\- required: \[targetRatio\]

\- required: \[minRatio\]

\# Simple data types

\#

\# Enumerations

\#

FlMbrType:

anyOf:

\- type: string

enum:

\- FL_CLIENT

\- FL_SERVER

\- type: string

description: \>

This string provides forward-compatibility with future

extensions to the enumeration and is not used to encode

content defined in the present version of this API.

description: \|

Represents supported type of the FL member.

Possible values are:

\- FL_CLIENT: Identifies the supported FL member type is FL client.

\- FL_SERVER: Identifies the supported FL member type is FL server.

EnterLeave:

anyOf:

\- type: string

enum:

\- ENTERING

\- LEAVING

\- type: string

description: \>

This string provides forward-compatibility with future

extensions to the enumeration and is not used to encode

content defined in the present version of this API.

description: \|

Represents if the supported FL member is entering or leaving the available list.

Possible values are:

\- ENTERING: Identifies the FL member is entering the available list.

\- LEAVING: Identifies the FL member is leaving the available list.

AvailFlMbr:

anyOf:

\- type: string

enum:

\- AVAILABLE

\- UNAVAILABLE

\- type: string

description: \>

This string provides forward-compatibility with future

extensions to the enumeration and is not used to encode

content defined in the present version of this API.

description: \|

Represents the supported FL member availability

Possible values are:

\- AVAILABLE: Identifies the FL member is available.

\- UNAVAILABLE: Identifies the FL member is not available.

RenewableEnergySource:

anyOf:

\- type: string

enum:

\- WIND

\- SOLAR

\- GEOTHERMY

\- type: string

description: \>

This string provides forward-compatibility with future

extensions to the enumeration and is not used to encode

content defined in the present version of this API.

description: \|

Represents the source of the renewable energy

Possible values are:

\- WIND: Identifies the source of renewable energy is wind.

\- SOLAR: Identifies the source of renewable energy is solar.

\- GEOTHERMY: Identifies the source of renewable energy is geothermic.

RenewableEnergyFormula:

anyOf:

\- type: string

enum:

\- AVERAGE

\- MEDIAN

\- type: string

description: \>

This string provides forward-compatibility with future

extensions to the enumeration and is not used to encode

content defined in the present version of this API.

description: \|

Represents the statistical formula for calculation of renewable energy

Possible values are:

\- AVERAGE: Identifies the statistical formula for calculation of renewable energy is based

on averaging.

\- MEDIAN: Identifies the statistical formula for calculation of renewable energy is based

on median.

RenewableEnergyChange:

anyOf:

\- type: string

enum:

\- EXPECTATION

\- ACTUAL

\- type: string

description: \>

This string provides forward-compatibility with future

extensions to the enumeration and is not used to encode

content defined in the present version of this API.

description: \|

Represents the base of the change of renewable energy

Possible values are:

\- EXPECTATION: Identifies the change of renewable energy is based on expectation.

\- ACTUAL: Identifies the change of renewable energy is based on actual.

RenewableEnergyCause:

anyOf:

\- type: string

enum:

\- MOBILITY

\- WEATHER

\- COVERAGE_LOSS

\- type: string

description: \>

This string provides forward-compatibility with future

extensions to the enumeration and is not used to encode

content defined in the present version of this API.

description: \|

Represents the cause of the renewable energy

Possible values are:

\- MOBILITY: Identifies the cause for change in renewable energy is mobility.

\- WEATHER: Identifies the cause for change in renewable energy is weather.

\- COVERAGE_LOSS: Identifies the cause for change in renewable energy is loss of coverage.

RenewableEnergyGranularity:

anyOf:

\- type: string

enum:

\- FL_MEMBER

\- FL_TASK

\- FL_SERVICE

\- type: string

description: \>

This string provides forward-compatibility with future

extensions to the enumeration and is not used to encode

content defined in the present version of this API.

description: \|

Represents the granularity of the renewable energy

Possible values are:

\- FL_MEMBER: Identifies the granularity of renewable energy is for FL member.

\- FL_TASK: Identifies the granularity of renewable energy is for FL task.

\- FL_SERVICE: Identifies the granularity of renewable energy is for FL service.
