# CML Serving Demo



### A Brief Introduction to CML Serving

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
