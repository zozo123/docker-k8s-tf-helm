# Zero-to-Hero Course: Docker, Kubernetes, Minikube, Terraform, and Helm

This all-encompassing guide is crafted to transform you from a novice into a skilled professional in Docker, Kubernetes (using Minikube), Terraform, and Helm. The instructions are tailored for Mac users with **Homebrew**, **Docker Desktop**, and **Terminal** already set up.

---

## Module 1: Docker Fundamentals

### Learning Objectives
- Grasp the basics of containerization and Docker.
- Create, execute, and manage Docker containers and images.
- Implement Docker volumes and networking.

### Course Content

#### 1. Understanding Containers
- **Key Concepts:**
  - **Containers vs. Virtual Machines (VMs):**
    - **Containers** are lightweight, share the host operating system, and are perfect for microservices.
    - **VMs** are heavier, include a complete OS, and are ideal for isolating entire environments.
  - **Advantages of Containerization:**
    - Portability, scalability, uniformity across different environments, and efficient resource utilization.

#### 2. Verifying Docker Installation (Pre-installed via Docker Desktop)
- **Check Docker Version:**
  ```bash
  docker --version
  ```
  *Sample Output:*
  ```
  Docker version 20.10.7, build f0df350
  ```

#### 3. Creating Docker Images

- **Developing a Simple Node.js Application**

  - **Directory Structure:**
    ```
    my-node-app/
    ├── Dockerfile
    ├── package.json
    └── app.js
    ```

  - **app.js:**
    ```javascript
    const express = require('express');
    const app = express();
    const port = 3000;

    app.get('/', (req, res) => {
      res.send('Hello, Docker!');
    });

    app.listen(port, () => {
      console.log(`App listening at http://localhost:${port}`);
    });
    ```

  - **package.json:**
    ```json
    {
      "name": "my-node-app",
      "version": "1.0.0",
      "main": "app.js",
      "scripts": {
        "start": "node app.js"
      },
      "dependencies": {
        "express": "^4.17.1"
      }
    }
    ```

  - **Dockerfile:**
    ```dockerfile
    FROM node:14

    # Set working directory
    WORKDIR /usr/src/app

    # Install dependencies
    COPY package*.json ./
    RUN npm install

    # Copy source code
    COPY . .

    EXPOSE 3000
    CMD ["npm", "start"]
    ```

- **Building the Docker Image:**
  ```bash
  cd my-node-app
  docker build -t my-node-app:1.0 .
  ```
  *Sample Output:*
  ```
  Successfully built <image_id>
  Successfully tagged my-node-app:1.0
  ```

#### 4. Running Docker Containers

- **Start the Container:**
  ```bash
  docker run -d -p 3000:3000 --name my-running-app my-node-app:1.0
  ```
  *Explanation of Flags:*
  - `-d`: Run container in detached mode.
  - `-p 3000:3000`: Map host port 3000 to container port 3000.
  - `--name`: Assign a specific name to the container.

- **Confirm Container is Active:**
  ```bash
  docker ps
  ```
  *Sample Output:*
  ```
  CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS                    NAMES
  <container_id> my-node-app:1.0 "docker-entrypoint.s…"   10 seconds ago   Up 9 seconds    0.0.0.0:3000->3000/tcp   my-running-app
  ```

- **Access the Application:**
  - Open `http://localhost:3000` in your web browser to view "Hello, Docker!"

#### 5. Managing Docker Volumes

- **Create a New Volume:**
  ```bash
  docker volume create my-data
  ```
  *Sample Output:*
  ```
  my-data
  ```

- **Run Container with Mounted Volume:**
  ```bash
  docker run -d -p 3000:3000 --name my-running-app -v my-data:/usr/src/app/data my-node-app:1.0
  ```

- **Inspect Volume Mount:**
  ```bash
  docker inspect my-running-app
  ```
  - Check the `Mounts` section to ensure the volume is connected.

#### 6. Docker Networking

- **Establish a Custom Network:**
  ```bash
  docker network create my-network
  ```
  *Sample Output:*
  ```
  123abc456def789ghi
  ```

- **Launch Containers on the Network:**
  ```bash
  docker run -d --name app1 --network my-network my-node-app:1.0
  docker run -d --name app2 --network my-network my-node-app:1.0
  ```

- **Inspect the Network:**
  ```bash
  docker network inspect my-network
  ```
  - Ensure `app1` and `app2` are connected.

- **Enable Communication Between Containers:**
  - **Access `app1` from `app2`:**
    ```bash
    docker exec -it app2 sh
    ```
    Inside the container:
    ```bash
    curl http://app1:3000
    ```
    *Expected Output:*
    ```
    Hello, Docker!
    ```

### Hands-On Exercises

