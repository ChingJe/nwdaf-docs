---
spec: TS 28.105
version: 19.6.0
release: '19'
supplemental: true
applicability: cross-release-reference-not-rel18-requirement
clause: contents
title: Contents
source_archive: 28105-j60.zip
source_document: 28105-j60.docx
source_archive_sha256: 089e1ce07dccc54d8a24698a0bba1a4ce98bbeaf15e0e61591e76cd979363c65
source_document_sha256: b19d56076f9a7e55838d64c44d15840ebdec46c773546fa3d89d232640897564
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
build_id: r18-core-20260812-rel19-ts28105-supplemental
---

# Contents

Foreword [11](#foreword)

1 Scope [13](#scope)

2 References [13](#_Toc225446454)

3 Definitions of terms, symbols and abbreviations [14](#definitions-of-terms-symbols-and-abbreviations)

3.1 Terms [14](#terms)

3.2 Symbols [15](#symbols)

3.3 Abbreviations [15](#abbreviations)

4 Concepts and overview [16](#concepts-and-overview)

4.1 Overview [16](#overview)

4.2 Management of AI/ML capabilities for RAN [17](#management-of-aiml-capabilities-for-ran)

4.3 Management of AI/ML capabilities for 5GC [17](#management-of-aiml-capabilities-for-5gc)

4.4 Management of AI/ML capabilities for MDA [17](#management-of-aiml-capabilities-for-mda)

4a AI/ML management functionality and service framework [17](#a-aiml-management-functionality-and-service-framework)

4a.0 ML model lifecycle [17](#a.0-ml-model-lifecycle)

4a.1 Functionality and service framework for ML model training [18](#a.1-functionality-and-service-framework-for-ml-model-training)

4a.2 AI/ML functionalities management scenarios (relation with managed AI/ML features) [19](#a.2-aiml-functionalities-management-scenarios-relation-with-managed-aiml-features)

5 Void [22](#void)

6 AI/ML management use cases and requirements [22](#aiml-management-use-cases-and-requirements)

6.1 ML model lifecycle management capabilities [22](#ml-model-lifecycle-management-capabilities)

6.2 Void [23](#void-1)

6.2a Void [23](#a-void)

6.2b ML model training [23](#b-ml-model-training)

6.2b.1 Description [23](#b.1-description)

6.2b.2 Use cases [23](#b.2-use-cases)

6.2b.2.1 ML model training requested by consumer [23](#b.2.1-ml-model-training-requested-by-consumer)

6.2b.2.2 ML model training initiated by producer [24](#b.2.2-ml-model-training-initiated-by-producer)

6.2b.2.3 ML model selection [24](#b.2.3-ml-model-selection)

6.2b.2.4 Managing ML model training processes [24](#b.2.4-managing-ml-model-training-processes)

6.2b.2.5 Handling errors in data and ML decisions [25](#b.2.5-handling-errors-in-data-and-ml-decisions)

6.2b.2.6 ML model joint training [25](#b.2.6-ml-model-joint-training)

6.2b.2.7 ML model validation performance reporting [26](#b.2.7-ml-model-validation-performance-reporting)

6.2b.2.8 Training data effectiveness reporting [26](#b.2.8-training-data-effectiveness-reporting)

6.2b.2.9 Performance management for ML model training [26](#b.2.9-performance-management-for-ml-model-training)

6.2b.2.9.1 Overview [26](#b.2.9.1-overview)

6.2b.2.9.2 Performance indicator selection for MLmodel training [26](#b.2.9.2-performance-indicator-selection-for-mlmodel-training)

6.2b.2.9.3 ML model performance indicators query and selection for ML model training [27](#b.2.9.3-ml-model-performance-indicators-query-and-selection-for-ml-model-training)

6.2b.2.9.4 MnS consumer policy-based selection of ML model performance indicators for ML model training [27](#b.2.9.4-mns-consumer-policy-based-selection-of-ml-model-performance-indicators-for-ml-model-training)

6.2b.2.10 ML-Knowledge-based Transfer Learning [27](#b.2.10-ml-knowledge-based-transfer-learning)

6.2b.2.10.1 Description [27](#b.2.10.1-description)

6.2b.2.10.2 Use cases [28](#b.2.10.2-use-cases)

6.2b.2.11 ML model training for multiple contexts [29](#b.2.11-ml-model-training-for-multiple-contexts)

6.2b.2.12 ML Pre-specialised training [30](#b.2.12-ml-pre-specialised-training)

6.2b.2.13 ML Fine-tuning of pre-specialised trained model [30](#b.2.13-ml-fine-tuning-of-pre-specialised-trained-model)

6.2b.2.14 Management of distributed training [30](#b.2.14-management-of-distributed-training)

6.2b.2.15 Management of Federated learning [31](#b.2.15-management-of-federated-learning)

6.2b.2.15.1 Description [31](#b.2.15.1-description)

6.2b.2.15.2 Use cases [31](#b.2.15.2-use-cases)

6.2b.2.16 Management of Reinforcement Learning [32](#b.2.16-management-of-reinforcement-learning)

6.2b.2.16.1 Description [32](#b.2.16.1-description)

6.2b.2.16.2 Use cases [33](#b.2.16.2-use-cases)

6.2b.2.17 Training data statistics [34](#b.2.17-training-data-statistics)

6.2b.3 Requirements for ML model training [35](#b.3-requirements-for-ml-model-training)

6.2c ML model testing [39](#c-ml-model-testing)

6.2c.1 Description [39](#c.1-description)

6.2c.2 Use cases [40](#c.2-use-cases)

6.2c.2.1 Consumer-requested ML model testing [40](#c.2.1-consumer-requested-ml-model-testing)

6.2c.2.2 Producer-initiated ML model testing [40](#c.2.2-producer-initiated-ml-model-testing)

6.2c.2.3 Joint testing of multiple ML models [40](#c.2.3-joint-testing-of-multiple-ml-models)

6.2c.2.4 Performance management for ML model testing [40](#c.2.4-performance-management-for-ml-model-testing)

6.2c.2.4.1 Overview [40](#c.2.4.1-overview)

6.2c.2.4.2 Performance indicator selection for ML model testing [40](#c.2.4.2-performance-indicator-selection-for-ml-model-testing)

6.2c.2.4.3 ML model performance indicators query and selection for ML model testing [41](#c.2.4.3-ml-model-performance-indicators-query-and-selection-for-ml-model-testing)

6.2c.2.4.4 MnS consumer policy-based selection of ML model performance indicators for ML model testing [41](#c.2.4.4-mns-consumer-policy-based-selection-of-ml-model-performance-indicators-for-ml-model-testing)

6.2c.3 Requirements for ML model testing [41](#c.3-requirements-for-ml-model-testing)

6.3 AI/ML inference emulation [42](#aiml-inference-emulation)

6.3.1 Description [42](#description)

6.3.2 Use cases [42](#use-cases)

6.3.2.1 AI/ML inference emulation [42](#aiml-inference-emulation-1)

6.3.2.2 ML inference emulation environment selection [42](#ml-inference-emulation-environment-selection)

6.3.3 Requirements for Managing AI/ML inference emulation [42](#requirements-for-managing-aiml-inference-emulation)

6.4 ML model deployment [43](#ml-model-deployment)

6.4.1 ML model loading [43](#ml-model-loading)

6.4.1.1 Description [43](#description-1)

6.4.1.2 Use cases [43](#use-cases-1)

6.4.1.2.1 Consumer requested ML model loading [43](#consumer-requested-ml-model-loading)

6.4.1.2.2 Control of producer-initiated ML model loading [43](#control-of-producer-initiated-ml-model-loading)

6.4.1.2.3 ML model registration [43](#ml-model-registration)

6.4.1.3 Requirements for ML model loading [43](#requirements-for-ml-model-loading)

6.5 AI/ML inference [44](#aiml-inference)

6.5.1 AI/ML inference performance management [44](#aiml-inference-performance-management)

6.5.1.1 Description [44](#description-2)

6.5.1.2 Use cases [44](#use-cases-2)

6.5.1.2.1 AI/ML inference performance evaluation [44](#aiml-inference-performance-evaluation)

6.5.1.2.2 AI/ML performance measurements selection based on MnS consumer policy [45](#aiml-performance-measurements-selection-based-on-mns-consumer-policy)

6.5.1.3 Requirements for AI/ML inference performance management [45](#requirements-for-aiml-inference-performance-management)

6.5.2 AI/ML update control [46](#aiml-update-control)

6.5.2.1 Description [46](#description-3)

6.5.2.2 Use cases [46](#use-cases-3)

6.5.2.2.1 Availability of new capabilities or ML models [46](#availability-of-new-capabilities-or-ml-models)

6.5.2.2.2 Triggering ML model update [46](#triggering-ml-model-update)

6.5.2.3 Requirements for AIML update control [46](#requirements-for-aiml-update-control)

6.5.3 AI/ML inference capabilities management [47](#aiml-inference-capabilities-management)

6.5.3.1 Description [47](#description-4)

6.5.3.2 Use cases [47](#use-cases-4)

6.5.3.2.1 Identifying capabilities of ML models [47](#identifying-capabilities-of-ml-models)

6.5.3.2.2 Mapping of the capabilities of ML models [48](#mapping-of-the-capabilities-of-ml-models)

6.5.3.3 Requirements for AI/ML inference capabilities management [48](#requirements-for-aiml-inference-capabilities-management)

6.5.4 AI/ML inference capability configuration management [49](#_CR6_5_4)

6.5.4.1 Description [49](#description-5)

6.5.4.2 Use cases [49](#use-cases-5)

6.5.4.2.1 Managing NG-RAN AI/ML-based distributed Network Energy Saving [49](#managing-ng-ran-aiml-based-distributed-network-energy-saving)

6.5.4.2.2 Managing NG-RAN AI/ML-based distributed Mobility Optimization [49](#managing-ng-ran-aiml-based-distributed-mobility-optimization)

6.5.4.2.3 Managing NG-RAN AI/ML-based distributed Load Balancing [49](#managing-ng-ran-aiml-based-distributed-load-balancing)

6.5.4.3 Requirements for AI/ML inference management [49](#requirements-for-aiml-inference-management)

6.5.5 AI/ML Inference History [50](#aiml-inference-history)

6.5.5.1 Description [50](#description-6)

6.5.5.2 Use cases [50](#use-cases-6)

6.5.5.2.1 AI/ML Inference History - tracking inferences and context [50](#aiml-inference-history---tracking-inferences-and-context)

6.5.5.3 Requirements for AI/ML Inference History [51](#requirements-for-aiml-inference-history)

6.5.6 Managing ML models in use in a live network [51](#managing-ml-models-in-use-in-a-live-network)

6.5.6.1 Description [51](#description-7)

6.5.6.2 Use cases [51](#use-cases-7)

6.5.6.2.1 Handling of underperforming ML trained models in live networks [51](#handling-of-underperforming-ml-trained-models-in-live-networks)

6.5.6.2.2 Performance monitoring of Network Functions with ML trained models in live networks [51](#performance-monitoring-of-network-functions-with-ml-trained-models-in-live-networks)

6.5.6.3 Requirements for Managing ML models in use in a live network [51](#requirements-for-managing-ml-models-in-use-in-a-live-network)

6.5.7 AI/ML inference explainability [52](#aiml-inference-explainability)

6.5.7.1 Description [52](#description-8)

6.5.7.2 Use cases [52](#use-cases-8)

6.5.7.2.1 Management of explanation in AI/ML inference [52](#management-of-explanation-in-aiml-inference)

6.5.7.3 Requirements for AI/ML inference explainability managment [52](#requirements-for-aiml-inference-explainability-managment)

7 Information model definitions for AI/ML management [53](#information-model-definitions-for-aiml-management)

7.1 Imported and associated information entities [53](#imported-and-associated-information-entities)

7.1.1 Imported information entities and local labels [53](#imported-information-entities-and-local-labels)

7.1.2 Associated information entities and local labels [53](#_CR7_1_2)

7.2 Void [53](#void-2)

7.2a Common information model definitions for AI/ML management [53](#a-common-information-model-definitions-for-aiml-management)

7.2a.1 Class diagram [53](#a.1-class-diagram)

7.2a.1.1 Relationships [53](#a.1.1-relationships)

7.2a.1.2 Inheritance [54](#a.1.2-inheritance)

7.2a.2 Class definitions [54](#a.2-class-definitions)

7.2a.2.1 MLModel [54](#a.2.1-mlmodel)

7.2a.2.1.1 Definition [54](#a.2.1.1-definition)

7.2a.2.1.2 Attributes [55](#a.2.1.2-attributes)

7.2a.2.1.3 Attribute constraints [55](#a.2.1.3-attribute-constraints)

7.2a.2.1.4 Notifications [55](#a.2.1.4-notifications)

7.2a.2.2 MLModelRepository [55](#a.2.2-mlmodelrepository)

7.2a.2.2.1 Definition [55](#a.2.2.1-definition)

7.2a.2.2.2 Attributes [55](#a.2.2.2-attributes)

7.2a.2.2.3 Attribute constraints [56](#a.2.2.3-attribute-constraints)

7.2a.2.2.4 Notifications [56](#a.2.2.4-notifications)

7.2a.2.3 MLModelCoordinationGroup [56](#a.2.3-mlmodelcoordinationgroup)

7.2a.2.3.1 Definition [56](#a.2.3.1-definition)

7.2a.2.3.2 Attributes [56](#a.2.3.2-attributes)

7.2a.2.3.3 Attribute constraints [56](#a.2.3.3-attribute-constraints)

7.2a.2.3.4 Notifications [56](#a.2.3.4-notifications)

7.3 Void [56](#void-3)

7.3a Information model definitions for AI/ML operational phases [56](#a-information-model-definitions-for-aiml-operational-phases)

7.3a.1 Information model definitions for ML model training [56](#a.1-information-model-definitions-for-ml-model-training)

7.3a.1.1 Class diagram [56](#a.1.1-class-diagram)

7.3a.1.1.1 Relationships [56](#a.1.1.1-relationships)

7.3a.1.1.2 Inheritance [57](#a.1.1.2-inheritance)

7.3a.1.2 Class definitions [57](#a.1.2-class-definitions)

7.3a.1.2.1 MLTrainingFunction [57](#a.1.2.1-mltrainingfunction)

7.3a.1.2.1.1 Definition [57](#a.1.2.1.1-definition)

7.3a.1.2.1.2 Attributes [58](#a.1.2.1.2-attributes)

7.3a.1.2.1.3 Attribute constraints [58](#a.1.2.1.3-attribute-constraints)

7.3a.1.2.1.4 Notifications [58](#a.1.2.1.4-notifications)

7.3a.1.2.2 MLTrainingRequest [58](#a.1.2.2-mltrainingrequest)

7.3a.1.2.2.1 Definition [58](#a.1.2.2.1-definition)

7.3a.1.2.2.2 Attributes [59](#a.1.2.2.2-attributes)

7.3a.1.2.2.3 Attribute constraints [60](#a.1.2.2.3-attribute-constraints)

7.3a.1.2.2.4 Notifications [60](#a.1.2.2.4-notifications)

7.3a.1.2.3 MLTrainingReport [60](#a.1.2.3-mltrainingreport)

7.3a.1.2.3.1 Definition [60](#a.1.2.3.1-definition)

7.3a.1.2.3.2 Attributes [61](#a.1.2.3.2-attributes)

7.3a.1.2.3.3 Attribute constraints [61](#a.1.2.3.3-attribute-constraints)

7.3a.1.2.3.4 Notifications [61](#a.1.2.3.4-notifications)

7.3a.1.2.4 MLTrainingProcess [61](#a.1.2.4-mltrainingprocess)

7.3a.1.2.4.1 Definition [61](#a.1.2.4.1-definition)

7.3a.1.2.4.2 Attributes [62](#a.1.2.4.2-attributes)

7.3a.1.2.4.3 Attribute constraints [63](#a.1.2.4.3-attribute-constraints)

7.3a.1.2.4.4 Notifications [63](#a.1.2.4.4-notifications)

7.3a.1b Information model definitions for ML model testing [63](#a.1b-information-model-definitions-for-ml-model-testing)

7.3a.1b.1 Class diagram [63](#a.1b.1-class-diagram)

7.3a.1b.1.1 Relationships [63](#a.1b.1.1-relationships)

7.3a.1b.1.2 Inheritance [63](#a.1b.1.2-inheritance)

7.3a.1b.2 Class definitions [64](#a.1b.2-class-definitions)

7.3a.1b.2.1 MLTestingFunction [64](#a.1b.2.1-mltestingfunction)

7.3a.1b.2.2 MLTestingRequest [64](#a.1b.2.2-mltestingrequest)

7.3a.1b.2.3 MLTestingReport [65](#a.1b.2.3-mltestingreport)

7.3a.2 Information model definitions for AI/ML inference emulation [66](#a.2-information-model-definitions-for-aiml-inference-emulation)

7.3a.2.1 Class diagram [66](#a.2.1-class-diagram)

7.3a.2.1.1 Relationships [66](#a.2.1.1-relationships)

7.3a.2.1.2 Inheritance [67](#a.2.1.2-inheritance)

7.3a.2.2 Class definitions [67](#a.2.2-class-definitions)

7.3a.2.2.1 AIMLInferenceEmulationFunction [67](#a.2.2.1-aimlinferenceemulationfunction)

7.3a.2.2.1.1 Definition [67](#a.2.2.1.1-definition)

7.3a.2.2.1.2 Attributes [67](#a.2.2.1.2-attributes)

7.3a.2.2.1.3 Attribute constraints [67](#a.2.2.1.3-attribute-constraints)

7.3a.2.2.1.4 Notifications [67](#a.2.2.1.4-notifications)

7.3a.3 Information model definitions for ML model deployment [67](#a.3-information-model-definitions-for-ml-model-deployment)

7.3a.3.1 Class diagram [67](#a.3.1-class-diagram)

7.3a.3.1.1 Relationships [67](#a.3.1.1-relationships)

7.3a.3.1.2 Inheritance [68](#a.3.1.2-inheritance)

7.3a.3.2 Class definitions [68](#a.3.2-class-definitions)

7.3a.3.2.1 MLModelLoadingRequest [68](#a.3.2.1-mlmodelloadingrequest)

7.3a.3.2.1.1 Definition [68](#a.3.2.1.1-definition)

7.3a.3.2.1.2 Attributes [69](#a.3.2.1.2-attributes)

7.3a.3.2.1.3 Attribute constraints [69](#a.3.2.1.3-attribute-constraints)

7.3a.3.2.1.4 Notifications [69](#a.3.2.1.4-notifications)

7.3a.3.2.2 MLModelLoadingPolicy [69](#a.3.2.2-mlmodelloadingpolicy)

7.3a.3.2.2.1 Definition [69](#a.3.2.2.1-definition)

7.3a.3.2.2.2 Attributes [69](#a.3.2.2.2-attributes)

7.3a.3.2.2.3 Attribute constraints [69](#a.3.2.2.3-attribute-constraints)

7.3a.3.2.2.4 Notifications [70](#a.3.2.2.4-notifications)

7.3a.3.2.3 MLModelLoadingProcess [70](#a.3.2.3-mlmodelloadingprocess)

7.3a.3.2.3.1 Definition [70](#a.3.2.3.1-definition)

7.3a.3.2.3.2 Attributes [70](#a.3.2.3.2-attributes)

7.3a.3.2.3.3 Attribute constraints [71](#a.3.2.3.3-attribute-constraints)

7.3a.3.2.3.4 Notifications [71](#a.3.2.3.4-notifications)

7.3a.4 Information model definitions for AI/ML inference [71](#a.4-information-model-definitions-for-aiml-inference)

7.3a.4.1 Class diagram [71](#a.4.1-class-diagram)

7.3a.4.1.1 Relationships [71](#a.4.1.1-relationships)

7.3a.4.1.2 Inheritance [72](#a.4.1.2-inheritance)

7.3a.4.2 Class definitions [72](#a.4.2-class-definitions)

7.3a.4.2.1 MLUpdateFunction [72](#a.4.2.1-mlupdatefunction)

7.3a.4.2.1.1 Definition [72](#a.4.2.1.1-definition)

7.3a.4.2.1.2 Attributes [73](#a.4.2.1.2-attributes)

7.3a.4.2.1.3 Attribute constraints [73](#a.4.2.1.3-attribute-constraints)

7.3a.4.2.1.4 Notifications [73](#a.4.2.1.4-notifications)

7.3a.4.2.2 MLUpdateRequest [73](#a.4.2.2-mlupdaterequest)

7.3a.4.2.2.1 Definition [73](#a.4.2.2.1-definition)

7.3a.4.2.2.2 Attributes [74](#a.4.2.2.2-attributes)

7.3a.4.2.2.3 Attribute constraints [74](#a.4.2.2.3-attribute-constraints)

7.3a.4.2.2.4 Notifications [74](#a.4.2.2.4-notifications)

7.3a.4.2.3 MLUpdateProcess [74](#a.4.2.3-mlupdateprocess)

7.3a.4.2.3.1 Definition [74](#a.4.2.3.1-definition)

7.3a.4.2.3.2 Attributes [75](#a.4.2.3.2-attributes)

7.3a.4.2.3.3 Attribute constraints [75](#a.4.2.3.3-attribute-constraints)

7.3a.4.2.3.4 Notifications [75](#a.4.2.3.4-notifications)

7.3a.4.2.4 MLUpdateReport [75](#a.4.2.4-mlupdatereport)

7.3a.4.2.4.1 Definition [75](#a.4.2.4.1-definition)

7.3a.4.2.4.2 Attributes [76](#a.4.2.4.2-attributes)

7.3a.4.2.4.3 Attribute constraints [76](#a.4.2.4.3-attribute-constraints)

7.3a.4.2.4.4 Notifications [76](#a.4.2.4.4-notifications)

7.3a.4.2.5 AIMLInferenceFunction [76](#a.4.2.5-aimlinferencefunction)

7.3a.4.2.5.1 Definition [76](#a.4.2.5.1-definition)

7.3a.4.2.5.2 Attributes [76](#a.4.2.5.2-attributes)

7.3a.4.2.5.3 Attribute constraints [77](#a.4.2.5.3-attribute-constraints)

7.3a.4.2.5.4 Notifications [77](#a.4.2.5.4-notifications)

7.3a.4.2.6 AIMLInferenceReport [77](#a.4.2.6-aimlinferencereport)

7.3a.4.2.6.1 Definition [77](#a.4.2.6.1-definition)

7.3a.4.2.6.2 Attributes [77](#a.4.2.6.2-attributes)

7.3a.4.2.6.3 Attribute constraints [77](#a.4.2.6.3-attribute-constraints)

7.3a.4.2.6.4 Notifications [77](#a.4.2.6.4-notifications)

7.4 Data type definitions [78](#data-type-definitions)

7.4.1 ModelPerformance \<\<dataType\>\> [78](#modelperformance-datatype)

7.4.1.1 Definition [78](#definition)

7.4.1.2 Attributes [78](#attributes)

7.4.1.3 Attribute constraints [78](#attribute-constraints)

7.4.1.4 Notifications [78](#notifications)

7.4.2 Void [78](#void-4)

7.4.3 MLContext \<\<dataType\>\> [78](#mlcontext-datatype)

7.4.3.1 Definition [78](#definition-1)

7.4.3.2 Attributes [78](#attributes-1)

7.4.3.3 Attribute constraints [78](#attribute-constraints-1)

7.4.3.4 Notifications [79](#notifications-1)

7.4.4 SupportedPerfIndicator \<\<dataType\>\> [79](#supportedperfindicator-datatype)

7.4.4.1 Definition [79](#definition-2)

7.4.4.2 Attributes [79](#attributes-2)

7.4.4.3 Attribute constraints [79](#attribute-constraints-2)

7.4.4.4 Notifications [79](#notifications-2)

7.4.5 AvailMLCapabilityReport \<\<dataType\>\> [79](#availmlcapabilityreport-datatype)

7.4.5.1 Definition [79](#definition-3)

7.4.5.2 Attributes [80](#attributes-3)

7.4.5.3 Attribute constraints [80](#attribute-constraints-3)

7.4.5.4 Notifications [80](#notifications-3)

7.4.6 AIMLManagementPolicy \<\<dataType\>\> [80](#aimlmanagementpolicy-datatype)

7.4.6.1 Definition [80](#definition-4)

7.4.6.2 Attributes [80](#attributes-4)

7.4.6.3 Attribute constraints [80](#attribute-constraints-4)

7.4.6.4 Notifications [80](#notifications-4)

7.4.7 ManagedActivationScope \<\<choice\>\> [80](#managedactivationscope-choice)

7.4.7.1 Definition [80](#definition-5)

7.4.7.2 Attributes [81](#attributes-5)

7.4.7.3 Attribute constraints [81](#attribute-constraints-5)

7.4.7.4 Notifications [81](#notifications-5)

7.4.8. MLCapabilityInfo \<\<dataType\>\> [81](#mlcapabilityinfo-datatype)

7.4.8.1. Definition [81](#definition-6)

7.4.8.2 Attributes [81](#attributes-6)

7.4.8.3 Attribute constraints [81](#attribute-constraints-6)

7.4.8.4 Notifications [81](#notifications-6)

7.4.9 InferenceOutput \<\<dataType\>\> [81](#inferenceoutput-datatype)

7.4.9.1 Definition [81](#definition-7)

7.4.9.2 Attributes [82](#attributes-7)

7.4.9.3 Attribute constraints [82](#attribute-constraints-7)

7.4.9.4 Notifications [82](#notifications-7)

7.4.10 AIMLInferenceName \<\<choice\>\> [82](#aimlinferencename-choice)

7.4.10.1 Definition [82](#definition-8)

7.4.10.2 Attributes [82](#attributes-8)

7.4.10.3 Attribute constraints [82](#attribute-constraints-8)

7.4.10.4 Notifications [82](#notifications-8)

7.4.11 DataStatisticalProperties \<\<dataType\>\> [83](#datastatisticalproperties-datatype)

7.4.11.1 Definition [83](#definition-9)

7.4.11.2 Attributes [83](#attributes-9)

7.4.11.3 Attribute constraints [83](#attribute-constraints-9)

7.4.11.4 Notifications [83](#notifications-9)

7.4.12 DistributedTrainingExpectation \<\<dataType\>\> [83](#distributedtrainingexpectation-datatype)

7.4.12.1 Definition [83](#definition-10)

7.4.12.2 Attributes [83](#attributes-10)

7.4.12.3 Attribute constraints [83](#_Toc225446752)

7.4.12.4 Notifications [83](#notifications-10)

7.4.13 PotentialImpactInfo \<\<dataType\>\> [83](#potentialimpactinfo-datatype)

7.4.13.1 Definition [83](#definition-11)

7.4.13.2 Attributes [84](#attributes-11)

7.4.13.3 Attribute constraints [84](#_Toc225446757)

7.4.13.4 Notifications [84](#notifications-11)

7.4.14 ImpactedPM \<\<dataType\>\> [84](#impactedpm-datatype)

7.4.14.1 Definition [84](#definition-12)

7.4.14.2 Attributes [84](#attributes-12)

7.4.14.3 Attribute constraints [84](#attribute-constraints-12)

7.4.14.4 Notifications [84](#notifications-12)

7.4.15 MLKnowledge \<\<dataType\>\> [84](#mlknowledge-datatype)

7.4.15.1 Definition [84](#definition-13)

7.4.15.2 Attributes [85](#attributes-13)

7.4.15.3 Attribute constraints [85](#attribute-constraints-13)

7.4.15.4 Notifications [85](#notifications-13)

7.4.16 EnvironmentScope \<\<choice\>\> [85](#environmentscope-choice)

7.4.16.1 Definition [85](#definition-14)

7.4.16.2 Attributes [85](#attributes-14)

7.4.16.3 Attribute constraints [85](#attribute-constraints-14)

7.4.16.4 Notifications [85](#notifications-14)

7.4.17 SupportedLearningTechnology \<\<dataType\>\> [86](#supportedlearningtechnology-datatype)

7.4.17.1 Definition [86](#definition-15)

7.4.17.2 Attributes [86](#attributes-15)

7.4.17.3 Attribute constraints [86](#attribute-constraints-15)

7.4.17.4 Notifications [86](#notifications-15)

7.4.18 RLRequirement \<\<dataType\>\> [86](#rlrequirement-datatype)

7.4.18.1 Definition [86](#definition-16)

7.4.18.2 Attributes [87](#attributes-16)

7.4.18.3 Attribute constraints [87](#attribute-constraints-16)

7.4.18.4 Notifications [87](#notifications-16)

7.4.19 ClusteringCriteria \<\<dataType\>\> [87](#clusteringcriteria-datatype)

7.4.19.1 Definition [87](#definition-17)

7.4.19.2 Attributes [87](#attributes-17)

7.4.19.3 Attribute constraints [88](#attribute-constraints-17)

7.4.19.4 Notifications [88](#notifications-17)

7.4.20 FLParticipationInfo \<\<dataType\>\> [88](#flparticipationinfo-datatype)

7.4.20.1 Definition [88](#definition-18)

7.4.20.2 Attributes [88](#attributes-18)

7.4.20.3 Attribute constraints [88](#attribute-constraints-18)

7.4.20.4 Notifications [88](#notifications-18)

7.4.21 FLRequirement \<\<dataType\>\> [88](#flrequirement-datatype)

7.4.21.1 Definition [88](#definition-19)

7.4.21.2 Attributes [89](#attributes-19)

7.4.21.3 Attribute constraints [89](#attribute-constraints-19)

7.4.21.4 Notifications [89](#notifications-19)

7.4.22 FLClientSelectionCriteria \<\<dataType\>\> [89](#flclientselectioncriteria-datatype)

7.4.22.1 Definition [89](#definition-20)

7.4.22.2 Attributes [89](#attributes-20)

7.4.22.3 Attribute constraints [89](#attribute-constraints-20)

7.4.22.4 Notifications [89](#notifications-20)

7.4.23 FLReportPerClient \<\<dataType\>\> [89](#flreportperclient-datatype)

7.4.23.1 Definition [89](#definition-21)

7.4.23.2 Attributes [90](#attributes-21)

7.4.23.3 Attribute constraints [90](#attribute-constraints-21)

7.4.23.4 Notifications [90](#notifications-21)

7.4a Enumerations [90](#a-enumerations)

7.4a.1 NgRanInferenceType \<\<enumeration\>\> [90](#a.1-ngraninferencetype-enumeration)

7.5 Attribute definitions [91](#attribute-definitions)

7.5.1 Attribute properties [91](#attribute-properties)

7.5.2 Constraints [108](#constraints)

7.6 Common notifications [108](#common-notifications)

7.6.1 Configuration notifications [108](#configuration-notifications)

8 Service components [109](#service-components)

8.0 General [109](#general)

8.1 Lifecycle management operations for AI/ML management MnS [109](#lifecycle-management-operations-for-aiml-management-mns)

9 Solution Set (SS) [110](#solution-set-ss)

9.1 OpenAPI document for provisioning MnS [110](#openapi-document-for-provisioning-mns)

9.2 OpenAPI document for AI/ML management [111](#openapi-document-for-aiml-management)

10 Stage 3 definition for AI/ML Management [111](#stage-3-definition-for-aiml-management)

10.1 RESTful HTTP-based solution set [111](#restful-http-based-solution-set)

10.1.1 ML model training [111](#ml-model-training)

10.1.2 ML model testing [112](#ml-model-testing)

10.1.3 AI/ML inference emulation [112](#aiml-inference-emulation-2)

10.1.4 ML model deployment [113](#ml-model-deployment-1)

10.1.5 AI/ML inference [114](#aiml-inference-1)

Annex A (informative): PlantUML source code for NRM class diagrams [116](#annex-a-informative-plantuml-source-code-for-nrm-class-diagrams)

A.1 General [116](#a.1-general)

A.2 PlantUML code for Figure 7.3a.1.1.1-1: NRM fragment for ML model training [116](#a.2-plantuml-code-for-figure-7.3a.1.1.1-1-nrm-fragment-for-ml-model-training)

A.3 PlantUML code for Figure 7.3a.1.1.2-1: Inheritance Hierarchy for ML model training related NRMs [117](#a.3-plantuml-code-for-figure-7.3a.1.1.2-1-inheritance-hierarchy-for-ml-model-training-related-nrms)

A.4 PlantUML code for Figure 7.2a.1.2-1: Inheritance Hierarchy for common information models for AI/ML management [118](#a.4-plantuml-code-for-figure-7.2a.1.2-1-inheritance-hierarchy-for-common-information-models-for-aiml-management)

A.5 PlantUML code for Figure 7.2a.1.1-1: Relationships for common information models for AI/ML management [118](#a.5-plantuml-code-for-figure-7.2a.1.1-1-relationships-for-common-information-models-for-aiml-management)

A.6 PlantUML code for Figure 7.3a.1.1.1-2: NRM fragment for ML model testing [118](#a.6-plantuml-code-for-figure-7.3a.1.1.1-2-nrm-fragment-for-ml-model-testing)

A.7 PlantUML code for Figure 7.3a.1.1.2-2: Inheritance Hierarchy for ML model testing related NRMs [119](#a.7-plantuml-code-for-figure-7.3a.1.1.2-2-inheritance-hierarchy-for-ml-model-testing-related-nrms)

A.8 PlantUML code for Figure 7.3a.4.1.1-1: NRM fragment for ML update [119](#a.8-plantuml-code-for-figure-7.3a.4.1.1-1-nrm-fragment-for-ml-update)

A.9 PlantUML code for Figure 7.3a.4.1.2-1: Inheritance Hierarchy for ML update related NRMs [120](#a.9-plantuml-code-for-figure-7.3a.4.1.2-1-inheritance-hierarchy-for-ml-update-related-nrms)

A.10 PlantUML code for Figure 7.3a.3.1.1-1: NRM fragment for ML model loading [120](#a.10-plantuml-code-for-figure-7.3a.3.1.1-1-nrm-fragment-for-ml-model-loading)

A.11 PlantUML code for Figure 7.3a.3.1.2-1: Inheritance Hierarchy for ML model loading related NRMs [121](#a.11-plantuml-code-for-figure-7.3a.3.1.2-1-inheritance-hierarchy-for-ml-model-loading-related-nrms)

A.12 PlantUML code for Figure 7.3a.4.1.1-2: NRM fragment for AI/ML inference function [121](#a.12-plantuml-code-for-figure-7.3a.4.1.1-2-nrm-fragment-for-aiml-inference-function)

A.13 PlantUML code for Figure 7.3a.4.1.2-2: Inheritance Hierarchy for AI/ML inference function [122](#a.13-plantuml-code-for-figure-7.3a.4.1.2-2-inheritance-hierarchy-for-aiml-inference-function)

A.14 PlantUML code for Figure 7.3a.2.1.1-1: NRM fragment for AI/ML inference emulation Control [122](#a.14-plantuml-code-for-figure-7.3a.2.1.1-1-nrm-fragment-for-aiml-inference-emulation-control)

A.15 PlantUML code for Figure 7.3a.2.1.2-1: AI/ML inference emulation Inheritance Relations [123](#a.15-plantuml-code-for-figure-7.3a.2.1.2-1-aiml-inference-emulation-inheritance-relations)

Annex B (normative): OpenAPI definition of the AI/ML NRM [124](#annex-b-normative-openapi-definition-of-the-aiml-nrm)

B.1 General [124](#b.1-general)

B.2 Solution Set (SS) definitions [124](#b.2-solution-set-ss-definitions)

B.2.1 OpenAPI document "TS28105_AiMlNrm.yaml" [124](#b.2.1-openapi-document-ts28105_aimlnrm.yaml)

Annex C (informative): Change history [125](#annex-c-informative-change-history)
