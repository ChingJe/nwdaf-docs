---
spec: TS 28.105
version: 19.6.0
release: '19'
supplemental: true
applicability: cross-release-reference-not-rel18-requirement
clause: '3'
title: 3 Definitions of terms, symbols and abbreviations
source_archive: 28105-j60.zip
source_document: 28105-j60.docx
source_archive_sha256: 089e1ce07dccc54d8a24698a0bba1a4ce98bbeaf15e0e61591e76cd979363c65
source_document_sha256: b19d56076f9a7e55838d64c44d15840ebdec46c773546fa3d89d232640897564
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
build_id: r18-core-20260812-rel19-ts28105-supplemental
---

# 3 Definitions of terms, symbols and abbreviations

## 3.1 Terms

For the purposes of the present document, the terms given in 3GPP TR 21.905 \[1\] and the following apply. A term defined in the present document takes precedence over the definition of the same term, if any, in 3GPP TR 21.905 \[1\].

**ML model:** a manageable representation of an ML model algorithm.

NOTE 1: an ML model algorithm is a mathematical algorithm through which running a set of input data can generate a set of inference output.

NOTE 2: ML model algorithm is proprietary and not in scope for standardization and therefore not treated in this specification.

NOTE 3: ML model may include metadata. Metadata may include e.g. information related to the trained model, and applicable runtime context.

**ML model training:** a process performed by an ML training function to take training data, run it through an ML model algorithm, derive the associated loss and adjust the parameterization of that ML model iteratively based on the computed loss and generate the trained ML model.

**ML model initial training:** a process of training an initial version of an ML model.

**ML model re-training:** a process of training a previously trained version of an ML model and generate a new version.

NOTE 4: a new version of a trained ML model supports the same type of inference as the previous version of the ML model, i.e., the data type of inference input and data type of inference output remain unchanged between the two versions of the ML model, but parameter values might be different for the re-trained model.

**ML model pre-specialized training**: the process of training an ML model on a dataset not specific to any type of inference.

**ML model Fine-tuning**: the process of training a pre-specialised trained ML model to narrow its inference scope to a new single inference type, generating a new ML model.

NOTE 5: The pre-specialised trained model supports an inference scope that may be potentially adapted to support a list of inference types, such as MDA types in MDA, analytics types in NWDAF, type of AI/ML supported functions in NG-RAN, or vendor-specific extensions.

NOTE 6: The inference scope refers to a list of inference types that the ML model may be potentially adapted to support.

NOTE 7: The type of inference represents the specific type of ML inference supported by the model, such as MDA types in MDA, Analytics types in NWDAF, type of AI/ML supported functions in NG-RAN, or vendor-specific extensions.

**Distributed training:** a process of distributing the training workload across multiple ML training functions**.**

**Federated learning:** a distributed machine learning approach where the ML model is trained collaboratively by multiple ML training functions. This includes multiple FL clients, which perform training on local data, and one FL server, which aggregates model outcomes from the clients iteratively without exchanging data samples.

**Horizontal federated learning:** a federated learning technique without exchanging/sharing local data set, wherein the local data set in different HFL clients for local model training have the same feature space for different samples.

**FL Client:** a training function that trains an ML model on local data and shares only the model updates with the FL server, preserving data privacy.

**FL Server:** a function that aggregates the ML model updates from FL Clients to produce a global ML model.

**Reinforcement learning**: a machine learning approach in which an RL agent interacts with an RL environment by observing states, taking actions and receiving rewards as feedback. The RL agent learns a decision making policy by maximizing rewards over time through trial and error.

**ML model joint training:** a process of training a group of ML models.

**ML training function**: a logical function with ML model training capabilities.

**ML knowledge**: the implicit information representing experience gained by the training of an ML model

NOTE 8: Examples of experience include statistics (e.g. a distribution) or summaries (e.g. tables) indicating the ML model’s recommended output for a given set of input data.

**ML model testing:** a process of evaluating the performance of an ML model using testing data different from data used for model training and validation.

**ML model joint testing**: a process of evaluating the performance of a group of ML models using testing data different from data used for model training and validation.

**ML testing function**: a logical function with ML model testing capabilities.

**AI/ML inference**: a process of running a set of input data through a trained ML model to produce set of output data, such as predictions.

NOTE 9: The inference represents the process to realize the AI capabilities by utilizing a trained ML model and other AI enablers if needed, hence the AI/ML prefix is used when referring to inference as compared to training and testing.

**AI/ML inference function**: a logical function that employs trained ML model(s) to conduct inference.

**AI/ML inference emulation**: running the inference process to evaluate the performance of an ML model in an emulation environment before deploying it into the target environment.

**ML model deployment:** a process of making a trained ML model available for use in the target environment.

**ML model loading**: a process of making a trained ML model available to an inference function.

**AI/ML activation**: a process of enabling the inference capability of an AI/ML inference function.

**AI/ML deactivation**: a process of disabling the inference capability of an AI/ML inference function.

## 3.2 Symbols

Void.

## 3.3 Abbreviations

For the purposes of the present document, the abbreviations given in TR 21.905 \[1\] and TS 28.533 \[15\]. An abbreviation defined in the present document takes precedence over the definition of the same abbreviation, if any, in TR 21.905 \[1\] and TS 28.533 \[15\].

AI Artificial Intelligence

ML Machine Learning

FL Federated Learning

HFL Horizontal Federated Learning

VFL Vertical Federated Learning
