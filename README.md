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
