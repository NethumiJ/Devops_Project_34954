# DevOps Practical

**Name:** J K N S Jayakody
**Index No:** 34954

A Flask app taken through the full DevOps pipeline: source control → CI (GitHub Actions) → Docker → Kubernetes (Minikube) → continuous deployment (Render).

## Project structure

```
devops-practical/
├── app.py                     # Flask app (reads PORT env var, defaults to 5000)
├── test_app.py                # PyTest quality gate
├── requirements.txt           # flask==3.0.0, pytest==7.4.0
├── Dockerfile                 # python:3.9-slim image
├── .github/workflows/ci.yml   # CI pipeline: install deps + run pytest on push to main
└── k8s/
    ├── deployment.yaml        # 2 replicas of the flask container
    └── service.yaml           # NodePort service, port 80 -> 5000
```

## Phase 1 – Local dev & source control
```bash
pip install -r requirements.txt
pytest
python app.py          # http://localhost:5000
```

## Phase 2 – CI
Every push to `main` runs `.github/workflows/ci.yml` (checkout → Python 3.9 → install → pytest). See the **Actions** tab.

## Phase 3 – Docker
```bash
docker build -t flask-app:v1 .
docker run -p 5000:5000 flask-app:v1
docker login
docker tag flask-app:v1 <DOCKERHUB-USERNAME>/flask-app:v1
docker push <DOCKERHUB-USERNAME>/flask-app:v1
```

## Phase 4 – Kubernetes (Minikube)
Set `image:` in `k8s/deployment.yaml` to your Docker Hub image, then:
```bash
minikube start --driver=docker
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get pods -w
minikube service flask-service
```

## Phase 5 – Chaos test (self-healing)
```bash
kubectl get pods
kubectl delete pod <POD-NAME>
kubectl get pods       # a replacement pod is created to keep 2 replicas
```

## Phase 6 – Continuous deployment (Render)
1. Render → **New + → Web Service** → connect this repo.
2. Branch `main`, Runtime **Docker**, Instance type **Free** → **Create Web Service**.
3. Render injects `PORT`; `app.py` reads it.
4. Change the message in `app.py`, `git push`, and the live URL updates automatically.

Current live message: `CI/CD and Docker Pipeline is a massive success! V2 - J K N S Jayakody (34954)`
