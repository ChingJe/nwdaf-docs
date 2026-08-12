---
spec: TS 28.105
version: 19.6.0
release: '19'
supplemental: true
applicability: cross-release-reference-not-rel18-requirement
clause: Annex A
title: 'Annex A (informative): PlantUML source code for NRM class diagrams'
source_archive: 28105-j60.zip
source_document: 28105-j60.docx
source_archive_sha256: 089e1ce07dccc54d8a24698a0bba1a4ce98bbeaf15e0e61591e76cd979363c65
source_document_sha256: b19d56076f9a7e55838d64c44d15840ebdec46c773546fa3d89d232640897564
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
build_id: r18-core-20260812-rel19-ts28105-supplemental
---

# Annex A (informative): PlantUML source code for NRM class diagrams

## A.1 General

This annex contains the PlantUML source code for the NRM diagrams defined in clause 7.2a of the present document.

## A.2 PlantUML code for Figure 7.3a.1.1.1-1: NRM fragment for ML model training

@startuml

skinparam ClassStereotypeFontStyle normal

skinparam ClassBackgroundColor White

skinparam shadowing false

skinparam monochrome true

hide members

hide circle

'skinparam maxMessageSize 250

skinparam nodesep 60

class ManagedEntity \<\<ProxyClass\>\>

class MLModel \<\<InformationObjectClass\>\>

class MLModelCoordinationGroup \<\<InformationObjectClass\>\>

class MLTrainingFunction \<\<InformationObjectClass\>\>

class MLTrainingRequest \<\<InformationObjectClass\>\>

class MLTrainingReport \<\<InformationObjectClass\>\>

class MLTrainingProcess \<\<InformationObjectClass\>\>

class MLModelRepository \<\<InformationObjectClass\>\>

ManagedEntity "1" \*-- "\*" MLTrainingFunction: \<\<names\>\>

MLTrainingFunction "1" \*-- "\*" MLTrainingProcess: \<\<names\>\>

MLTrainingFunction "1" \*-- "\*" MLTrainingRequest: \<\<names\>\>

MLTrainingFunction "1" \*-- "\*" MLTrainingReport: \<\<names\>\>

MLTrainingFunction "1" \*-- "\*" ThresholdMonitor : \<\<names\>\>

MLTrainingFunction "\*" --\> "1" MLModelRepository

MLTrainingProcess "1" \<-r-\> "1" MLTrainingReport

MLTrainingProcess "1" -l-\> "0..1" MLTrainingRequest

MLTrainingRequest "\*" --\> "0..1" MLModel

MLTrainingRequest "1" -r-\> "0..1" MLModelCoordinationGroup

MLTrainingReport "1" --\> "0..1" MLModel

MLTrainingReport "1" --\> "0..1" MLModelCoordinationGroup

MLTrainingProcess "1" --\> "0..1" MLModel

MLTrainingProcess "1" --\> "0..1" MLModelCoordinationGroup

MLModel"\*" -l-\> "1" ThresholdMonitor

MLTrainingReport "1" -r\> "0..1" MLTrainingReport

note left of ManagedEntity

  This represents the following IOCs:

    SubNetwork or

    ManagedFunction or

    ManagedElement

  end note

@enduml

## A.3 PlantUML code for Figure 7.3a.1.1.2-1: Inheritance Hierarchy for ML model training related NRMs

@startuml

skinparam ClassStereotypeFontStyle normal

skinparam ClassBackgroundColor White

skinparam shadowing false

skinparam monochrome true

hide members

hide circle

'skinparam maxMessageSize 250

class Top \<\<InformationObjectClass\>\>

class ManagedFunction \<\<InformationObjectClass\>\>

class MLTrainingFunction \<\<InformationObjectClass\>\>

class MLTrainingRequest \<\<InformationObjectClass\>\>

class MLTrainingProcess \<\<InformationObjectClass\>\>

class MLTrainingReport \<\<InformationObjectClass\>\>

ManagedFunction \<\|-- MLTrainingFunction

Top \<\|-- MLTrainingRequest

Top \<\|-- MLTrainingProcess

Top \<\|-- MLTrainingReport

@enduml

## A.4 PlantUML code for Figure 7.2a.1.2-1: Inheritance Hierarchy for common information models for AI/ML management

