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
* A working installation of the [beta CDP CLI](https://docs.cloudera.com/cdp-public-cloud/cloud/cli/topics/mc_beta_cdp_cli.html).


### Step by Step Instructions

All commands can be run from your local machine's terminal and assume that you have installed the beta CDP CLI.

#### Register a HF Model into the CML Model Registry

List all MLFlow Model Registries present in the CDP Environment.

```
cdp ml list-model-registries
```

Generate a CDP Token and save it as an Environment Variable:

```
export CDP_TOKEN=$(cdp iam generate-workload-auth-token --workload-name DE | jq -r '.token')
```

From the resulting list choose the model registry that is in the same environment as your Cloudera AI Inference cluster. The registry’s domain can be found in the domain field.

```
MR_DOMAIN=<domain of the registry>
```

To register a model from Hugging Face, execute the following query:

```
curl -v -H "Content-Type: application/json" \
-H "Authorization: Bearer ${CDP_TOKEN}" \
"${MR_DOMAIN}/api/v2/models" \
-d '{
  "name": "llama2_7b_chat_awq_from_hf",
    "createModelVersionRequestPayload": {
      "metadata": {
        "model_repo_type": "HF"
      },
      "downloadModelRepoRequest": {
      "source": "HF",
      "repo_id": "TheBloke/Llama-2-7B-Chat-AWQ"
      }
    }
}'
```

Take note of the model id from the response to the above command. Then use that for the ```<model_id>``` field in the following request:

```
curl -v -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${CDP_TOKEN}" \
  "${MR_DOMAIN}/api/v2/models/<model_id>
```

#### Create a Cloudera AI Inference Serving App

Run the following command to first generate the CLI skeleton.

```
cdp ml create-ml-serving-app --generate-cli-skeleton > /tmp/create-serving-app-input.json
```

Open the JSON file and customize it.

* Obtain the environmentCrn from the CDP Management Console.
* Pick a name for the appName value. Do not use underscore characters.  

AWS:

```
{
"appName": "my-cml-serving-app",
"environmentCrn": "Your-CDP-environment-CRN",
"isPrivateCluster": false,
"provisionK8sRequest": {
"instanceGroups": [
{
"instanceType": "m5.4xlarge",
"instanceCount": 1,
"rootVolume": {
"size": 256
},
"autoscaling": {
"minInstances": 0,
"maxInstances": 5,
"enabled": true
}
},
{
"instanceType": "p3.8xlarge",
"instanceCount": 1,
"rootVolume": {
"size": 1024
},
"autoscaling": {
"minInstances": 0,
"maxInstances": 5,
"enabled": true
}
}
],
"environmentCrn": "Your-CDP-environment-CRN",
"tags": [
{
"key": "experience",
"value": "cml-serving"
}
]
},
"mlservingVersion": "1.1.0-b53",
"usePublicLoadBalancer": false
}
```

Finally, pass the file to the creation command

```
cdp ml create-ml-serving-app --cli-input-json :///path/to/filecreate-serving-app-input.json
```

A successful invocation of the creation command will return the CRN of the Cloudera AI Inference App instance that is being created. The command provisions a Kubernetes cluster and installs the necessary software components.

Run the following command to obtain important metadata about your AI Inference App. Take note of the "appCrn" field.  

```
cdp ml list-ml-serving-apps
```

Set the following environment variables in order to be able to use the CDP CLI. Obtain a new Token.

```
export DOMAIN=$(cdp ml describe-ml-serving-app --app-crn <app-crn> | jq -r '.app.cluster.domainName')

export CDP_TOKEN=$(cdp iam generate-workload-auth-token --workload-name DE | jq -r '.token')
```

Retrieve the Registered Model ID and Version ID from the MLFlow Registry UI. Then create a JSON file with the following model definition:

```
vi ./path/to/file/model-spec-cml-registry.json

{
  "namespace": "serving-default",
  "name": "llama2-7b-chat-awq-from-hf",
  "source": {
    "registry_source": {
    "version": <model-version-id-from-mlflow-registry-ui>,
    "model_id": "<registered-model-id-from-mlflow-registry-ui>"
    }
  },
  "autoscaling": {
  "min_replicas": "0",
  "max_replicas": "4"
  },
  "resources": {
  "num_gpus": "1"
  }
}
```

Create the model endpoint:

```
curl -v -H "Content-Type: application/json" -H "Authorization: Bearer ${CDP_TOKEN}" "https://${DOMAIN}/api/v1alpha1/deployEndpoint" -d @./path/to/file/model-spec-cml-registry.json
```

You should now see an entry in the Model Endpoints section of the CML UI. Run the following command to "describe" the model endpoint and obtain information about the payload it requires in order to interact with it.

```
curl -H "Content-Type: application/json" -H "Authorization: Bearer ${CDP_TOKEN}" "https://${DOMAIN}/namespaces/serving-default/endpoints/llama2-7b-chat-awq-from-hf/v2/models/model"
```

Finally, replace the model ID with the model ID obtained from the above response into the following command, and run it in order to make an inference call to the model.

```
curl -H "Content-Type: application/json" \
-H "Authorization: Bearer ${CDP_TOKEN}" \
"https://${DOMAIN}/namespaces/serving-default/endpoints/llama2-7b-chat-awq-from-hf/v1/completions" -d '{
  "prompt": "What should I do for a four day holiday trip in Spain?",
  "model": "<your-model-id-here>",
  "max_tokens": 200,
  "top_p": 1,
  "n": 1,
  "stream": false,
  "frequency_penalty": 0.0
}'
```


Create a


export DOMAIN=$(cdp ml describe-ml-serving-app --app-crn "crn:cdp:ml:us-west-1:558bc1d2-8867-4357-8524-311d51259233:mlserving:47680838-2766-4b3d-b85e-09e262e7bc46" | jq -r '.app.cluster.domainName')


curl -H "Content-Type: application/json" -H "Authorization: Bearer ${CDP_TOKEN}" "https://${DOMAIN}/api/v1alpha1/listEndpoints" -d '{
"namespace": "serving-default"
}'

curl -H "Content-Type: application/json" -H "Authorization: Bearer ${CDP_TOKEN}" "https://ml-ad159813-0c6.se-sandb.a465-9q4k.cloudera.site/namespaces/serving-default/endpoints/llama2-70b-chat-on-2-a10g/v1/models" | jq



#### Serving Cluster Deployment

#### Model Deployment

#### Inference Request

### Documentation and References

[CML Documentation](https://docs.cloudera.com/machine-learning/cloud/product/topics/ml-product-overview.html)

[CML Serving Release Notes](https://docs.cloudera.com/cdp-public-cloud-preview-features/cloud/ml-ai-inference-service/ml-ai-inference-service.pdf)
