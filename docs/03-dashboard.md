# Kubernetes-dashboard

In this section we'll get the Kubernetes Dashboard up and running. Kubernetes-Dashboard is a web app that provides a nice and simple gui for interacting with Kubernetes.

How this stuff works:
We'll set up an "Authentication Proxy" based on Apache2 and the `mod_oidc` project. When a client requests the dashboard URL, it hits the auth proxy. If the client isn't already authenticated, mod_oidc redirects the client to Auth0 for logon.

The "source code" for the auth0 proxy is included in the `kube_auth0_proxy` folder of this repo, and setup to auto-build on https://hub.docker.com/r/trondhindenes/kubernetes-auth0/ so you don't have to build it yourself. You'll find this image referenced in the deployment spec described in the following sections.

When the client is logged-in to Auth0, mod_oidc is able to pass an `Authorization` header containig the Auth0 `id_token` to the Kubernetes Dashboard. The Kubernetes Dashboard is really smart, and passes the Bearer token directly to the Kubernetes apiserver it's talking to, so that everything happens in the context of the logged-in user. Kubernetes uses the `Client Id` we already configured to verify the user, either using the user's email or the user's group membership.

All of this is contained in the folder `dashboard_deployment` in this repo. 
It's worth noting that you'll likely want to change the configuration a bit. You'll probably use either a Load balancer or an Ingress to expose the dashboard. However, I wanted a setup that doesn't make any assumptions, so I'm using a NodePort service type. This means that Kubernetes will assign an arbitrary port to the dashboard. This also means that we'll have to run things in the right order:

Start with setting up the service so that we'll get a port:
`kubectl apply -f dashboard_deployment/01_service.yml`
Then, figure out the port by running:
`kubectl --namespace kube-system get service`

As you can see, mine is running on port 32170 on my single Kubernetes host:
![screenshot from 2017-12-25 20-34-16](https://user-images.githubusercontent.com/1747120/34342527-1e50ec6e-e9b3-11e7-92f1-68f1819b871f.png)

At this point you need to go back to your Auth0 client settings and add the url to the list of allowed callback urls (in addition to the existing `openidconnect.net`. Make sure you add `https://<kubernetes_worker_ip>:<service_port>/`.)

Now we can edit the `02_deployment.yml` file in the same folder. Change the value of `thiscontainerurl` so that it points to: `https://<kubernetes_worker_ip>:<service_port>/` - again, remember the trailing slash. (The auth0 proxy needs to know "its own" url, so in a real-world scenario this wold probably be a url and not just an ip address). Also replace the values for the following environment variables in the same file:   
`auth0clientid`   
`auth0clientsecret`   
`auth0domain`   

Then deploy the thing:
`kubectl apply -f dashboard_deployment/02_deployment.yml`

Now if you access the Kubernetes node ip/service port in your browser, you _should_ be taken to an Auth0 login prompt and then redirected to the Kubernetes dashboard, which should look something like this:

![screenshot from 2017-12-25 20-46-15](https://user-images.githubusercontent.com/1747120/34342587-d58f0c98-e9b4-11e7-9570-ba3383361a53.png)

This means that everything is working - Kubernetes verified the user and figured out that it didn't have access to anything at all. This is totally expected, and caused by the last missing bit: A ClusterRole/ClusterRoleBinding which "ties" the Auth0 group we defined earlier to a set of Kubernetes permissions. We'll do that with the file `03_auth.yml`. Here we've set up a ClusterRole called `oidc-admin-role` which basically is a full Kubernetes Admin. Then we set up a ClusterRoleBinding which states that any member of the Auth0 group `KubernetesAdmins` can use that role.
Invoke it by running: `kubectl apply -f dashboard_deployment/03_auth.yml`.

If you refresh the Kubernetes dashboard, all access denied messages should be gone:
![screenshot from 2017-12-25 20-52-31](https://user-images.githubusercontent.com/1747120/34342602-9c32067a-e9b5-11e7-8bfd-24fa15fd1d82.png)

As a last test you can remove the `KubernetesAdmins` group from your Auth0 user, and open up a new browser to perform a "fresh" login session. You should be presented by the "Access Denied" message in the dashboard again.

So that's pretty much it. Kubernetes Dashboard is an awesome tool, especially for people who don't have a lot of Kubernetes expertise, and by following this guide you can provide developers etc with limited access to your cluster without sharing "root" credentials.