@startuml

skinparam ClassStereotypeFontStyle normal

skinparam ClassBackgroundColor White

skinparam shadowing false

skinparam monochrome true

hide members

hide circle

'skinparam maxMessageSize 250

class Top \<\<InformationObjectClass\>\>

class MLModelRepository \<\<InformationObjectClass\>\>

class MLModel \<\<InformationObjectClass\>\>

class MLModelCoordinationGroup \<\<InformationObjectClass\>\>

Top \<\|-- MLModelRepository

Top \<\|-- MLModel

Top \<\|-- MLModelCoordinationGroup

@enduml

## A.5 PlantUML code for Figure 7.2a.1.1-1: Relationships for common information models for AI/ML management

@startuml

skinparam ClassStereotypeFontStyle normal

skinparam ClassBackgroundColor White

skinparam shadowing false

skinparam monochrome true

hide members

hide circle

'skinparam maxMessageSize 250

skinparam nodesep 60

class ManagedEntity \<\<ProxyClass\>\>

class MLModelRepository \<\<InformationObjectClass\>\>

class MLModel \<\<InformationObjectClass\>\>

class MLModelCoordinationGroup \<\<InformationObjectClass\>\>

ManagedEntity "1" \*-- "\*" MLModelRepository : \<\<names\>\>

MLModelRepository "1" \*-- "\*" MLModel: \<\<names\>\>

MLModelRepository "1" \*-- "\*" MLModelCoordinationGroup: \<\<names\>\>

MLModelCoordinationGroup "\*" -r-\> "2..\*" MLModel

note left of ManagedEntity

This represents the following IOCs:

ManagedElement or

SubNetwork

end note

@enduml

## A.6 PlantUML code for Figure 7.3a.1.1.1-2: NRM fragment for ML model testing

@startuml

skinparam ClassStereotypeFontStyle normal

skinparam ClassBackgroundColor White

skinparam shadowing false

skinparam monochrome true

hide members

hide circle

'skinparam maxMessageSize 250

class MLTestingEntity \<\<ProxyClass\>\>

class TestingFunction \<\<ProxyClass\>\>

class MLModel \<\<InformationObjectClass\>\>

class MLModelCoordinationGroup \<\<InformationObjectClass\>\>

class MLTestingFunction \<\<InformationObjectClass\>\>

class MLTestingRequest \<\<InformationObjectClass\>\>

class MLTestingReport \<\<InformationObjectClass\>\>

MLTestingEntity "1" \*-- "\*" MLTestingFunction: \<\<names\>\>

TestingFunction "1" \*-- "\*" MLTestingRequest : \<\<names\>\>

TestingFunction "1" \*-- "\*" MLTestingReport : \<\<names\>\>

MLTestingRequest "\*" --\> "0..1" MLModel

MLTestingRequest "\*" --\> "0..1" MLModelCoordinationGroup

MLTestingReport "\*" -l-\> "1" MLTestingRequest

(MLTestingRequest, MLModel) ... (MLTestingRequest, MLModelCoordinationGroup) : {xor}

note left of MLTestingEntity

Represents the following IOCs:

Subnetwork or

ManagedFunction or

ManagedElement

end note

note left of TestingFunction

Represents the following IOCs:

MLTestingFunction or

MLTrainingFunction

end note

@enduml

## A.7 PlantUML code for Figure 7.3a.1.1.2-2: Inheritance Hierarchy for ML model testing related NRMs

@startuml

skinparam ClassStereotypeFontStyle normal

skinparam ClassBackgroundColor White

skinparam shadowing false

skinparam monochrome true

hide members

hide circle

'skinparam maxMessageSize 250

class Top \<\<InformationObjectClass\>\>

class ManagedFunction \<\<InformationObjectClass\>\>

class MLTestingFunction \<\<InformationObjectClass\>\>

class MLTestingRequest \<\<InformationObjectClass\>\>

class MLTestingReport \<\<InformationObjectClass\>\>

ManagedFunction \<\|-- MLTestingFunction

Top \<\|-- MLTestingRequest

Top \<\|-- MLTestingReport

@enduml

## A.8 PlantUML code for Figure 7.3a.4.1.1-1: NRM fragment for ML update

@startuml

