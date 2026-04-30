\# exp-12

\# Experiment 12: Study and Analyse Container Orchestration using Kubernetes



\*\*Name:\*\* Akshat Gupta

\*\*Course:\*\* Containerization and DevOps  



\---



\## Objective



Learn why Kubernetes is used, its basic concepts, and how to deploy, scale, and fix apps using Kubernetes commands.



\---



\## Why Kubernetes over Docker Swarm?



| Reason | Explanation |

|--------|-------------|

| Industry standard | Most companies use Kubernetes |

| Powerful scheduling | Automatically decides where to run your app |

| Large ecosystem | Many tools and plugins available |

| Cloud-native support | Works on AWS, Google Cloud, Azure, etc. |



\---



\## Core Kubernetes Concepts (Simple Explanation)



| Docker Concept | Kubernetes Equivalent | What it means |

|---------------|----------------------|----------------|

| Container | Pod | A pod is a group of one or more containers. Smallest unit in K8s. |

| Compose service | Deployment | Describes how your app should run (e.g., 2 copies, which image to use) |

| Load balancing | Service | Exposes your app to the outside world or other pods |

| Scaling | ReplicaSet | Ensures a certain number of pod copies are always running |



\---



\## Hands-On Lab (Using k3d or Minikube)



> \*\*Note:\*\* We assume you already have `kubectl` and a cluster (k3d or Minikube) installed.



\---



\## PART A – Core Tasks



\### Task 1: Create a Deployment



A deployment tells Kubernetes:

\- Which container image to use (e.g., WordPress)

\- How many copies (replicas) to run

\- How to identify the pods (labels)



\*\*Step 1:\*\* Create the file `wordpress-deployment.yaml`



```yaml

\# wordpress-deployment.yaml

apiVersion: apps/v1          # Which Kubernetes API to use

kind: Deployment             # Type of resource

metadata:

&#x20; name: wordpress            # Name of this deployment

spec:

&#x20; replicas: 2                # Run 2 identical pods

&#x20; selector:

&#x20;   matchLabels:

&#x20;     app: wordpress         # Pods with this label belong to this deployment

&#x20; template:                  # Template for the pods

&#x20;   metadata:

&#x20;     labels:

&#x20;       app: wordpress       # Label applied to each pod

&#x20;   spec:

&#x20;     containers:

&#x20;     - name: wordpress

&#x20;       image: wordpress:latest   # Docker image

&#x20;       ports:

&#x20;       - containerPort: 80       # Port inside the container

```

<img width="1280" height="835" alt="image" src="https://github.com/user-attachments/assets/ebcb95b6-3a5b-4ee7-b04d-13ee641ea5d8" />



\*\*Step 2:\*\* Apply the deployment



```bash

kubectl apply -f wordpress-deployment.yaml

```



> \*\*What happens?\*\* Kubernetes creates 2 pods running WordPress.



<img width="984" height="123" alt="image" src="https://github.com/user-attachments/assets/8dbe7917-2083-455c-929a-7b26f2adc6d5" />



\---



\### Task 2: Expose the Deployment as a Service



Pods are temporary (they can be deleted or recreated). A Service gives them a fixed IP and exposes them to the outside.



\*\*Step 1:\*\* Create the file `wordpress-service.yaml`



```yaml

\# wordpress-service.yaml

apiVersion: v1

kind: Service

metadata:

&#x20; name: wordpress-service

spec:

&#x20; type: NodePort            # Exposes service on a port of each node (VM)

&#x20; selector:

&#x20;   app: wordpress          # Send traffic to pods with this label

&#x20; ports:

&#x20;   - port: 80              # Service port

&#x20;     targetPort: 80        # Pod port

&#x20;     nodePort: 30007       # External port (range: 30000-32767)

```

<img width="630" height="692" alt="image" src="https://github.com/user-attachments/assets/4c146b90-5ec6-4ae4-a39a-ec9f587d5b66" />



\*\*Step 2:\*\* Apply the service



```bash

kubectl apply -f wordpress-service.yaml

```







\---



\### Task 3: Verify Everything



\*\*Check if pods are running:\*\*



```bash

kubectl get pods

```



<img width="983" height="88" alt="image" src="https://github.com/user-attachments/assets/15b97893-2d08-49fe-8be5-944fbc547dab" />



\*\*Check the service:\*\*



```bash

kubectl get svc

```

<img width="980" height="175" alt="image" src="https://github.com/user-attachments/assets/399a3532-f4ea-44bc-8043-f547c6bda377" />





\*\*Access WordPress in your browser:\*\*



```

http://localhost:30007

```



\- \*\*Minikube:\*\* Usually `minikube ip`

\- \*\*k3d:\*\* Usually `localhost`

<img width="946" height="209" alt="image" src="https://github.com/user-attachments/assets/c989ec84-b4bd-4ece-baef-c42379d43693" />





<img width="710" height="484" alt="image" src="https://github.com/user-attachments/assets/ab62d14d-3091-40c3-ab45-e72f27b9e5c1" />





\---



\### Task 4: Scale the Deployment



Increase the number of pods from 2 to 4:



```bash

kubectl scale deployment wordpress --replicas=4

```

<img width="966" height="174" alt="image" src="https://github.com/user-attachments/assets/efcc3029-c197-4fe6-922f-d59dd201d1d4" />



Verify:



```bash

kubectl get pods

```

<img width="922" height="153" alt="image" src="https://github.com/user-attachments/assets/ddf14e5d-4c99-4952-a428-2f6dfbc56693" />





> You should now see 4 running pods.



> \*\*Why scale?\*\* More traffic → more copies → better performance.