1. **Exercise 1:** Build and launch a Docker container for the provided Node.js application.
   - **Steps:**
     ```bash
     cd my-node-app
     docker build -t my-node-app:1.0 .
     docker run -d -p 3000:3000 --name my-running-app my-node-app:1.0
     ```
     - Visit `http://localhost:3000` to verify.

2. **Exercise 2:** Use Docker volumes to retain application data.
   - **Steps:**
     ```bash
     docker volume create my-data
     docker run -d -p 3000:3000 --name my-running-app -v my-data:/usr/src/app/data my-node-app:1.0
     ```

3. **Exercise 3:** Link two containers via a custom Docker network and enable inter-container communication.
   - **Steps:**
     ```bash
     docker network create my-network
     docker run -d --name app1 --network my-network my-node-app:1.0
     docker run -d --name app2 --network my-network my-node-app:1.0
     docker exec -it app2 sh -c "curl http://app1:3000"
     ```
     *Expected Output:*
     ```
     Hello, Docker!
     ```

---

## Module 2: Kubernetes Foundations with Minikube

### Learning Objectives
- Comprehend Kubernetes architecture and its fundamental components.
- Deploy, scale, and manage applications within a Kubernetes cluster using Minikube.

### Course Content

#### 1. Installing Minikube and kubectl

- **Install kubectl:**
  ```bash
  brew install kubectl
  ```
  *Verify Installation:*
  ```bash
  kubectl version --client
  ```
  *Sample Output:*
  ```
  Client Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.0", ...}
  ```

- **Install Minikube:**
  ```bash
  brew install minikube
  ```

- **Start Minikube Cluster:**
  ```bash
  minikube start
  ```
  - Wait for the cluster to initialize.

- **Check Minikube Status:**
  ```bash
  minikube status
  ```
  *Sample Output:*
  ```
  host: Running
  kubelet: Running
  apiserver: Running
  kubeconfig: Configured
  ```

#### 2. Core Kubernetes Concepts

- **Key Components:**
  - **Cluster:** A collection of nodes running Kubernetes.
  - **Node:** An individual machine within the cluster.
  - **Pod:** The smallest deployable unit, containing one or more containers.
  - **Deployment:** Manages Pods, ensuring the desired number of replicas.
  - **Service:** Exposes Pods to external network traffic.

#### 3. Deploying Your Initial Application

- **Build Docker Image for Minikube:**
  ```bash
  eval $(minikube docker-env)
  docker build -t my-node-app:1.0 ./my-node-app
  ```
  - **Reset Docker Environment:**
    ```bash
    eval $(minikube docker-env -u)
    ```

- **Create a Deployment:**
  ```bash
  kubectl create deployment my-node-app --image=my-node-app:1.0
  ```
  *Sample Output:*
  ```
  deployment.apps/my-node-app created
  ```

- **Expose the Deployment:**
  ```bash
  kubectl expose deployment my-node-app --type=NodePort --port=3000
  ```
  *Sample Output:*
  ```
  service/my-node-app exposed
  ```

- **Access the Application:**
  ```bash
  minikube service my-node-app
  ```
  - This command will open the service URL in your default browser.

#### 4. Scaling Deployments

- **Increase Replicas to 3:**
  ```bash
  kubectl scale deployment my-node-app --replicas=3
  ```

- **Confirm Scaling:**
  ```bash
  kubectl get deployments
  kubectl get pods
  ```
  *Sample Output:*
  ```
  NAME          READY   UP-TO-DATE   AVAILABLE   AGE
  my-node-app   3/3     3            3           2m

  NAME                                READY   STATUS    RESTARTS   AGE
  my-node-app-6b8b75c9b7-abcde        1/1     Running   0          2m
  my-node-app-6b8b75c9b7-fghij        1/1     Running   0          2m
  my-node-app-6b8b75c9b7-klmno        1/1     Running   0          2m
  ```

#### 5. Rolling Updates and Rollbacks

- **Modify the Application:**
  - Change the message in `app.js` to:
    ```javascript
    res.send('Hello, Kubernetes!');
    ```

- **Rebuild the Docker Image:**
  ```bash
  eval $(minikube docker-env)
  docker build -t my-node-app:2.0 ./my-node-app
  eval $(minikube docker-env -u)
  ```

- **Update Deployment with New Image:**
  ```bash
  kubectl set image deployment/my-node-app my-node-app=my-node-app:2.0
  ```

- **Monitor the Update:**
  ```bash
  kubectl rollout status deployment/my-node-app
  ```
  *Sample Output:*
  ```
  deployment "my-node-app" successfully rolled out
  ```

- **Rollback to Previous Version if Necessary:**
  ```bash
  kubectl rollout undo deployment/my-node-app
  ```
  *Sample Output:*
  ```
  deployment.apps/my-node-app rolled back
  ```

