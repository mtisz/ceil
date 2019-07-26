# `max`: Auto-provisioned Mini PC (amd64) cluster running K8S on bare-metal

Enter `make help` to see available commands.

Hints:
* See [branch master](https://github.com/helmuthva/ceil) for the RPi variant of this cluster
* See [Project jetson](https://github.com/helmuthva/jetson) on how to add an Nvidia Jetson Nano (arm64) as a worker node to this cluster.

## Goals

* Setup auto-provisioned Mini PC (amd64) cluster running K8S on bare-metal inc. router
* Educate myself on Ansible + K8S + GitOps for CI/CD/PD from bottom to top
* Refresh knowledge regarding networking and Python
* Enhanced PHP/SF4 stack for K8S supporting HPA, progressive deployments and a/b testing
* Advanced networking and firewalling to use one node as router *and* k8s master

## Tasks

### Phase 0: Hardware

- [x] Wire up three gigabyte bace 3160 and accessories

![alt text](https://raw.githubusercontent.com/helmuthva/ceil/max/doc/assets/max.jpg "Max Stack")

### Phase 1: Foundation

- [x] Central CloudOps entrypoint is `make`
- [x] Setup and teardown of all steps individually
- [x] Setup and teardown in one step
- [x] Setup of k8s cluster on amd64 using Ansible inc. weave networking and k8s dashboard
- [x] Helm/tiller for additional deployments
- [x] Traefik as ingress inc. Traefik dashboard
- [x] busybox-http using Traefik as ingress for demos
- [x] Grafana and prometheus
- [x] Minio for S3 compatible storage

### Phase 2: Storage and Loadbalancing

- [x] Dynamic volume provisioning using Heketi + GlusterFS spanning thumb drives
- [x] Enabled persistence for grafana and prometheus
- [x] MetalLB as LoadBalancer service

### Phase 3: Router

- [x] Act as DHCP client using dhcpcd
- [x] Act as DHCP & DNS server for K8S subnet using dnsmasq
- [x] Act as gateway from wlan0 (WiFi) to eth0 (K8S subnet) using iptables
- [x] Act as VPN server using OpenVPN
- [x] Dynamically update domain vpn.maxxx.pro (or similar) using ddclient and Cloudflare v4 API
- [x] Raise Firewall using ufw
- [x] Act as Docker registry mirror using official docker image `registry:2`
- [x] Act as private Docker registry
- [x] Act as http(s) proxy from router to k8s ingress for public access
- [x] Act as backup server for Docker and k8s volumes using borgbackup
- [ ] kail and harbor

### Phase 4: PiWatch

- [x] Deploy kubewatch to push K8S events to arbitrary webhook
- [ ] Configure to push to PiTraffic of `ceil` cluster (RPi)

### Phase 5: Magento and Wordpress (separate repos)

- [x] Deploy production ready Magento and Wordpress stacks
- [x] Automate development workflows using Skaffold  

### Phase 6: PiPHP (separate repo)

- [ ] Deploy amd64 version of PiPHP
- [ ] Automate dev-workflow using Skaffold  

### Phase 7: Auto-Scaling
- [ ] Autoscaling using HPA and custom metrics
- [ ] Zero-Scaling using Osiris
- [ ] Relevant dashboards in grafana

### Phase 8: Mesh-Networking
- [ ] Istio for Mesh-Networking
- [ ] Visibility tools
- [ ] Additional tools

### Phase 9: GitOps and Progressive Delivery

- [ ] Flagger for Helm using mesh network
- [ ] Canary deployments using mesh network
- [ ] ...

### Phase 10: CI and emphemeral test environments
- [x] Setup CI using JenkinsX
- [ ] ...

### Phase 11: A/B testing

- [ ] Using mesh network
- [ ] ...

### Phase 12: Sharing is caring

- [x] Open source under GPlv3
- [x] Links to useful material for further studies
- [ ] Refactor, esp. move roles into separate repos for reuse, use ansible vault etc.
- [ ] Prepare interactive install script automating the step to manually copy and edit `.tpl` files
- [ ] GitHub Page
- [ ] Write a series of blog posts
- [ ] Prepare a workshop presentation
- [ ] Educate peers in meetups

### Phase 13: Evaluation
- [ ] Evaluate with commercial projects in development


## Layers and tools

* CloudOps
  * Workstation: MacBook Pro
  * Package manager: Homebrew
  * Entrypoints: `make` and `kubectl` (GitOps in second step)
* Hardware
  * Mini PCs: 3x Gigabyte bace 3160 with 8GB SO-DIMM
  * Storage: 3x 128GiB SSDs (OS + containers) + 1x 128GiB USB ThumbDrive (Volumes for Docker registries on router) + 3x 128GiB USB ThumbDrives (GlusterFS) 
  * Networking: 5-port GBit/s switch + WiFi router connected to router
* Software
  * OS: Debian Stretch
  * Networking for router: iptables, dhcpcd, dnsmasq, OpenVPN, ddclient, CloudFlare
  * Configuration management: Ansible
  * Orchestration: Kubernetes (K8S)
  * K8S installation: `kadm`
  * Networking: weave
  * Persistence: GlusterFS + Heketi for dynamic volume provisioning
  * Ingresss: Traefik
  * Loadbalancer: MetaLB
  * Deployments: helm
  * Monitoring and Dashboarding: prometheus, grafana

## Install this repository

1) Fork this repository and clone to your workstation
2) Walk all files with suffix `.tpl`, create a copy in the same directory without said suffix and enter specifics where invited by capital letters

## Provision Mini PCs

1) Manual provisioning of base OS on servers by installing Debian Stretch from ISO on USB thumbs
2) Manual provisioning of wifi connectivity on `max-one` for bootstrapping using wpa_supplicant etc.

