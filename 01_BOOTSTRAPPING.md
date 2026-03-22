# Welcome to the Prep of the vCluster Workshop at KubeCon EU 2026 - Bootstrapping the Platform 01! 💻

> ⚠️ Note: You will need just 5–10 minutes to complete the setup and then you are ready for the workshop session!

This guide is for the workshop session:

**1:00 – 2:30 PM: Building a Multi-Tenancy Platform Beyond Kubernetes – Alexander Hoeft and Artem Lajko**

Please make sure to read the [prerequisites and setup steps](00_PREREQUISITES.md) before the workshop to ensure a smooth experience during the session!

Fork this [repository](https://github.com/la-demos/vcluster-platform-blueprint-kubecon-eu-26).

You will find the following folder structure in the repo:

## Folder Structure

* `kubara` (binary built for MacOS Arm64. You can download another version from the [Pocket kubara releases](https://github.com/la-demos/vcluster-platform-blueprint-kubecon-eu-26/releases/tag/0.0.0) or build it yourself from the [Pocket kubara](https://github.com/la-demos/pocket-kubara).

* `css-bitwarden.yaml` (example file)
* `vcluster-config-controlplane.yaml` (example file)
* `00_PREREQUISITES.md` (prerequisites and setup steps)
* `01_BOOTSTRAPPING.md` (this file)
* `02_ADDITIONAL_USE_CASES.md` (additional use cases for the setup we have created in this workshop)
* `100_ADDITIONAL_INFORMATION.md` (additional information beyond the workshop content)
* `z_images` (images used in the documentation)


Let’s start!

# Step 1 - vCluster Setup 💻

For this workshop, we are running [vCluster in Docker (vind)](https://github.com/loft-sh/vind), so everyone is able to build a full Kubernetes platform on top of it.


<img src="z_images/01-001.png" width="1000" style="border-radius: 24px;" />

## Prep Steps

```bash
vcluster upgrade --version v0.32.0
vcluster use driver docker
```

## vCluster Platform

> ⚠️ Note: you will need to create an account after the platform is started. You will getting automatically logged in and just need to fill out missing data.

```bash
vcluster platform start --version v4.7.0
```

## Deploy a vCluster as Control Plane

> ⚠️ macOS: Run the command with `sudo` to allow vCluster to create the required loopback interface aliases. Without these privileges, LoadBalancer services will not be supported and a warning will be shown. Note that `sudo` is not required for creating the Docker network itself

```bash
sudo vcluster create controlplane -f vcluster-config-controlplane.yaml
```

You should now have a running vCluster with 2 worker nodes. You can check the status accordingly.

```bash
vcluster list

 NAME        | STATUS  | CONNECTED | AGE
---------------+---------+-----------+------
controlplane | running | True      | 13h

# Or

kubectl get nodes

NAME           STATUS   ROLES                  AGE   VERSION
controlplane   Ready    control-plane,master   13h   v1.35.0
worker-1       Ready    <none>                 13h   v1.35.0
worker-2       Ready    <none>                 13h   v1.35.0
```

Now let’s start setting up the platform on top of it.

# Step 2 - Kubara Setup Explained 💻

## What is kubara?

Kubara is an open-source framework from platform engineers for platform engineers, created by STACKIT to help users bootstrap a production-ready Kubernetes platform based on distros.

For this workshop, we use a lightweight fork of the Kubara general distro. This is not an official release and was created specifically for the workshop, so you have exclusive access to it.

If you want to take a closer look, just visit the [repository](https://github.com/kubara-io/kubara).

You will start with:

```bash
./kubara init --prep
```

This will create a `.env`. You can use most of the values from here, just replace the Git repository values with your own.
The following is just an example:

```bash
# 🔁 Everything in <angle brackets> MUST be replaced.
# 💡 Dummy values (without <>) are optional and can be left as-is if not needed

### Project related values
PROJECT_NAME='controlplane'
PROJECT_STAGE='prod'
DOMAIN_NAME='172.18.255.254.traefik.me'

### Argo CD related values
ARGOCD_WIZARD_ACCOUNT_PASSWORD='Sw0rdF1sh!42_1'

### Git repository values
ARGOCD_GIT_HTTPS_URL='https://github.com/<REPLACE_ME>'
```

After that, run the init command to create a `config.yaml`:

```bash
./kubara init
```

Fill out the missing values in the `config.yaml`. Override values as needed. For example, if your GitOps repository does not use the `main` branch, override `services.argocd.repo.https.targetRevision`. Otherwise, you can keep the default values.

The only value you might need to change is the DNS (IP address), depending on your container engine network setup. You will get this information later, so you can ignore this block for now:

`dnsName: cp-workshop-prod.172.18.255.254.traefik.me`
→ `dnsName: cp-workshop-prod.YOUR-IP-ADDRESS.traefik.me`

# Step 3 - Bootstrapping the Kubernetes Platform 💻

## External Secrets Setup with Bitwarden

> ⚠️ Note: you will need here the creaed accoess token from the machine account.

In this part, you will create a secret for the Bitwarden access token in the `external-secrets` namespace.

```bash
kubectl create ns external-secrets
```

```bash
kubectl create secret generic bitwarden-access-token \
  --from-literal=token="0.a046...." \
  -n external-secrets
```

## Generate

```bash
./kubara generate --helm
```

This will create a catalog with Helm umbrella charts and dedicated overlays for the controlplane cluster.

Push and commit the changes to your Git repository and then bootstrap your platform!!

## Bootstrapping the Platform

Hopefully, you already replaced the `organizationID` and `projectID` in the `css-bitwarden.yaml` file.
If not, do it now. Otherwise, check the prerequisites and complete the Bitwarden setup first.

Then execute:

```bash
./kubara bootstrap --with-es-css-file css-bitwarden.yaml controlplane --with-es-crds
```

* controlplane is the `PROJECT_NAME` you have set in your `.env` file.

Now check if everything is up and running:

```bash
kubectl get pods -A
```

If something is stuck, check the Argo CD setup using port forwarding:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open:

```
http://localhost:8080/argocd
```

> ⚠️ Note: Since we deploy everything in Docker locally, it may take a few minutes for Argo CD to generate all applications. You've earned a 5-minute break — grab a tea and come back shortly.

Log in with user `wizard` and your chosen password.

If you see a white screen, clear your browser cache or open the page in incognito mode.

If everything is running correctly, the Traefik service should now have an external IP address:

```bash
kubectl get svc -n traefik
```

Example:

```bash
NAME      TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                      AGE
traefik   LoadBalancer   10.105.221.10    172.18.255.254   80:31291/TCP,443:30840/TCP   22m
```

If you have the same address, you don’t need to change anything.

Just open your ingress host in the browser:

```bash
kubectl get ing -n argocd

NAME                        CLASS     HOSTS                                     ADDRESS          PORTS   AGE
argocd-additional-ingress   traefik   controlplane-prod.172.18.255.254.traefik.me   172.18.255.254   80      15h
```

Host:
[http://controlplane-prod.172.18.255.254.traefik.me/argocd/](http://controlplane-prod.172.18.255.254.traefik.me/argocd/)

You should see the Argo CD dashboard after logging in.

You will also see the homer-dashboard if you just open the host without the `/argocd` path:

[http://controlplane-prod.172.18.255.254.traefik.me](http://controlplane-prod.172.18.255.254.traefik.me)


<img src="z_images/kubara-developer-portal.png" width="1000" style="border-radius: 24px;" />

# Congratulations!

You have successfully set up a fully running environment.

We look forward to seeing you on March 23 at the workshop, where we’ll explore the environment and show you some cool things you can build with it!

Don't forget to leave a ⭐️ on the [vind repository](https://github.com/loft-sh/vind) or [kubara repository](https://github.com/kubara-io/kubara) to show your support for the community project!