#### 6. Utilizing the Kubernetes Dashboard

- **Enable the Dashboard Addon:**
  ```bash
  minikube addons enable dashboard
  ```

- **Launch the Dashboard:**
  ```bash
  minikube dashboard
  ```
  - This command will open the Kubernetes Dashboard in your default browser.

### Hands-On Exercises

1. **Exercise 1:** Deploy the Node.js application to Kubernetes using Minikube.
   - **Steps:**
     ```bash
     eval $(minikube docker-env)
     docker build -t my-node-app:1.0 ./my-node-app
     eval $(minikube docker-env -u)
     kubectl create deployment my-node-app --image=my-node-app:1.0
     kubectl expose deployment my-node-app --type=NodePort --port=3000
     minikube service my-node-app
     ```

2. **Exercise 2:** Scale the deployment and observe the changes.
   - **Steps:**
     ```bash
     kubectl scale deployment my-node-app --replicas=5
     kubectl get deployments
     kubectl get pods
     ```

3. **Exercise 3:** Perform a rolling update and rollback the deployment.
   - **Steps:**
     - Update `app.js` with a new message.
     - Rebuild the Docker image:
       ```bash
       eval $(minikube docker-env)
       docker build -t my-node-app:2.0 ./my-node-app
       eval $(minikube docker-env -u)
       ```
     - Update the deployment:
       ```bash
       kubectl set image deployment/my-node-app my-node-app=my-node-app:2.0
       kubectl rollout status deployment/my-node-app
       ```
     - If needed, rollback:
       ```bash
       kubectl rollout undo deployment/my-node-app
       ```

4. **Exercise 4:** Access and explore the Kubernetes Dashboard.
   - **Steps:**
     ```bash
     minikube dashboard
     ```
     - Navigate through Deployments, Pods, Services, and other resources via the Dashboard interface.

---

## Module 3: Terraform Fundamentals

### Learning Objectives
- Comprehend Infrastructure as Code (IaC) concepts.
- Develop and manage Terraform configurations.
- Deploy basic cloud infrastructure using Terraform.

### Course Content

#### 1. Installing Terraform

- **Install Terraform via Homebrew:**
  ```bash
  brew tap hashicorp/tap
  brew install hashicorp/tap/terraform
  ```
  *Verify Installation:*
  ```bash
  terraform version
  ```
  *Sample Output:*
  ```
  Terraform v1.0.11
  ```

#### 2. Core Terraform Concepts

- **Providers:** Interfaces to interact with cloud services (e.g., AWS, Azure, GCP).
- **Resources:** Entities managed by Terraform (e.g., virtual machines, networks).
- **State:** Maintains the current state of the infrastructure.

#### 3. Configuring Cloud Provider Credentials

- **Using AWS as an Example:**
  - **Install AWS CLI:**
    ```bash
    brew install awscli
    ```
  - **Set Up AWS Credentials:**
    ```bash
    aws configure
    ```
    - Input your AWS Access Key, Secret Key, preferred region, and output format when prompted.

#### 4. Crafting Your Initial Terraform Configuration

- **Create a Project Directory:**
  ```bash
  mkdir terraform-aws
  cd terraform-aws
  ```

- **Create `main.tf`:**
  ```hcl
  provider "aws" {
    region = "us-west-2"
  }

  resource "aws_instance" "example" {
    ami           = "ami-0c55b159cbfafe1f0" # Amazon Linux 2
    instance_type = "t2.micro"

    tags = {
      Name = "TerraformExample"
    }
  }

  output "instance_id" {
    value = aws_instance.example.id
  }
  ```

- **Initialize Terraform:**
  ```bash
  terraform init
  ```

- **Plan the Deployment:**
  ```bash
  terraform plan
  ```

- **Apply the Configuration:**
  ```bash
  terraform apply
  ```
  - Confirm by typing `yes` when prompted.

#### 5. Managing Terraform State

- **View Current State:**
  ```bash
  terraform show
  ```

- **Destroy Provisioned Infrastructure:**
  ```bash
  terraform destroy
  ```
  - Confirm by typing `yes` when prompted.

#### 6. Utilizing Variables and Outputs

- **Modify `main.tf` to Incorporate Variables:**
  ```hcl
  variable "region" {
    default = "us-west-2"
  }

  variable "instance_type" {
    default = "t2.micro"
  }

  provider "aws" {
    region = var.region
  }

  resource "aws_instance" "example" {
    ami           = "ami-0c55b159cbfafe1f0"
    instance_type = var.instance_type

    tags = {
      Name = "TerraformExample"
    }
  }

  output "instance_id" {
    value = aws_instance.example.id
  }
  ```

