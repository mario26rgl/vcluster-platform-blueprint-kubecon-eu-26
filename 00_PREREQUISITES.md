# Prerequisites - 00

Please ensure you have the following prerequisites in place before starting the workshop.

## Required Software

Please ensure the following tools are installed:

* **[Docker Desktop](https://www.docker.com/products/docker-desktop/)**, **[OrbStack](https://orbstack.dev/)**, or another container runtime
* **[kubectl](https://kubernetes.io/docs/tasks/tools/)**
* **[vcluster CLI](https://www.vcluster.com/docs/platform/install/quick-start-guide)**
* **[Helm](https://helm.sh/docs/intro/install/)**
* **[Git CLI](https://git-scm.com/install/)**
* Access to a **Git repository**

## Hardware Requirements

* **16–32 GB RAM**
* **8–16 CPU cores**
* **50–100 GB free disk space**
* **Linux or macOS**

> ⚠️ Windows is not recommended for this workshop and has not been tested.

## Recommended Configuration

To ensure a smooth workshop experience, increase your container runtime (e.g., Docker Desktop) resource limits to at least:

* **10–16 GB RAM**
* **6–8 CPU cores**
* **50–100 GB free disk space**

**⚠️ Note for Docker users ⚠️**

Our vCluster setup currently assumes that the Docker network uses the subnet 172.20.0.0/16.

If not adjusted, the Ingress resources may contain incorrect domains, and you might not be able to access Argo CD or the Homer Dashboard.

**What to do:**

Update the subnet/IP-related values in the values.yaml files (Argo CD and Homer) to match your Docker network subnet before deploying.

# Tool Requirements

All required tools are available in their free tier.

* **Bitwarden Secrets Manager**

# Bitwarden Secrets Manager Setup

## 1. Create an Account

Go to [https://bitwarden.com](https://bitwarden.com) and create an account and organization if you don’t already have one.

> ⚠️ Switch to **Secrets Manager** in the left sidebar!

## 2. Create a Project

* Create a new project.
* Copy the **Project ID**.

Example:

```
Project ID: 1c2eb3f1-360d-42...
```

## 3. Create a Machine Account

* Create a new machine account.
* Assign it to the previously created project.
* Generate an **Access Token**.
* Copy the access token. You will need it later.

Go to `Machine Accounts` → `Choose your machine account` → `Config`

and copy the following:

* **Identity Server URL**
  `https://vault.bitwarden.com/identity`

* **API Server URL**
  `https://vault.bitwarden.com/api`

* **Organization ID**
  `af3fe9e1-e0bc-4f66-...`

If you are using bitwarden.com, you mainly need:

* Organization ID
* Project ID

and can keep the default URLs in the provided `css-bitwarden.yaml` file.

## 4. Update Configuration File

Open the [css-bitwarden.yaml](css-bitwarden.yaml) file and replace:

* `organizationID`
* `projectID`

with your own values.

# Required Skills

This is **not a beginner-level workshop**.

You should have a basic understanding of:

* Kubernetes
* Helm
* GitOps
* Linux
* Git

You don’t need to be an expert, but you should:

* Be comfortable navigating Kubernetes commands
* Know how to interact with the Kubernetes API
* Be able to retrieve relevant cluster information
* Understand basic Helm concepts
* Be familiar with Git workflows and commands

Now let's move on to the next step and start [bootstrapping the platform](01_BOOTSTRAPPING.md)!