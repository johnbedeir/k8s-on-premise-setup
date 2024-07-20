# On-Premise Setup on Minikube using Ingress and MetalLB

This guide provides instructions to set up a Minikube cluster with Ingress, MetalLB, and StorageClass, and exposes the application via ngrok or router port forwarding.

## Prerequisites

- Minikube
- Docker

## Enable Ingress on Minikube

```sh
minikube addons enable ingress
```

## Enable MetalLB on Minikube

```sh
minikube addons enable metallb
minikube addons configure metallb
```

## Enable StorageClass on Minikube

```sh
minikube addons enable default-storageclass
```

## Add Minikube App Domain Locally

```sh
sudo echo "$(minikube ip) app.johnydev.com" | sudo tee -a /etc/hosts
```

## Install MetalLB on Minikube

Set the driver for Minikube:

```sh
minikube config set driver virtualbox
```

Deploy MetalLB on Minikube:

```sh
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml
```

Set the MetalLB configuration:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - IP_RANGE_START-IP_RANGE_END
```

Replace the IP range with a range that fits your network.

Run the following command to get the Minikube IP:

```sh
minikube ip
```

For example, if the IP returned is `192.168.49.2`, it indicates your Minikube VM is on the `192.168.49.0/24` subnet.

Determine a range of IP addresses within your subnet that are not currently in use. This can usually be done by checking your router's DHCP configuration to see which IP addresses are assigned automatically.

Verify IP Address Availability:

```sh
for ip in $(seq 240 250); do ping -c 1 192.168.49.$ip; done
```

This will check the availability of the IP range from `192.168.49.240` to `192.168.49.250`.

Apply and save RBAC configuration:

```sh
kubectl create --save-config -f k8s/rbac.yml
```

## Expose Application Publicly

### Option 1: ngrok

1. Create an account on [ngrok](https://dashboard.ngrok.com/signup).
2. Login.
3. Download ngrok on your machine.
4. Add your auth token by running the command presented to you:

```sh
ngrok config add-authtoken TOKEN
```

5. Generate a public domain to access your app:

```sh
ngrok http http://APP_EXTERNAL_IP:APP_PORT
```

You will find the domain to access your application from the ngrok dashboard/Endpoints.

#### To Keep the Command Running:

Run ngrok in a container on the host machine or on a VM on the cloud:

```sh
docker run -d --net=host -it -e NGROK_AUTHTOKEN=TOKEN ngrok/ngrok:latest http http://APP_EXTERNAL_IP:APP_PORT
```

### Option 2: Router Port Forwarding

1. Login to your router configuration.
2. In the port forward page, create the following rule:

```
Custom Service Name: MinikubeApp
Service: Other
Protocol: TCP
External Host: (leave empty)
External Port: 80
Internal Host: minikube ip
Internal Port: Application port 8080 or 30000 as it was set
```

For the Minikube IP, it should be in the same range as your router (local IPv4 IP). For example, if your local IPv4 IP is `192.168.0.1`, then Minikube should be `192.168.0.x`. You can achieve this by starting Minikube using the following command:

```sh
minikube start --driver=virtualbox --host-only-cidr="192.168.0.1/24"
```

**Note:** If Minikube gives an error that it cannot start with that IP, the manual way is to access the VirtualBox network adapter and try to add the IP range. If it doesn't accept, then this method won't work for you.