- **Apply Configuration with Variables:**
  ```bash
  terraform apply -var="instance_type=t2.small"
  ```

### Hands-On Exercises

1. **Exercise 1:** Develop and deploy a Terraform configuration to launch an AWS EC2 instance.
   - **Steps:**
     ```bash
     mkdir terraform-aws
     cd terraform-aws
     # Create main.tf with the provided content
     terraform init
     terraform plan
     terraform apply
     ```
     - Verify the instance in the AWS Management Console.

2. **Exercise 2:** Adjust the configuration to use variables for specifying the region and instance type.
   - **Steps:**
     - Update `main.tf` as illustrated above.
     - Apply the configuration with a custom instance type:
       ```bash
       terraform apply -var="instance_type=t2.small"
       ```

3. **Exercise 3:** Retrieve and display the EC2 instance ID post-deployment.
   - **Steps:**
     ```bash
     terraform output instance_id
     ```

4. **Exercise 4:** Remove the deployed infrastructure using Terraform.
   - **Steps:**
     ```bash
     terraform destroy
     ```
     - Confirm by typing `yes` when prompted.

---

## Module 4: Deploying Applications with Terraform and Kubernetes

### Learning Objectives
- Leverage Terraform to manage Kubernetes clusters and resources.
- Deploy applications using Terraform in combination with Kubernetes.

### Course Content

#### 1. Installing the Terraform Kubernetes Provider

- **Enhance `main.tf` to Include the Kubernetes Provider:**
  ```hcl
  provider "kubernetes" {
    config_path = "~/.kube/config"
  }
  ```

- **Initialize Terraform:**
  ```bash
  terraform init
  ```

#### 2. Managing Kubernetes Namespaces with Terraform

- **Add a Namespace Resource to `main.tf`:**
  ```hcl
  resource "kubernetes_namespace" "example" {
    metadata {
      name = "example-namespace"
    }
  }
  ```

- **Apply the Configuration:**
  ```bash
  terraform apply
  ```
  - Confirm by typing `yes`.

#### 3. Deploying a Kubernetes Deployment via Terraform

- **Incorporate Deployment and Service Resources into `main.tf`:**
  ```hcl
  resource "kubernetes_deployment" "my_node_app" {
    metadata {
      name      = "my-node-app"
      namespace = kubernetes_namespace.example.metadata[0].name
    }

    spec {
      replicas = 2

      selector {
        match_labels = {
          app = "my-node-app"
        }
      }

      template {
        metadata {
          labels = {
            app = "my-node-app"
          }
        }

        spec {
          container {
            name  = "my-node-app"
            image = "my-node-app:1.0"
            ports {
              container_port = 3000
            }
          }
        }
      }
    }
  }

  resource "kubernetes_service" "my_node_app_service" {
    metadata {
      name      = "my-node-app-service"
      namespace = kubernetes_namespace.example.metadata[0].name
    }

    spec {
      selector = {
        app = kubernetes_deployment.my_node_app.spec.0.template.0.metadata.0.labels.app
      }

      port {
        port        = 3000
        target_port = 3000
        node_port   = 30001
      }

      type = "NodePort"
    }
  }
  ```

- **Apply the Configuration:**
  ```bash
  terraform apply
  ```
  - Confirm by typing `yes`.

- **Access the Application:**
  ```bash
  minikube service my-node-app-service --namespace=example-namespace
  ```
  - This will open the service in your browser.

#### 4. Automating Application Deployment

- **Build and Load the Docker Image into Minikube:**
  ```bash
  eval $(minikube docker-env)
  docker build -t my-node-app:1.0 ./my-node-app
  minikube image load my-node-app:1.0
  eval $(minikube docker-env -u)
  ```

- **Ensure Terraform References the Correct Image:**
  - Verify that the image field in `kubernetes_deployment` is set to `my-node-app:1.0`.

- **Re-Apply Terraform Configuration:**
  ```bash
  terraform apply
  ```
  - Confirm by typing `yes` if prompted.

### Hands-On Exercises

1. **Exercise 1:** Set up Terraform to create a Kubernetes namespace.
   - **Steps:**
     ```hcl
     resource "kubernetes_namespace" "example" {
       metadata {
         name = "example-namespace"
       }
     }
     ```
     ```bash
     terraform apply
     ```

2. **Exercise 2:** Deploy the Node.js application within the newly created namespace using Terraform.
   - **Steps:**
     ```hcl
     resource "kubernetes_deployment" "my_node_app" { ... }
     ```
     ```bash
     terraform apply
     ```

3. **Exercise 3:** Expose the application through a Kubernetes Service using Terraform.
   - **Steps:**
     ```hcl
     resource "kubernetes_service" "my_node_app_service" { ... }
     ```
     ```bash
     terraform apply
     ```