Notes:
- Tool for flashing ISO image on USB thumbs: https://www.balena.io/etcher/
- Debian ISO image with (non-free) drivers for Intel based wifi chip: http://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/9.9.0+nonfree/amd64/iso-cd/

## Setup router

1) Make a DHCP reservation for `max-one` in your home or company WiFi router with IP address `192.168.0.111` -  it will register as `max-one` at your WiFi router
2) Set up a static route to the k8s subnet `12.0.0.0` with `192.168.0.111` as gateway in your company or home wifi router - if this is not achievable use `make workstation-route-add` to add a route on your workstation.
3) For VPN setup port forwarding (sometimes called "virtual server") in your company or home wifi router for port `1194` (or whatever you configured in  `router/roles/vpn/defaults/main.yml`) to `192.168.0.111`
4) For http(s) proxy to k8s ingress using HAProxy setup port forwarding (sometimes called "virtual server") in your company or home wifi router for ports `80` and `443` to `192.168.0.111`
5) Add `192.168.0.111` as the second nameserver for the (WiFi) connection of your workstation using system settings
6) Reboot `max-one` to pickup its IP address via `make router-reboot` - it will register via ZeroConf/Avahi on your workstation as `max-one.local`
7) Check via `make router-check-ip` if the IP address has been picked up
8) Setup networking services on router using `make router-setup`
9) Wait for 1 minute than check if the k8s nodes (`max-{one,two,three}.dev`) have picked up their designated IP addresses from the router in the range `12.0.0.101` to `12.0.0.103`:  `make k8s-check-ip` 

Notes:
- Danger: wipes thumb drive in router
- It might take some time until the Zeroconf/Avahi distributed the name `max-one.local` in your network. You can check by ssh'ing into the router via `make router-ssh`
- The router will manage / route to the subnet `12.0.0.[0-128]` (`12/25`) the K8S nodes will life in and act as their DHCP and DNS server
- The router updates the IP address of `router.maxxx.pro` via DDNS
- Furthermore the router acts as an OpenVPN server
- After setting up the router wait for a minute to check if the k8s nodes have picked up the designated IPs using `make k8s-check-ip`
- After the k8s nodes picked up their IP addresses you can ssh into them using `make {one,two,three}-ssh`
- If on your workstation `nslookup max-{one,two,three}.dev` works but `ping max-{one,two,three}.dev` does not, reestablish the (WiFi) connection of your workstation
- If you want to play with the traffic lights mounted on top of the router: `make router-traffic`
- The last step of the router setup is building [PiWatch](https://github.com/helmuthva/piwatch) which takes ca. 15 minutes for the 1st build
- Last but not least the router provides a docker registry mirror and private docker registry consumed by the K8S nodes

## Setup K8S and execute all deployments

1) Execute `make setup` to setup K8S inc. persistence and deploy everything at once - takes ca. 45 minutes. 

Notes:
- `max-one` is set up as k8s master
- Danger: wipes thumb drives for setting up GlusterFS.

Alternatively you can execute the setup and deploy steps one-by-one as described below

## Interact, open dashboards and UIs

1) Establish proxy to cluster (leave open in separate terminal): `make k8s-proxy` 
2) List nodes: `make nodes-show`
3) List pods: `make pods-show`
4) Generate bearer token for accessing K8S dashboard: `make  k8s-dashboard-bearer-token-show`
5) Access K8S dashboard in your browser and enter token: `make k8s-dashboard-open`
6) Open Traefik UI in your browser: `make traefik-ui-open`
8) Show webpage in your browser: `make httpd-open`
8) Open Prometheus UI in your browser: `make prometheus-open`
9) Open Grafana dashboards in your browser: `make grafana-open`

## Setup K8S inc. persistence and helm/tiller

1) Wipe thumb drives for GlusterFS using `make thumb-wipe`
2) Setup K8S cluster inc. persistence via GlusterFS+Heketi and helm/tiller for later deployments: `make k8s-setup`. 

Notes:
- `max-one` is set up as k8s master

## Deploy

1) Execute all deployments using `make all-deploy` or deploy step by step as documented below.
2) Interact, open dashboards and UIs as documented above.

## Delete deployments

1) All deployments provide an individual make target for deleting the deployment, e.g. `ngrok-delete`. Execute `make help` to see all commands.
2) Execute `make all-delete` to delete all deployments at once

## Remove K8S inc. persistence and helm/tiller

1) Execute `make k8s-remove`.

## Teardown

1) Execute `make teardown` to delete all deployments and remove K8S.

## Obstacles 

* Examples for setting up K8S on bare metal mostly outdated and/or incomplete or making undocumented assumptions or not using Ansible correctly => full rewrite
* RBAC is rather new and not yet accounted for in deployment procedures of all tools and services => amend
* Most ansible playbooks do not provide a teardown role => build yourself

## Additional references

* https://github.com/luxas/kubeadm-workshop (custom autoscaling, by luxas)
* https://luxaslabs.com/ (slides by luxas)
* https://medium.com/vescloud/kubernetes-storage-performance-comparison-9e993cb27271 (Kubernetes Storage Performance Benchmark)
* https://medium.com/@carlosedp/multiple-traefik-ingresses-with-letsencrypt-https-certificates-on-kubernetes-b590550280cf (traefik,let's encrypt)
* https://stefanprodan.com/2018/expose-kubernetes-services-over-http-with-ngrok/ (ngrok, k8s)

## Notes

* Password file for vault must reside in the root of this project, named .ansible.password
* Encrypt single variable: `ansible-vault encrypt_string  --vault-password-file .ansible.password 'SECRET'`
* Decrypt single variable: `yq read YAML_FILE 'YAML_PATH' | ansible-vault decrypt --vault-password-file .ansible.password`