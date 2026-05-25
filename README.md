# Federated Learning with Secure Aggregation & Blockchain Logging

A federated learning project that demonstrates **Non-IID data simulation**, **Homomorphic Encryption (HE)** for secure aggregation, and **blockchain-based audit logging** of model updates — built using the **Fluke** FL framework.

> **Note:** This project does not use a practical client-server network architecture. Instead, the [Fluke](https://github.com/makgyver/fluke) library is used to **simulate** the client-server federated learning setup. Fluke internally manages server broadcasts, client local training, and model aggregation through its `Server`, `Client`, and `Channel` APIs — all running within a single process.

---

## Project Structure

```
├── NonIID_Simulation.ipynb    # Notebook A: IID vs Non-IID comparison
├── Secure_Aggregation.ipynb   # Notebook B: HE encryption + blockchain logging
├── CNN.py                     # Custom model definitions (MNIST_CNN, HeartModel, DiabetesModel)
└── README.md
```

---

## Notebooks

### `NonIID_Simulation.ipynb` — IID vs Non-IID Data Distribution

Compares federated learning performance under two data distribution strategies:

- **IID** — data is uniformly distributed across clients
- **Non-IID** — data is distributed using Dirichlet-based label skew (`label_dirichlet_skew`), where each client gets a biased subset of classes



**Result:** IID achieves ~97% accuracy; Non-IID achieves ~95% accuracy, demonstrating the challenge of heterogeneous data in FL.

---

### `Secure_Aggregation.ipynb` — HE Encryption + Blockchain Audit Trail

Builds on the Non-IID simulation by adding **privacy** and **auditability** layers:

**Homomorphic Encryption (CKKS via TenSEAL):**
- Client model weights are flattened, chunked, and encrypted using CKKS scheme
- Server aggregates encrypted weights (sum + scale) without seeing raw values
- Decrypted average is loaded back into the global model

**Blockchain Logging:**
- A lightweight `ModelChain` class logs every FL round as a block
- Each block stores: round number, accuracy, SHA-256 hash of global model, hashes of each client model, and the previous block's hash
- Chain integrity can be verified — tampering with any block breaks the hash chain



**Result:** Achieves ~94% accuracy with HE — encryption does not degrade model performance. ``Main overhead`` is in client→server direction Server to Client download is exceptionally fast (~0.0021s) because it transmits raw plaintext, whereas the Client to Server upload acts as the primary system bottleneck (~0.7611s) due to the heavy computing time required to flatten, chunk, and transform model parameters into encrypted CKKS ciphertexts



## Fluke Simulates Client-Server FL

Fluke is **not** a networking library — it does **not** send models over TCP/IP between separate machines. Instead:

- `FedAVG` creates a `Server` object and `N` `Client` objects in the same process
- `server.broadcast_model(clients)` copies the global model into each client's memory via Fluke's internal `Channel`
- Each `client.local_update()` trains on its local data partition
- `server.receive_client_models()` reads trained models from the channel
- `server.aggregate()` averages the models (FedAvg)


The communication overhead measured (Server→Client and Client→Server times) reflects the **in-memory simulation time**, not actual network latency.

---

## Datasets

The following datasets were used in this project (links provided for reference):

- **MNIST** – Handwritten digit recognition  
  [https://www.kaggle.com/datasets/hojjatk/mnist-dataset](https://www.kaggle.com/datasets/hojjatk/mnist-dataset)

- **Diabetes Dataset** – Medical data for diabetes prediction  
  [https://www.kaggle.com/datasets/mathchi/diabetes-data-set](https://www.kaggle.com/datasets/mathchi/diabetes-data-set)

- **Heart Disease Dataset** – Cardiovascular data for heart disease prediction  
  [https://www.kaggle.com/datasets/johnsmith88/heart-disease-dataset](https://www.kaggle.com/datasets/johnsmith88/heart-disease-dataset)

> The MNIST dataset is automatically downloaded by Fluke; the diabetes and heart disease datasets were loaded manually via `CSVDataset` for extension experiments.
----