4. **Exercise 4:** Access the deployed application via the service URL.
   - **Steps:**
     ```bash
     minikube service my-node-app-service --namespace=example-namespace
     ```
     - Confirm the message in your browser.

---

## Module 5: Helm for Kubernetes

### Learning Objectives
- Streamline Kubernetes application management with Helm.
- Utilize Helm charts, manage releases, and customize deployments.

### Course Content

#### 1. Installing Helm

- **Install Helm Using Homebrew:**
  ```bash
  brew install helm
  ```
  *Verify Installation:*
  ```bash
  helm version
  ```
  *Sample Output:*
  ```
  version.BuildInfo{Version:"v3.6.3", GitCommit:"...", GitTreeState:"clean", GoVersion:"go1.16.5"}
  ```

#### 2. Grasping Helm Concepts

- **Core Concepts:**
  - **Charts:** Bundles of pre-configured Kubernetes resources.
  - **Releases:** Instances of charts deployed in a Kubernetes cluster.
  - **Values:** Configuration parameters that allow customization of charts.

#### 3. Deploying a Helm Chart

- **Add the Ingress Nginx Helm Repository:**
  ```bash
  helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
  helm repo update
  ```

- **Install the Nginx Ingress Controller Chart:**
  ```bash
  helm install my-nginx ingress-nginx/ingress-nginx
  ```

- **Confirm Installation:**
  ```bash
  helm list
  kubectl get pods -n default
  ```

#### 4. Customizing Helm Deployments

- **Create a `values.yaml` File:**
  ```yaml
  replicaCount: 2
  image:
    repository: nginx
    tag: stable
  service:
    type: NodePort
    port: 80
    nodePort: 30002
  ```

- **Deploy with Customized Values:**
  ```bash
  helm install my-custom-nginx ingress-nginx/ingress-nginx -f values.yaml
  ```

- **Access the Nginx Service:**
  ```bash
  minikube service my-custom-nginx-ingress-nginx-controller --url
  ```
  - Use the provided URL to reach Nginx.

#### 5. Managing Helm Releases

- **Upgrade an Existing Release:**
  - **Modify `values.yaml` (e.g., increase `replicaCount` to 3).**
  - **Apply the Upgrade:**
    ```bash
    helm upgrade my-custom-nginx ingress-nginx/ingress-nginx -f values.yaml
    ```

- **Rollback to a Previous Release:**
  ```bash
  helm rollback my-custom-nginx 1
  ```

- **Remove a Helm Release:**
  ```bash
  helm uninstall my-custom-nginx
  ```

### Hands-On Exercises

1. **Exercise 1:** Deploy the Nginx Helm chart in Minikube.
   - **Steps:**
     ```bash
     helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
     helm repo update
     helm install my-nginx ingress-nginx/ingress-nginx
     ```

2. **Exercise 2:** Customize the Helm chart by editing `values.yaml` to set a specific replica count and service type.
   - **Steps:**
     - Create a `values.yaml` with desired settings.
     - Deploy using the customized values:
       ```bash
       helm install my-custom-nginx ingress-nginx/ingress-nginx -f values.yaml
       ```

3. **Exercise 3:** Upgrade the Helm release with new configurations.
   - **Steps:**
     - Update `values.yaml` (e.g., change `replicaCount` to 3).
     - Apply the upgrade:
       ```bash
       helm upgrade my-custom-nginx ingress-nginx/ingress-nginx -f values.yaml
       ```

4. **Exercise 4:** Rollback the Helm release to a previous version.
   - **Steps:**
     ```bash
     helm rollback my-custom-nginx 1
     ```

5. **Exercise 5:** Uninstall the Helm release.
   - **Steps:**
     ```bash
     helm uninstall my-custom-nginx
     ```

---

## Module 6: Advanced Kubernetes Concepts

### Learning Objectives
- Utilize advanced Kubernetes features such as ConfigMaps, Secrets, PersistentVolumes, and Ingress.
- Set up monitoring and logging for Kubernetes applications.

### Course Content

#### 1. ConfigMaps and Secrets for Application Configuration

- **Create a ConfigMap:**
  ```bash
  kubectl create configmap app-config --from-literal=ENV=production --namespace=example-namespace
  ```
  *Verify Creation:*
  ```bash
  kubectl get configmaps -n example-namespace
  ```

- **Create a Secret:**
  ```bash
  kubectl create secret generic app-secret --from-literal=PASSWORD=supersecret --namespace=example-namespace
  ```
  *Verify Creation:*
  ```bash
  kubectl get secrets -n example-namespace
  ```

