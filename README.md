# Kubernetes-Auth0

This repo contains the things needed to set up Auth0 as the authentication/authorization backend for Kubernetes, using OpenID Connect.

After following this repo you should be able to:
- Use the Kubernetes Dashboard with authentication pass-thru
- Authenticate to kubectl using id tokens

### Known issues
- Kubernetes supports auto-updating creds using refresh tokens, but I haven't gotten that to work yet. This means that you'll have to update the token in your `kubeconfig` file when your current token expires in order for `kubectl` to keep working
- Proper logout/session handling isn't implemented

### Requirements
- This documentation includes a full Kubernetes deployment using [Kubelini](https://github.com/trondhindenes/Kubelini), so all you need are two ubuntu 16.04 vms which you can access using ssh. The two vms need to be able to reach each other (so essentially on the same network).
- You need an active auth0 subscription, a free one is more than enough. You also need a user in auth0, for example by activating the Google integration, and using your gmail user. Or something.
- If you deploy your Kubernetes cluster using Kubelini, you need access to an S3 bucket.

### Do the things
[1. Make the Auth0](docs/01-auth0.md)   
[2. Make the Kubernetes](docs/02-kubernetes.md)   
[3. Deploy `kubernetes-dashboard` and the `mod_oidc` proxy](docs/03-dashboard.md)   
[4. Test OpenID Connect tokens with kubectl](docs/04-tokens.md)
