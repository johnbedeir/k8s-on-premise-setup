- minikube
- docker

Enable Ingress on Minikube

```
minikube addons enable ingress
```

Add minikube app domain locally:

```
sudo echo "$(minikube ip) app.johnydev.com" | sudo tee -a /etc/hosts
```

Install metallb on minikube

```
minikube config set driver virtualbox
```

OR

```
minikube config set driver qemu2
```

Deploy Metallb on minikube:

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml
```

Set the metallb config:

```
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

```
minikube ip
```

For example, if the IP returned is 192.168.49.2, it indicates your Minikube VM is on the 192.168.49.0/24 subnet.

Determine a range of IP addresses within your subnet that are not currently in use. This can usually be done by checking your router's DHCP configuration to see which IP addresses are assigned automatically.

Verify IP Address Availability:

```
for ip in $(seq 240 250); do ping -c 1 192.168.49.$ip; done
```

That will check the availability of the IP range from 192.168.49.240 to 192.168.49.250

Apply and save RBAC configuration:

```
kubectl create --save-config -f k8s/rbac.yml
```
