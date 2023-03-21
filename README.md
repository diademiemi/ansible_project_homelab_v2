# Ansible project for my homelab setup.

This repository is part of my homelab_v2 setup, please check the following repository for more information: https://github.com/diademiemi/homelab_v2  

This project deploys Libvirt on the dedicated machines and installs K3S on the virtual machines.  

It includes tasks to deploy ArgoCD and configure it with KSOPS to use age secret encryption. The playbook `playbooks/04-patch-argocd-ksops.yml` automates the procedure to patch ArgoCD to use KSOPS for secret decryption.  

## [Project Structure](#project-structure)

File/Directory | Purpose
--- | ---
`inventory/` | Ansible inventories
`inventory/main/hosts.yml` | Ansible inventory file for my hosts
`playbooks/` | Ansible playbooks
`roles/` | Ansible roles
`ansible.cfg` | Ansible configuration file
`Vagrantfile` | Vagrant virtual machines setup file

### [Ansible](#ansible)
First, you should install the roles required for the playbooks.  

```bash
ansible-galaxy install -r roles/requirements.yml
```

We are assuming the Vagrant inventory is used. Please replace the `inventory/vagrant`  with the inventory you are using.  

Install libvirt on the dedicated machines with:  
```bash
ansible-playbook -i inventory/vagrant playbooks/01-libvirt.yml
```

Continue to the Terraform project to provision the virtual machines: https://github.com/diademiemi/terraform_homelab_v2  

Once the virtual machines are provisioned, you can continue with the Ansible project.

Prepare the virtual machines with:  
```bash
ansible-playbook -i inventory/vagrant playbooks/02-prepare.yml
```
Please set up ZFS volumes now if the hosts use ZFS.  

Install K3S on the virtual machines with:  
```bash
ansible-playbook -i inventory/vagrant playbooks/03-k3s-cluster.yml
```
This playbook adds a cron to uncordon nodes. I have a Home Assistant automation to safely drain and shutdown the local node to save power when it is not in use.  

Install ArgoCD on the K3S clusters with:  
```bash
ansible-playbook -i inventory/vagrant playbooks/04-deploy-argocd.yml
```


#### [ArgoCD Web UI](#argocd-web-ui)
On your local machine, you should retrieve the Kubeconfig for the cluster.  

Replace `local_vm` with the hostname of a k3s server.  

```bash
scp local_vm:.kube/config kubeconfig.yaml
```
This will save the kubeconfig to `./kubeconfig.yaml`.

You can access the ArgoCD web UI by retrieving the password with:  
```bash
export KUBECONFIG=${PWD}/kubeconfig.yaml
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```
You can port-forward the ArgoCD web UI with:
```bash
export KUBECONFIG=${PWD}/kubeconfig.yaml
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
The ArgoCD web UI is now available at [https://localhost:8080](https://localhost:8080). Accept the self-signed certificate and log in with "admin" as username and the password retrieved above.  

#### [ArgoCD KSOPS](#argocd-ksops)
Before adding the project to ArgoCD, you need to enable KSOPS. Secrets are encrypted in the repository and you need to provide the private key to decrypt them.  

To do this, you need to install KSOPS into ArgoCD, this project contains a playbook to do this.  
`ansible-playbook -i inventory/vagrant playbooks/enable-ksops.yml`   

This will prompt for the private age key to decrypt the KSOPS secret. Use the private key of the age keypair you used to encrypt the secrets.  
