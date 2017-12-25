### Auth0Proxy
A general-purpose docker image for creating an auth0-based reverse proxy, based on apache and openidc

# What does it do:
This image allows quick and easy auth-enabling of any webapp.

# Setting up:
All settings are controlled by env vars, 
see the docker-compose file in this repo for an example of setting these up.   
Make sure your auth0 client allows callbacks from `https://<container_and_port>/auth/`

# SSL
Authproxy will auto-generate self-signed certificates and use these. This can be overridden by supplying the two `CERT`   
environment variables described below. This allows you to inject "real" certificates instead of using the auto-generated ones.


Envvars you need, in this example my container will be accessed on `https://192.168.99.100:5000/`
(the keys/secrets are just examples, so hack away)
```yaml
auth0domain: stuff.eu.auth0.com #my auth0 domain
auth0clientid: oQPvf7aVhKrX345345qQA40BeOim5ic #auth0client id
#Remember to add this url in auth0's list of callback urls and append /auth/, for example https://192.168.99.100:5000/auth/
auth0clientsecret: faqMDfef7pg6h6324598345U_v01234312488XJ1-jdREzyxqerOh #auth0client secret
thiscontainerurl: https://192.168.99.100:5000/ #Since I'm publishing to port 5000, this will be the url to hit in my browser. Remember the trailing slash
proxyto: http://www.vg.no/ #The site I want to forward requests to. Remember the trailing slash
CERT_FILE_PATH: the path of the ssl certifikate See the above section regarding certs!!
CERT_KEY_PATH: the path of the ssl cert key. See the above section regarding certs!!
OIDC_SESSION_TIMEOUT_SECONDS: defaults to 300. See https://github.com/pingidentity/mod_auth_openidc/blob/master/auth_openidc.conf#L443
```

# Known bugs
I'm struggling to get stdout from the apache process to be emitted.