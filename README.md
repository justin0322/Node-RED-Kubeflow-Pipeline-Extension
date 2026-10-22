# Node-RED Kubeflow Pipeline Extension

**A low-code AutoML interface: build, train, and deploy machine-learning models on Kubeflow by wiring nodes in Node-RED — no pipeline code required.**

![Node-RED](https://img.shields.io/badge/Node--RED-8F0000?logo=nodered&logoColor=white)
![Kubeflow](https://img.shields.io/badge/Kubeflow_Pipelines-1.8-4285F4?logo=kubeflow&logoColor=white)
![KServe](https://img.shields.io/badge/KServe-model_serving-1F6FEB)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?logo=javascript&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![MinIO](https://img.shields.io/badge/MinIO-C72E49?logo=minio&logoColor=white)

> University capstone project, developed by a team of three.

## Overview

Kubeflow is powerful but requires users to write Python, compile pipeline YAML, and manage Kubernetes resources.
This project wraps that workflow in **custom Node-RED nodes**, so a user can drag-and-drop a complete ML workflow:

```
[ datasets ] → [ data process ] → [ algorithm ] → [ model ] → [ deploy ]
   CSV / MinIO     fill NaN, encoding,   Random Forest /     compile & run on     serve on
                   scaling, reshape      Decision Tree /     Kubeflow Pipelines   KServe
                                         Logistic Regression
```

Each node contributes a code fragment from a template library. The `model` node assembles them into a
Kubeflow Pipelines (KFP) definition, compiles it, uploads it through the KFP REST API, starts a run, and
streams the status back into Node-RED. The `deploy` node then creates a KServe `InferenceService`
from the trained model stored on a PVC.

### Architecture

![architecture](https://hackmd.io/_uploads/H1ZLgUsH2.png)

### Demo

[![demo video](https://i.ytimg.com/vi/72tXYl6FcvU/hqdefault.jpg)](https://youtu.be/72tXYl6FcvU)

## Features

- **AutoAI pipeline nodes** (`autoAI pipeline` palette category)
  - `datasets` – built-in datasets (MNIST, CIFAR-10), or upload a custom CSV to MinIO and pick the label column
  - `data process` – missing-value filling (value / mean / median), label & one-hot encoding, normalization / standardization, reshaping
  - `algorithm` – Random Forest, Decision Tree, Logistic Regression with configurable hyper-parameters
  - `model` – generates, compiles and submits the KFP pipeline, then polls the run until it finishes
  - `deploy` – deploys the trained model to KServe for online inference
- **Pre-built pipeline nodes** (`nodepipe`) – one-click Random Forest / Decision Tree / Logistic Regression pipelines
- **KFP REST API examples** – create / list / delete experiments, pipelines and runs (`examples/main/py/api_examples`)
- **Containerized** – Node-RED and MinIO started together with Docker Compose; credentials passed via environment variables

## Project Structure

```
examples/
├── Dockerfile, docker-compose.yaml, build.sh, run.sh   # Node-RED + MinIO runtime
├── .env.example                                        # environment variable template
└── main/                                               # Node-RED user directory
    ├── flows.json                                      # sample flows
    ├── node_modules/
    │   ├── autoAI pipeline/                            # custom AutoAI nodes (html + js)
    │   │   └── snippets.js                             # Python/KFP code templates
    │   └── nodepipe/                                   # pre-built pipeline nodes
    └── py/                                             # KFP pipeline scripts, KServe deploy, API examples
```

> Custom nodes live in `main/node_modules/` because Node-RED loads local nodes from its user directory.

## Getting Started

### Prerequisites

- Docker & Docker Compose
- A reachable Kubeflow (1.x, Dex auth) instance with KServe installed

### Run

```bash
git clone https://github.com/justin0322/Node-RED-Kubeflow-Pipeline-Extension.git
cd Node-RED-Kubeflow-Pipeline-Extension/examples

cp .env.example .env      # fill in your Kubeflow / MinIO credentials
./build.sh                # build the Node-RED image
./run.sh main             # start Node-RED (user dir: main) and MinIO
```

Then open:

- Node-RED editor: http://127.0.0.1:1880
- MinIO console: http://127.0.0.1:9001

In the editor:

1. Click **install dependency** to install the Python libraries (`kfp`, `kfp-server-api`, `minio`, …) inside the container.
2. Select a flow and click the inject (timestamp) button to trigger the pipeline.
3. Watch the run status in the debug panel, or in the Kubeflow Pipelines dashboard.

![install dependency](https://hackmd.io/_uploads/SymtCs4Sn.png)

### Test the KFP API scripts only

```bash
docker exec -it <container-id> bash
cd /data/main/py/api_examples
python3 list_pipelines.py
```

## Extending: add your own algorithm

1. Add a model-creation template in `snippets.js` (between `START_MODEL_CODE` / `END_MODEL_CODE`).
2. Expose its hyper-parameters in `alg/algorithms.html` and map them in `alg/algorithms.js`.
3. Add an upload/run template and select it in `model/model.js`.

A step-by-step tutorial (Chinese) is in [`examples/README.md`](examples/README.md).

## Team

Built as a university capstone project by a three-person team, with all members contributing
to design and implementation across the Node-RED nodes, Kubeflow pipelines, and KServe deployment.
Originally developed in the team repository `NightLightTw/kubeflow-Node-RED`, which is no longer publicly available.

## References

- [kube-nodered](https://github.com/footprintai/kube-nodered) – base Node-RED ↔ Kubeflow integration
- [Kubeflow Pipelines Python HTTP client (1.8.21)](https://github.com/kubeflow/pipelines/tree/1.8.21/backend/api/python_http_client)
- [node-red-contrib-pythonshell](https://github.com/namgk/node-red-contrib-pythonshell)

## License

See [LICENSE](LICENSE).
