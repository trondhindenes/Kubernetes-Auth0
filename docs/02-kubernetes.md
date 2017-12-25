# Kubernetes setup

Disclaimer: I wrote Kubelini because I wanted a lightweight way of provisioning Kubernetes things. You don't have to use it, this guide should work with minikube and any other kubernetes installer for that matter. All you need is a Kubernetes "flavor" that allows you to send custom attributes to the kube-apiserver. Minikube for instance, lets you do that like this:

```bash
minikube start \
      --extra-config=apiserver.Authorization.Mode=RBAC \
      --extra-config=apiserver.Authentication.OIDC.IssuerURL=https://myauth0domain.auth.com/ \
      --extra-config=apiserver.Authentication.OIDC.UsernameClaim=email \
      --extra-config=apiserver.Authentication.OIDC.ClientID="wakkawakka"
``` 

The flags you need to pass are:
`authorization-mode`: Set this to either `Node,RBAC` or `RBAC`. In short, you need `RBAC`.
`oidc-issuer-url`: This should be set to the value of the `Domain` in your Auth0 client settings. You MUST add a trailing slash. TRAILING. SLASH.
`oidc-username-claim`: Should be set to `email`
`oidc-groups-claim`: Should be set to `groups`. This is why we create the special rule in step 1, so that we have a `groups` object in the jwt data that Kubernetes inspects.
`oidc-client-id`: Should be set to the value of the `Client ID` from your Auth0 client settings.

Here's a snippet copied from the service definition on one of my Kubernetes master nodes (note the trailing slash!):
![screenshot from 2017-12-25 19-42-34](https://user-images.githubusercontent.com/1747120/34342295-ce5f1f16-e9ab-11e7-9521-26e5cdd1d3ae.png)

So if you have an existing cluster the above should be enough for you to set up the things. If not, follow the following steps to set up Kubernetes using Kubelini. You need to have 2 (Ubuntu 16.04) vms available, and you also need [Ansible](https://github.com/ansible/ansible) installed.

1. Clone https://github.com/trondhindenes/Kubelini and cd into it
2. Add a file called `hosts` to the dir you're in. Set the contents of the file to:
```ini
[all]
[all:vars]
#ansible_ssh_private_key_file=mykey.pem
ansible_user=ubuntu
ansible_password=MySuperPass
ansible_become_pass=MySuperPass
become=true

[kubernetes_master]
master1 ansible_host=192.168.218.131

[kubernetes_worker]
worker1 ansible_host=192.168.218.130
```

Replace the ip addresses with the addresses of your two vms. You can add as many as you like, but each vm should be either in the "master" or "worker" group.

edit the `group_vars/all.yml` file so that it contains the following (this is where you'll add some stuff from your Auth0 client, so make sure you get all the vars right!):
```yaml
---
kubernetes_version: 1.9.0

s3_sync_bucket: kubelinidev
cluster_ip_range: 10.32.0.0/16
cluster_ip_range_first_ip: 10.32.0.1
cluster_dns_address: 10.32.0.10
cluster_cidr: 10.200.0.0/16
cluster_domain: cluster.local

kubernetes_cluster_address: #Set this to the ip address of the first node in the "masters" group in your hosts file
allow_swap: yes


apiserver_oidc_enable: yes
apiserver_oidc_issuer_url: #Set this to the auth0 domain, for example https://trondhindenes.eu.auth0.com/. Remember the trailing slash!
apiserver_oidc_username_claim: email
apiserver_oidc_client_id: #Client id from your auth0 client!
apiserver_oidc_groups_claim: groups

networking_stack: flannel
weavenet_allocation_cidr: 10.200.0.0/16
flannel_allocation_cidr:  10.200.0.0/16
additional_kubernetes_static_users:
```

You also need to fill the `secrets/secrets.yml` according to the Kubelini documentation. This is used to access an S3 bucket used for certificate exchange during Kubernetes deployment.

After you've done this, and still in the "root" of the kubelini repo, run the following:
`ansible-playbook prep_cluster.yml -i hosts`   
`ansible-playbook site.yml -i hosts`

To verify that everything worked, log on to the "master" kubernetes node, and run `kubectl get nodes`. It should list out as many nodes as you added to the "worker" group in your hosts file.

Make sure you have kubectl available locally and on your path.
Make sure to download the `admin/admin.kubeconfig` from the s3 bucket. This allows you to run kubectl against the cluster from your local session using an admin cert (so not using Auth0). You can test this out by running `kubectl get nodes --kubeconfig <path_to_admin.kubeconfig>`

You can also copy admin.kubeconfig to the path `$/.kube/config` so you don't have to set the `--kubeconfig` parameter everytime you run `kubectl` - or merge it in with your existing kubeconfig.

At this point, you should have a working Kubernetes cluster, configured both with Auth0 and an alternative "admin key" so you can access the cluster without using Auth0. The admin key / kubeconfig should be kept private to the Kubernetes admins, as it provides full access to the entire Kubernetes cluster.