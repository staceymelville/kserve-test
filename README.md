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
