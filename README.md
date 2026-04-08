# Setup kserve with minikube

Make sure to have the container toolkit and all other docker system dependencies configured first. Also install kubernetes tools istioctl, minikube 

* minikube start --driver docker --container-runtime docker --gpus all
* curl -fsSL https://github.com/kserve/kserve/releases/download/v0.17.0/kserve-knative-mode-dependency-install.sh | bash
* curl -fsSL https://github.com/kserve/kserve/releases/download/v0.17.0/llmisvc-dependency-install.sh | bash
* kubectl apply --server-side -f https://github.com/kserve/kserve/releases/download/v0.17.0/kserve-crds.yaml
* kubectl apply --server-side -f https://github.com/kserve/kserve/releases/download/v0.17.0/kserve.yaml
* kubectl apply --server-side -f https://github.com/kserve/kserve/releases/download/v0.17.0/kserve-cluster-resources.yaml

* kubectl create namespace kserve-test
* kubectl apply -f kserve-huggingfaceserver-dotjson.yaml 
* kubectl apply -n kserve-test -f inference_service_dotjson.yaml


## Modifications

To get things to run with vllm v0.18.0 and dotjson the file kserve/python/vllm_model.py was modified. It is necessary to build a new huggingface-server docker image with these changes.

* clone kserve
* copy over updated vllm_model.py
* Set codeartifact token
  ```
  export CODEARTIFACT_AUTH_TOKEN=`aws codeartifact get-authorization-token --domain dottxt-ai --domain-owner 242201295967 --query authorizationToken --output text --region us-east-1`
  ```
* Build docker image (note the tag matches that in the serving runtime resource definition) 
  ```
  docker build -t huggingfaceserver:vllm-dev . -f huggingface_server.Dockerfile --secret id=catoken,env=CODEARTIFACT_AUTH_TOKEN
  ```

## Example 

```
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')

export SERVICE_HOSTNAME=$(kubectl get inferenceservice qwen-llm-dotjson -n kserve-test -o jsonpath='{.status.url}' | cut -d "/" -f 3) 

curl -v -H "Host: ${SERVICE_HOSTNAME}" -H "Content-Type: application/json" "http://${INGRESS_HOST}:${INGRESS_PORT}/openai/v1/chat/completions" -d @./schema-input-not.json

*   Trying 192.168.49.200:80...
* Established connection to 192.168.49.200 (192.168.49.200 port 80) from 192.168.49.1 port 51332 
* using HTTP/1.x
> POST /openai/v1/chat/completions HTTP/1.1
> Host: qwen-llm-dotjson.kserve-test.example.com
> User-Agent: curl/8.19.0
> Accept: */*
> Content-Type: application/json
> Content-Length: 389
> 
* upload completely sent off: 389 bytes
< HTTP/1.1 200 OK
< content-length: 579
< content-type: application/json
< date: Wed, 08 Apr 2026 07:01:05 GMT
< server: istio-envoy
< x-envoy-upstream-service-time: 889
< 
* Connection #0 to host 192.168.49.200:80 left intact
{"id":"chatcmpl-cd3988e9-36cc-4c74-98b6-26820f7f15a5","object":"chat.completion","created":1775631666,"model":"qwen","choices":[{"index":0,"message":{"role":"assistant","content":"4","refusal":null,"annotations":null,"audio":null,"function_call":null,"tool_calls":[],"reasoning":null},"logprobs":null,"finish_reason":"stop","stop_reason":null,"token_ids":null}],"service_tier":null,"system_fingerprint":null,"usage":{"prompt_tokens":10,"total_tokens":12,"completion_tokens":2,"prompt_tokens_details":null},"prompt_logprobs":null,"prompt_token_ids":null,"kv_transfer_params":null}% 
```