\---



\### Task 5: Self-Healing Demonstration



Kubernetes automatically replaces failed pods.



\*\*Step 1:\*\* Get pod names



```bash

kubectl get pods

```



\*\*Step 2:\*\* Delete one pod (replace `<pod-name>`):



```bash

kubectl delete pod <pod-name>

```



\*\*Step 3:\*\* Check pods again:



```bash

kubectl get pods

```



> You will still see 4 pods — the deleted one was \*\*automatically recreated\*\*.



> \*\*Why?\*\* The deployment ensures the desired number (4) is always running.



<img width="975" height="422" alt="image" src="https://github.com/user-attachments/assets/4480d47a-21ee-47f5-a65d-a350b67ec8a4" />



\---



\## PART B – Cluster Information



```bash

kubectl get nodes

kubectl cluster-info

```



<img width="983" height="215" alt="image" src="https://github.com/user-attachments/assets/0bc84fc4-ee14-4bfd-bcdd-f5b24c54cbfe" />





\---



\## PART C – Swarm vs Kubernetes (Comparison)



| Feature | Docker Swarm | Kubernetes |

|---------|-------------|------------|

| Setup | Very easy | More complex |

| Scaling | Basic | Advanced (auto-scaling) |

| Ecosystem | Small | Huge (monitoring, logging) |

| Industry use | Rare | Standard |



> \*\*Verdict:\*\* Learn Kubernetes — it's what companies use.



\---



\## PART D – Advanced Lab: Real Cluster with kubeadm



\### Lab Requirements

\- 2 or 3 virtual machines (e.g., VirtualBox, VMware)

\- Ubuntu 22.04 or 24.04

\- Each VM: 2+ CPU, 2+ GB RAM



\### High-Level Steps



\*\*Step 1:\*\* Install `kubeadm`, `kubelet`, `kubectl` on all nodes



```bash

\# Update system

sudo apt update



\# Install required packages

sudo apt install -y apt-transport-https ca-certificates curl



\# Add Kubernetes signing key

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | \\

&#x20; sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg



\# Add Kubernetes repository

echo 'deb \[signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \\

&#x20; https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | \\

&#x20; sudo tee /etc/apt/sources.list.d/kubernetes.list



\# Install kubeadm, kubelet, kubectl

sudo apt update

sudo apt install -y kubeadm kubelet kubectl



\# Hold versions to prevent auto-update

sudo apt-mark hold kubeadm kubelet kubectl

```



<img width="986" height="400" alt="image" src="https://github.com/user-attachments/assets/37c4f596-ba8a-4827-887a-f2b693decaea" />





\*\*Step 2:\*\* Initialize the control plane (master node only)



```bash

sudo kubeadm init

```



This command:

\- Sets up the control plane

\- Generates a token for worker nodes to join

\- Takes 2–3 minutes



\*\*Step 3:\*\* Set up kubeconfig (to use `kubectl`)



```bash

\# Create .kube directory for your user

mkdir -p $HOME/.kube



\# Copy admin config

sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config



\# Fix permissions

sudo chown $(id -u):$(id -g) $HOME/.kube/config

```



<img width="983" height="86" alt="image" src="https://github.com/user-attachments/assets/fb192ecb-3432-4720-9dd7-587f49c5586a" />



Now you can run `kubectl get nodes` to see the master.



\*\*Step 4:\*\* Install a network plugin (required for pods to communicate)



```bash

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

```



<img width="984" height="499" alt="image" src="https://github.com/user-attachments/assets/24a4214e-a429-4daa-81f7-d75eb794f9c9" />





Wait 1–2 minutes for Calico to start.



\*\*Step 5:\*\* Join worker nodes



After `kubeadm init`, you'll see a join command like:



```bash

kubeadm join 192.168.1.100:6443 --token abcdef.0123456789abcdef \\

&#x20;   --discovery-token-ca-cert-hash sha256:...

```



Run that exact command on each worker node.



\*\*Step 6:\*\* Verify the cluster (on master)



```bash

kubectl get nodes

```



<img width="976" height="111" alt="image" src="https://github.com/user-attachments/assets/3639303e-4226-45cb-aefd-c7bece0fdb7c" />





Expected output:

```

NAME       STATUS   ROLES           AGE   VERSION

master     Ready    control-plane   5m    v1.29.0

worker1    Ready    <none>          2m    v1.29.0

worker2    Ready    <none>          2m    v1.29.0

```



\---



\## Summary of Commands (Cheat Sheet)



| Goal | Command |

|------|---------|

| Apply a YAML file | `kubectl apply -f file.yaml` |

| See all pods | `kubectl get pods` |

| See all services | `kubectl get svc` |

| Scale a deployment | `kubectl scale deployment <name> --replicas=N` |

| Delete a pod | `kubectl delete pod <pod-name>` |

| See all nodes | `kubectl get nodes` |

| Cluster info | `kubectl cluster-info` |



\---



\## Conclusion



In this experiment, we:



\- Understood basic Kubernetes concepts (Pods, Deployments, Services, ReplicaSets)

\- Deployed WordPress using a \*\*Deployment\*\* and \*\*Service\*\*

\- \*\*Scaled\*\* the deployment from 2 to 4 replicas

\- Demonstrated \*\*self-healing\*\* by deleting a pod and watching it auto-recover

\- Compared \*\*Docker Swarm\*\* vs \*\*Kubernetes\*\*

\- Learned how to build a real production-style cluster with `kubeadm`



\*\*Next step:\*\* Try deploying your own custom app (e.g., a simple Node.js or Python app).



