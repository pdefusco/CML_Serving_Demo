# CML Serving Demo

The Cloudera AI Inference service provides a production-grade serving environment for hosting classic machine learning models, generative AI, and large language models (LLMs). It is designed to handle the challenges of production deployments, such as high availability, performance, fault tolerance, and scalability. The Cloudera AI Inference service allows data scientists and machine learning engineers to deploy their models quickly, without worrying about the infrastructure and maintenance.

In this demo you will deploy a CML Inference Cluster and a Llama2 model leveraging the CDP CLI.


### A Brief Introduction to CML Serving

Cloudera AI Inference is built with tight integration of NVIDIA NeMo Inference Microservices (NIMs), including NVIDIA Triton Inference Server, NeMo Inference Server for TensorRT LLMs, and NVIDIA NIM for LLMs, providing industry-leading inference performance on NVIDIA GPUs.

Model endpoint orchestration and management is built with the KServe model inference platform, a CNCF open source project. The platform provides standards-compliant model
inference protocols, such as Open Inference Protocol for predictive models and OpenAI API for generative AI models.

Cloudera AI Inference service is seamlessly integrated with the CML Registry standalone API enabling users to store, manage, and track their machine learning models throughout their lifecycle. This integration provides a central location to store model and application artifacts, metadata, and versions, making it easier to share and reuse ML artifacts across different teams and projects. It also simplifies the process of deploying models and applications to production, as users can select artifacts from the registry and deploy them with a single command.

#### Key Capabilities:

* Easy to use interface: This streamlines the complexities of deployment and
infrastructure, meaningfully reducing time to value for AI use cases.

* Real-time predictions: Cloudera AI Inference allows users to serve machine learning
models in real-time, providing low latency predictions for client requests.

* Monitoring and logging: Cloudera AI Inference includes functionality for monitoring and
logging, making it easier to troubleshoot issues and optimize workload performance.

* Advanced deployment patterns: Cloudera AI Inference includes functionality for
advanced deployment patterns, such as canary and blue-green deployments, and
supports A/B testing, enabling users to deploy new versions of models gradually and
compare their performance before deploying them to production.

* Hardware acceleration: Cloudera AI Inference integrates with NVIDIA to accelerate
inference performance on GPU cards.

* Model access: Cloudera AI Inference offers access to NVIDIA NeMo models and
optimized open-source models, tailored for NVIDIA hardware to increase inference
throughput and reduce latency.

* REST API: Cloudera AI Inference provides APIs for deploying, managing, and
monitoring online models. These APIs enable integration with CI/CD pipelines and other
tools used in the MLOps and LLMOps workflows.

* Command Line Interface: A CLI is provided to allow ease of use and scriptability. The
CLI is built from the REST API specification and provides the same capabilities.


### Requirements

* A CDP Public Cloud Environment (AWS or Azure OK) - Runtime Version 7.2.18-1.cdh7.2.18.p0.51297892 or above.
* A CML Workspace in CDP Public Cloud on version 2.0.45 or above.
* A CML Model Registry - make sure to sync with the CML Workspace.

### Step by Step Instructions

export DOMAIN=$(cdp ml describe-ml-serving-app --app-crn "crn:cdp:ml:us-west-1:558bc1d2-8867-4357-8524-311d51259233:mlserving:47680838-2766-4b3d-b85e-09e262e7bc46" | jq -r '.app.cluster.domainName')

export CDP_TOKEN=$(cdp iam generate-workload-auth-token --workload-name DE | jq -r '.token')

curl -H "Content-Type: application/json" -H "Authorization: Bearer ${CDP_TOKEN}" "https://${DOMAIN}/api/v1alpha1/listEndpoints" -d '{
"namespace": "serving-default"
}'

curl -H "Content-Type: application/json" -H "Authorization: Bearer ${CDP_TOKEN}" "https://ml-ad159813-0c6.se-sandb.a465-9q4k.cloudera.site/namespaces/serving-default/endpoints/llama2-70b-chat-on-2-a10g/v1/models" | jq

curl -H "Content-Type: application/json" \
-H "Authorization: Bearer ${CDP_TOKEN}" \
"https://ml-ad159813-0c6.se-sandb.a465-9q4k.cloudera.site/namespaces/serving-default/endpoints/llama2-70b-chat-on-2-a10g/v1/completions" -d '{
"prompt": "What should I do for a four day holiday trip in Spain?",
"model": "iyeb-xhn6-d979-r2od",
"max_tokens": 200,
"top_p": 1,
"n": 1,
"stream": false,
"frequency_penalty": 0.0
}'

#### Serving Cluster Deployment

#### Model Deployment

#### Inference Request

### Documentation and References

[CML Documentation](https://docs.cloudera.com/machine-learning/cloud/product/topics/ml-product-overview.html)

[CML Serving Release Notes](https://docs.cloudera.com/cdp-public-cloud-preview-features/cloud/ml-ai-inference-service/ml-ai-inference-service.pdf)