skinparam ClassStereotypeFontStyle normal

skinparam ClassBackgroundColor White

skinparam shadowing false

skinparam monochrome true

hide members

hide circle

'skinparam maxMessageSize 250

skinparam nodesep 60

class MLUpdateEntity \<\<ProxyClass\>\>

class MLUpdateFunction \<\<InformationObjectClass\>\>

class MLUpdateRequest \<\<InformationObjectClass\>\>

class MLUpdateProcess \<\<InformationObjectClass\>\>

class MLUpdateReport \<\<InformationObjectClass\>\>

class MLModel \<\<InformationObjectClass\>\>

MLUpdateEntity "1" \*-- "\*" MLUpdateFunction:\<\<names\>\>

MLUpdateFunction "1" \*-- "\*" MLUpdateRequest:\<\<names\>\>

MLUpdateFunction "1" \*-- "\*" MLUpdateProcess:\<\<names\>\>

MLUpdateFunction "1" \*-- "\*" MLUpdateReport:\<\<names\>\>

MLUpdateFunction "1" --\> "\*" "MLModel"

MLUpdateRequest "\*" \<-r-\> "1" "MLUpdateProcess"

MLUpdateProcess "1" \<-r-\> "1" "MLUpdateReport"

MLUpdateProcess "\*" --\> "\*" "MLModel"

MLUpdateReport "\*" --\> "\*" "MLModel"

MLUpdateRequest "\*" --\> "\*" "MLModel"

note left of MLUpdateEntity

Represents the IOCs:

SubNetwork or

ManagedFunction or

ManagementFunction

end note

@enduml

## A.9 PlantUML code for Figure 7.3a.4.1.2-1: Inheritance Hierarchy for ML update related NRMs

@startuml

skinparam ClassStereotypeFontStyle normal

skinparam ClassBackgroundColor White

skinparam shadowing false

skinparam monochrome true

hide members

hide circle

'skinparam maxMessageSize 250

class Top \<\<InformationObjectClass\>\>

class ManagedFunction \<\<InformationObjectClass\>\>

class MLUpdateFunction \<\<InformationObjectClass\>\>

class MLUpdateRequest \<\<InformationObjectClass\>\>

class MLUpdateProcess \<\<InformationObjectClass\>\>

class MLUpdateReport \<\<InformationObjectClass\>\>

ManagedFunction \<\|-- MLUpdateFunction

Top \<\|-- MLUpdateRequest

Top \<\|-- MLUpdateProcess

Top \<\|-- MLUpdateReport

@enduml

## A.10 PlantUML code for Figure 7.3a.3.1.1-1: NRM fragment for ML model loading

@startuml

skinparam ClassStereotypeFontStyle normal

skinparam ClassBackgroundColor White

skinparam shadowing false

skinparam monochrome true

hide members

hide circle

'skinparam maxMessageSize 250

class AIMLInferenceFunction \<\<InformationObjectClass\>\>

class MLModel \<\<InformationObjectClass\>\>

class MLModelLoadingRequest \<\<InformationObjectClass\>\>

class MLModelLoadingPolicy \<\<InformationObjectClass\>\>

class MLModelLoadingProcess \<\<InformationObjectClass\>\>

AIMLInferenceFunction"1" \*-- "\*" MLModelLoadingRequest : \<\<names\>\>

AIMLInferenceFunction "1" \*-- "\*" MLModelLoadingPolicy : \<\<names\>\>

AIMLInferenceFunction "1" \*-- "\*" MLModelLoadingProcess : \<\<names\>\>

MLModelLoadingRequest "1" \<-r- "\*" MLModelLoadingProcess

MLModelLoadingProcess "\*" -r-\> "1" MLModelLoadingPolicy

MLModelLoadingRequest "1" --\> "\*" MLModel

MLModelLoadingProcess "1" --\> "\*" MLModel

(MLModelLoadingProcess, MLModelLoadingRequest) ... (MLModelLoadingProcess, MLModelLoadingPolicy) : {xor}

@enduml

## A.11 PlantUML code for Figure 7.3a.3.1.2-1: Inheritance Hierarchy for ML model loading related NRMs

@startuml

skinparam ClassStereotypeFontStyle normal

skinparam ClassBackgroundColor White

skinparam shadowing false