- **Update Deployment to Reference ConfigMap and Secret:**
  - **Modify `main.tf` or Deployment YAML:**
    ```hcl
    spec {
      containers {
        name  = "my-node-app"
        image = "my-node-app:1.0"
        ports {
          container_port = 3000
        }
        env {
          name = "ENV"
          value_from {
            config_map_key_ref {
              name = "app-config"
              key  = "ENV"
            }
          }
        }
        env {
          name = "PASSWORD"
          value_from {
            secret_key_ref {
              name = "app-secret"
              key  = "PASSWORD"
            }
          }
        }
      }
    }
    ```

- **Apply the Updated Configuration:**
  ```bash
  terraform apply
  ```

#### 2. Persistent Storage with PersistentVolumes

- **Define a PersistentVolumeClaim (PVC):**
  - **Create `pvc.yaml`:**
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: app-pvc
      namespace: example-namespace
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
    ```

- **Apply the PVC:**
  ```bash
  kubectl apply -f pvc.yaml
  ```

- **Confirm PVC Status:**
  ```bash
  kubectl get pvc -n example-namespace
  ```

- **Mount PVC to Deployment:**
  - **Update Deployment YAML:**
    ```hcl
    spec {
      containers {
        name  = "my-node-app"
        image = "my-node-app:1.0"
        ports {
          container_port = 3000
        }
        volume_mounts {
          mount_path = "/usr/src/app/data"
          name       = "data-volume"
        }
        env {
          name = "ENV"
          value_from {
            config_map_key_ref {
              name = "app-config"
              key  = "ENV"
            }
          }
        }
        env {
          name = "PASSWORD"
          value_from {
            secret_key_ref {
              name = "app-secret"
              key  = "PASSWORD"
            }
          }
        }
      }
      volumes {
        name = "data-volume"
        persistent_volume_claim {
          claim_name = "app-pvc"
        }
      }
    }
    ```

- **Apply the Configuration:**
  ```bash
  terraform apply
  ```

#### 3. Ingress Controller and Rules

- **Enable Ingress Addon in Minikube:**
  ```bash
  minikube addons enable ingress
  ```

- **Create an Ingress Resource:**
  - **Create `ingress.yaml`:**
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: my-node-app-ingress
      namespace: example-namespace
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      rules:
        - host: my-node-app.local
          http:
            paths:
              - path: /
                pathType: Prefix
                backend:
                  service:
                    name: my-node-app-service
                    port:
                      number: 3000
    ```

- **Update `/etc/hosts` with the Ingress Host:**
  ```bash
  echo "$(minikube ip) my-node-app.local" | sudo tee -a /etc/hosts
  ```

- **Apply the Ingress Resource:**
  ```bash
  kubectl apply -f ingress.yaml
  ```

- **Access the Application via Ingress:**
  - Open `http://my-node-app.local` in your browser to view the application.

#### 4. Monitoring with Prometheus and Grafana

- **Install Prometheus Using Helm:**
  ```bash
  helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
  helm repo update
  helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace
  ```

- **Install Grafana Using Helm:**
  ```bash
  helm install grafana prometheus-community/grafana --namespace monitoring
  ```

- **Access Grafana Dashboard:**
  ```bash
  kubectl port-forward service/grafana 3000:80 --namespace monitoring
  ```
  - Navigate to `http://localhost:3000` in your browser.
  - **Default Credentials:**
    - **Username:** admin
    - **Password:** Retrieve using:
      ```bash
      kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
      ```

#### 5. Logging with Fluentd

- **Deploy Fluentd via Helm:**
  ```bash
  helm repo add stable https://charts.helm.sh/stable
  helm repo update
  helm install fluentd stable/fluentd --namespace logging --create-namespace
  ```
  *Note:* The `stable` repository might be deprecated. Consider using an alternative Fluentd Helm chart or deploy manually.

- **Verify Fluentd Deployment:**
  ```bash
  kubectl get pods -n logging
  ```

### Hands-On Exercises

1. **Exercise 1:** Create and utilize ConfigMaps and Secrets within your Kubernetes deployment.
   - **Steps:**
     ```bash
     kubectl create configmap app-config --from-literal=ENV=production --namespace=example-namespace
     kubectl create secret generic app-secret --from-literal=PASSWORD=supersecret --namespace=example-namespace
     # Update Deployment to reference ConfigMap and Secret
     terraform apply
     ```

2. **Exercise 2:** Establish a PersistentVolumeClaim and integrate it with your application.
   - **Steps:**
     ```bash
     kubectl apply -f pvc.yaml
     # Update Deployment to mount the PVC
     terraform apply
     ```

3. **Exercise 3:** Set up Ingress to expose your application on a custom domain.
   - **Steps:**
     ```bash
     kubectl apply -f ingress.yaml
     echo "$(minikube ip) my-node-app.local" | sudo tee -a /etc/hosts
     # Access via http://my-node-app.local
     ```

