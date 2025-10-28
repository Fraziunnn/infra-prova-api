## 🎯 Objetivos da Atividade

Este projeto tem como objetivo demonstrar o ciclo completo de desenvolvimento, conteinerização e deploy de uma aplicação simples utilizando práticas modernas de **Infraestrutura como Código (IaC)**.

O fluxo principal contempla:

1. Criação de uma **API Flask** simples.  
2. **Dockerização** da aplicação.  
3. Deploy local em **Kubernetes (Minikube ou Kind)**.  
4. Provisionamento de recursos AWS (ou simulação com **LocalStack**) via **Terraform**.  
5. Testes de integração utilizando **Postman** ou **cURL**.  

---

## 🗂 Estrutura do Repositório


---

## 🚀 1. API Flask

### Arquivo: `api/app.py`

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route("/")
def index():
    return jsonify({"message": "API de teste - Infraestrutura", "status": "ok"})

@app.route("/sum", methods=["POST"])
def soma():
    data = request.get_json() or {}
    a = data.get("a", 0)
    b = data.get("b", 0)
    try:
        s = float(a) + float(b)
        return jsonify({"a": a, "b": b, "sum": s})
    except Exception as e:
        return jsonify({"error": str(e)}), 400

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

Dependências (requirements.txt)

Flask==2.2.5

Dockerfile

FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]

🐳 2. Docker — Build e Teste Local
cd api
# construir imagem
docker build -t infra-prova-api:latest .

# executar localmente
docker run --rm -p 5000:5000 infra-prova-api:latest

Teste da API

# Teste do endpoint raiz
curl http://localhost:5000/

# Teste do endpoint /sum
curl -X POST http://localhost:5000/sum \
  -H "Content-Type: application/json" \
  -d '{"a": 3, "b": 4.5}'

☸️ 3. Deploy no Kubernetes

Arquivo: k8s/deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: infra-prova-api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: infra-prova-api
  template:
    metadata:
      labels:
        app: infra-prova-api
    spec:
      containers:
        - name: infra-prova-api
          image: infra-prova-api:latest
          ports:
            - containerPort: 5000

Arquivo: k8s/service.yaml

apiVersion: v1
kind: Service
metadata:
  name: infra-prova-api-svc
spec:
  selector:
    app: infra-prova-api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: NodePort

Comandos

kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

# descobrir porta e acessar
kubectl get svc infra-prova-api-svc

# se estiver usando minikube
minikube service infra-prova-api-svc --url

# ou
kubectl port-forward deployment/infra-prova-api 5000:5000

🧪 5. Testes de Integração

Use Postman, Insomnia ou cURL para testar os endpoints:

GET / → retorna mensagem e status “ok”

POST /sum → recebe JSON com a e b e retorna a soma

{
  "a": 3,
  "b": 4.5,
  "sum": 7.5
}