skinparam monochrome true

hide members

hide circle

'skinparam maxMessageSize 250

class Top \<\<InformationObjectClass\>\>

class MLModelLoadingRequest \<\<InformationObjectClass\>\>

class MLModelLoadingPolicy \<\<InformationObjectClass\>\>

class MLModelLoadingProcess \<\<InformationObjectClass\>\>

Top \<\|-- MLModelLoadingRequest

Top \<\|-- MLModelLoadingPolicy

Top \<\|-- MLModelLoadingProcess

@enduml

## A.12 PlantUML code for Figure 7.3a.4.1.1-2: NRM fragment for AI/ML inference function

@startuml

skinparam ClassStereotypeFontStyle normal

skinparam ClassBackgroundColor White

skinparam shadowing false

skinparam monochrome true

hide members

hide circle

'skinparam maxMessageSize 250

skinparam nodesep 60

class AIMLInferenceFunction \<\<InformationObjectClass\>\>

class AIMLInferenceReport \<\<InformationObjectClass\>\>

class MLModel \<\<InformationObjectClass\>\>

class ManagedEntity \<\<ProxyClass\>\>

class AIMLSupportedFunction \<\<ProxyClass\>\>

ManagedEntity "1" \*-- "\*" AIMLInferenceFunction : \<\<names\>\>

AIMLInferenceFunction "\*" \<-l-\> "\*" AIMLSupportedFunction

MLModel "\*" \<-r-\> "\*" AIMLSupportedFunction

MLModel "\*" \<-r-\> "\*" AIMLInferenceFunction

MLModel "1..\*" \<-r-\> "\*" AIMLInferenceReport

AIMLInferenceFunction "1" \*-- "\*" AIMLInferenceReport : \<\<names\>\>

note right of ManagedEntity \#white

Represents the IOCs:

ManagedElement or

SubNetwork or

ManagedFunction

end note

note top of AIMLSupportedFunction \#white

Represents the IOCs:

DMROFunction or

DLBOFunction or

DESManagementFunction or

MDAFunction or

AnLFFunction or

LMFFunction

end note

@enduml

## A.13 PlantUML code for Figure 7.3a.4.1.2-2: Inheritance Hierarchy for AI/ML inference function

@startuml

skinparam ClassStereotypeFontStyle normal

skinparam ClassBackgroundColor White

skinparam shadowing false

skinparam monochrome true

hide members

hide circle

'skinparam maxMessageSize 250

class Top \<\<InformationObjectClass\>\>

class AIMLInferenceFunction \<\< InformationObjectClass \>\>

class AIMLInferenceReport \<\<InformationObjectClass\>\>

ManagedFunction \<\|-- AIMLInferenceFunction

Top \<\|-- AIMLInferenceReport

@enduml

## A.14 PlantUML code for Figure 7.3a.2.1.1-1: NRM fragment for AI/ML inference emulation Control

@startuml

scale max 350 height

skinparam ClassStereotypeFontStyle normal

skinparam ClassBackgroundColor White

skinparam shadowing false

skinparam monochrome true

hide members

hide circle

'skinparam maxMessageSize 250

class ManagedEntity \<\<ProxyClass\>\>

class AIMLInferenceEmulationFunction \<\<InformationObjectClass\>\>

class AIMLInferenceReport \<\< InformationObjectClass \>\>

ManagedEntity "1" \*-- "\*" AIMLInferenceEmulationFunction: \<\<names\>\>

AIMLInferenceEmulationFunction "1" \*-- "\*" AIMLInferenceReport : \<\<names\>\>

note left of ManagedEntity

Represents the following IOCs:

SubNetwork or

ManagedFunction or

Managed Element

end note

@enduml

## A.15 PlantUML code for Figure 7.3a.2.1.2-1: AI/ML inference emulation Inheritance Relations

@startuml

skinparam ClassStereotypeFontStyle normal

skinparam ClassBackgroundColor White

skinparam shadowing false

skinparam monochrome true

hide members

hide circle

'skinparam maxMessageSize 250

class ManagedFunction \<\<InformationObjectClass\>\>

class AIMLInferenceEmulationFunction \<\< InformationObjectClass \>\>

ManagedFunction \<\|-- AIMLInferenceEmulationFunction

@enduml