4. **Exercise 4:** Deploy Prometheus and Grafana using Helm and visualize application metrics.
   - **Steps:**
     ```bash
     helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
     helm repo update
     helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace
     helm install grafana prometheus-community/grafana --namespace monitoring
     kubectl port-forward service/grafana 3000:80 --namespace monitoring
     # Access Grafana at http://localhost:3000
     ```

5. **Exercise 5:** Implement Fluentd for centralized log aggregation.
   - **Steps:**
     ```bash
     helm repo add stable https://charts.helm.sh/stable
     helm repo update
     helm install fluentd stable/fluentd --namespace logging --create-namespace
     ```

---

## Module 7: Capstone Project

### Learning Objectives
- Integrate Docker, Terraform, Kubernetes, Minikube, and Helm to deploy a comprehensive full-stack application.
- Apply all acquired knowledge to construct, deploy, and manage a real-world application.

### Course Content

#### 1. Planning the Project

- **Design Application Architecture:**
  - **Components:**
    - **Frontend:** React application.
    - **Backend:** Node.js API.
    - **Database:** MongoDB.

- **Organize Directory Structure:**
  ```
  capstone-project/
  ├── frontend/
  │   ├── Dockerfile
  │   ├── package.json
  │   └── src/
  ├── backend/
  │   ├── Dockerfile
  │   ├── package.json
  │   └── app.js
  ├── terraform/
  │   └── main.tf
  ├── kubernetes/
  │   ├── frontend-deployment.yaml
  │   ├── backend-deployment.yaml
  │   └── mongo-deployment.yaml
  └── helm/
      └── charts/
  ```

#### 2. Creating Docker Images

- **Frontend Dockerfile (React App):**
  ```dockerfile
  FROM node:14 AS build
  WORKDIR /app
  COPY package*.json ./
  RUN npm install
  COPY . .
  RUN npm run build

  FROM nginx:alpine
  COPY --from=build /app/build /usr/share/nginx/html
  EXPOSE 80
  CMD ["nginx", "-g", "daemon off;"]
  ```

- **Backend Dockerfile (Node.js API):**
  ```dockerfile
  FROM node:14
  WORKDIR /usr/src/app
  COPY package*.json ./
  RUN npm install
  COPY . .
  EXPOSE 3000
  CMD ["node", "app.js"]
  ```

- **Build and Load Images into Minikube:**
  ```bash
  eval $(minikube docker-env)
  docker build -t frontend:1.0 ./frontend
  docker build -t backend:1.0 ./backend
  minikube image load frontend:1.0
  minikube image load backend:1.0
  eval $(minikube docker-env -u)
  ```

#### 3. Deploying with Helm

- **Generate Helm Charts for Frontend and Backend:**
  ```bash
  helm create frontend
  helm create backend
  ```

- **Customize `values.yaml` for Both Charts:**
  - **Frontend `values.yaml`:**
    ```yaml
    replicaCount: 2
    image:
      repository: frontend
      tag: "1.0"
    service:
      type: NodePort
      port: 80
      nodePort: 30003
    ingress:
      enabled: true
      hosts:
        - host: frontend.local
          paths:
            - path: /
              pathType: Prefix
    ```

  - **Backend `values.yaml`:**
    ```yaml
    replicaCount: 2
    image:
      repository: backend
      tag: "1.0"
    service:
      type: NodePort
      port: 3000
      nodePort: 30004
    ingress:
      enabled: true
      hosts:
        - host: backend.local
          paths:
            - path: /
              pathType: Prefix
    ```

- **Deploy Helm Releases:**
  ```bash
  helm install frontend ./frontend --namespace capstone --create-namespace
  helm install backend ./backend --namespace capstone
  ```

- **Deploy MongoDB via Helm:**
  ```bash
  helm repo add bitnami https://charts.bitnami.com/bitnami
  helm repo update
  helm install mongo bitnami/mongodb --namespace capstone
  ```

#### 4. Setting Up Infrastructure with Terraform

- **Create `main.tf` for Kubernetes Resources:**
  ```hcl
  provider "kubernetes" {
    config_path = "~/.kube/config"
  }

  resource "kubernetes_namespace" "capstone" {
    metadata {
      name = "capstone"
    }
  }

  # Optionally manage Helm releases with Terraform Helm provider
  provider "helm" {
    kubernetes {
      config_path = "~/.kube/config"
    }
  }

  resource "helm_release" "frontend" {
    name       = "frontend"
    repository = "file://./frontend"
    chart      = "./frontend"
    namespace  = kubernetes_namespace.capstone.metadata[0].name
    values     = [file("./helm/charts/frontend/values.yaml")]
  }

  resource "helm_release" "backend" {
    name       = "backend"
    repository = "file://./backend"
    chart      = "./backend"
    namespace  = kubernetes_namespace.capstone.metadata[0].name
    values     = [file("./helm/charts/backend/values.yaml")]
  }
  ```

