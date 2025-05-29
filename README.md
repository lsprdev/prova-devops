# Passos que segui para configurar o ArgoCD

#### Passo 1: Criar cluster Kind como especificado.

```yaml
# kind-cluster.yaml
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

#### Passo 2: Criar o cluster Kind.

```bash
kind create cluster --config kind-cluster.yaml
```
#### Passo 3: Instalar o ArgoCD.

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
#### Passo 4: Expor o serviço do ArgoCD.

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

#### Passo 5: Obter a senha do ArgoCD.

```bash
kubectl -n argocd exec argocd-server-xxx -- argocd admin initial-password
```

#### Passo 6: Login no ArgoCD.

# Passos que segui para criar o projeto python com Github Workflow e k8s

#### Passo 1: Criar /app com os seguintes arquivos:

```python
# app/main.py
from fastapi import FastAPI
app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello BSI!"}

@app.get("/square/{x}")
def square(x: int):
    return {"result": x**2}

@app.get("/double/{x}")
def double(x: int):
    return {"result": x * 2}
```

```python
# app/test_main.py
from fastapi import FastAPI
app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello BSI!"}

@app.get("/square/{x}")
def square(x: int):
    return {"result": x**2}

@app.get("/double/{x}")
def double(x: int):
    return {"result": x * 2}
```

#### Passo 2: Criei e entrei em um ambiente virtual.

```bash
python3 -m venv venv
source venv/bin/activate
```
#### Passo 3: Instalar as dependências.

```bash
pip install fastapi uvicorn pytest pytest-asyncio
```
#### Passo 4: Criar o arquivo requirements.txt.

```bash
python -m pip freeze > /app/requirements.txt
```

#### Passo 5: Rodei e arrumei o teste com falha.

#### Passo 6: Crei o Dockerfile.

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY app/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app/ .

EXPOSE 9000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

#### Passo 7: Criei o workflow do Github e k8s.

#### Passo 8: Fiz push de tudo para meu repositório no [Github](https://github.com/lsprdev/prova-devops)

# Passos que segui para fazer deploy no ArgoCD

#### Passo 1: Clicar em "New App" no ArgoCD.
#### Passo 2: Preencher os campos do formulário.
- **Application Name**: `prova-devops`
- **Project**: `default`
- **Sync Policy**: `Automatic`
- **Repository URL**: `https://github.com/lsprdev/prova-devops.git`
- **Revision**: `HEAD`
- **Path**: `k8s
- **Cluster**: `https://kubernetes.default.svc`
- **Namespace**: `default`

#### Passo 3: Clicar em "Create" para criar o aplicativo.
#### Passo 4: Clicar em "Sync" para sincronizar o aplicativo.
#### Passo 5: Verificar o status do aplicativo.
#### Passo 6: Acessar o serviço do aplicativo.

```bash
kubectl port-forward pod/app-xxx --address 0.0.0.0 9000:8000
```

#### Passo 7: Acessar o aplicativo no navegador.

```plaintext
http://localhost:9000
```