- **Initialize and Apply Terraform Configuration:**
  ```bash
  cd terraform
  terraform init
  terraform apply
  ```
  - Confirm by typing `yes`.

#### 5. Testing and Scaling

- **Validate the Application:**
  - **Update `/etc/hosts` for Frontend Access:**
    ```bash
    echo "$(minikube ip) frontend.local" | sudo tee -a /etc/hosts
    ```
    - Open `http://frontend.local` in your browser.
  - **Ensure Frontend Communicates with Backend and Database.**

- **Scale Application Components:**
  ```bash
  kubectl scale deployment frontend --replicas=3 -n capstone
  kubectl scale deployment backend --replicas=3 -n capstone
  kubectl get deployments -n capstone
  ```

#### 6. Monitoring and Logging

- **Ensure Prometheus and Grafana are Monitoring the Project:**
  - **Add Necessary Annotations or Configurations in Helm Charts to Expose Metrics.**

- **Visualize Metrics in Grafana:**
  - **Access Grafana Dashboard:**
    ```bash
    kubectl port-forward service/grafana 3000:80 --namespace monitoring
    ```
    - Navigate to `http://localhost:3000` and set up dashboards for frontend and backend performance.

### Hands-On Exercises

1. **Exercise 1:** Design and outline the architecture for a full-stack application.
   - **Solution:**
     - **Identify Components:** Frontend (React), Backend (Node.js), Database (MongoDB).
     - **Establish Directory Structure.**

2. **Exercise 2:** Create Docker images for frontend and backend services.
   - **Solution:**
     ```bash
     eval $(minikube docker-env)
     docker build -t frontend:1.0 ./frontend
     docker build -t backend:1.0 ./backend
     minikube image load frontend:1.0
     minikube image load backend:1.0
     eval $(minikube docker-env -u)
     ```

3. **Exercise 3:** Deploy application components using Helm charts in Kubernetes.
   - **Solution:**
     ```bash
     helm create frontend
     helm create backend
     # Customize values.yaml for both charts
     helm install frontend ./frontend --namespace capstone --create-namespace
     helm install backend ./backend --namespace capstone
     ```

4. **Exercise 4:** Utilize Terraform to provision Kubernetes namespaces and related resources.
   - **Solution:**
     ```hcl
     # main.tf as provided above
     terraform apply
     ```

5. **Exercise 5:** Test the complete application to confirm all components interact correctly.
   - **Solution:**
     - Access the frontend and verify communication with the backend and database.

6. **Exercise 6:** Scale the application components and observe the effects.
   - **Solution:**
     ```bash
     kubectl scale deployment frontend --replicas=3 -n capstone
     kubectl scale deployment backend --replicas=3 -n capstone
     kubectl get deployments -n capstone
     ```

7. **Exercise 7:** Implement monitoring for the application using Prometheus and Grafana.
   - **Solution:**
     ```bash
     helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace
     helm install grafana prometheus-community/grafana --namespace monitoring
     kubectl port-forward service/grafana 3000:80 --namespace monitoring
     # Configure Grafana dashboards accordingly
     ```

---

## Outcome

Upon completing this course, you will have:

- **Docker:**
  - Created and managed Docker containers and images.
  - Employed Docker volumes and networking for data persistence and inter-container communication.

- **Kubernetes (Minikube):**
  - Deployed, scaled, and managed applications within a Kubernetes cluster.
  - Utilized advanced Kubernetes features like ConfigMaps, Secrets, PersistentVolumes, and Ingress.

- **Terraform:**
  - Developed Infrastructure as Code (IaC) configurations to provision and manage cloud resources.
  - Integrated Terraform with Kubernetes for automated deployments.

- **Helm:**
  - Streamlined Kubernetes application management using Helm charts and releases.
  - Customized and efficiently managed application deployments.

- **Full-Stack Application Deployment:**
  - Combined Docker, Kubernetes, Terraform, and Helm to deploy a comprehensive full-stack application.
  - Set up monitoring and logging to ensure application health and performance.

- **DevOps Skills:**
  - Automated infrastructure and application deployments using contemporary DevOps tools.
  - Seamlessly transitioned from local development environments to cloud-based Kubernetes clusters.

---

**Congratulations on completing the Zero-to-Hero course! You are now equipped with the essential skills to build, deploy, and manage modern applications using Docker, Kubernetes, Terraform, and Helm.**

---
