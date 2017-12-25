# Auth0

This page shows how to configure an Auth0 client for use with Kubernetes. You need an existing auth0 account, which you can get for free using Auth0's "development" pricing tier. You also need a user or two, which you can either add manually into Auth0 using a "Database" connection, or (much cooler) use one of their gazillion integrations. I have enable the "Google" integration in mine, which allows me to log in with my gmail account.

We'll use Auth0 groups to control access into our cluster. In an "enterprise" context these groups will likely come thru an ADFS integration or similar, but we'll simply add a group manually.

If you're using an integration, hit the "try" icon to test login, as this will create the auth0 user so that we can add groups to it:

![Screenshot from 2017-12-25 19-05-41.png](https://user-images.githubusercontent.com/1747120/34342134-3a092c02-e9a7-11e7-81fd-dcee6ad91a32.png)

After that's done, you should have (at least) one user in Auth0's "users" view:

Click your user and scroll down to the "metadata" field. This is where we'll add a group we can use for Kubernetes RBAC:

![screenshot from 2017-12-25 19-13-12](https://user-images.githubusercontent.com/1747120/34342163-d20f329e-e9a7-11e7-9a99-b60f0b34711f.png)

Add the following to `app_metadata` (remember to hit Save afterwards):   
```json
{
  "authorization": {
    "groups": [
      "KubernetesAdmins"
    ],
    "roles": [],
    "permissions": []
  }
}
```

We also need to set up a rule. This will be a super-simple rule that will expose the `groups` list to a "top-level" claim, which makes it easy for us to configure group-level access on Kubernetes' apiserver.

Still in auth0 to to "Rules", Select "Create Rule" and start with an empty one.

Paste in this code to the rule and call it "Kubernetes-Group-Claims" or something with similar sazz:

```javascript
function (user, context, callback) {
  //console.log(user.authorization.groups);
  context.idToken.groups = user.authorization.groups;

  callback(null, user, context);
}
```

Now it's time to create an Auth0 Client which will represent Kubernetes. Go to "Clients", "Create Client", call it "Kubernetes" for instance, and give it the type "Regular Web Applications".

There are a couple of the items we'll need to fill in after we've provisioned Kubernetes, but take note of the following from the "Settings" view:
![screenshot from 2017-12-25 19-22-17](https://user-images.githubusercontent.com/1747120/34342221-f8576682-e9a8-11e7-9588-922ac2f0d341.png)

- Domain
- Client ID
- Client Secret

Under "Allowed Callback URLs", for now go ahead and add the following:
`https://openidconnect.net/callback`

Under advanced settings -> OAuth, make sure your settings are as follows:
![screenshot from 2017-12-25 19-24-16](https://user-images.githubusercontent.com/1747120/34342224-3c5f5812-e9a9-11e7-910d-eb0f38a025d0.png)

JWT Signature Altorithm: RS256   
OIDC Conformant: Off

(Kubernetes only supports RS256, so it's super-important to get that configured.)


We can now test that we got everything right by using the OpenID Connect debugger. Browse to `https://openidconnect.net/`. Click the "Configuration" link, and fill in your `domain` from you Auth0 client settings, then click "use auth discovery document". Also fill in the client ID and Client secret. Mine looks like this:
![screenshot from 2017-12-25 19-28-45](https://user-images.githubusercontent.com/1747120/34342246-e101e72c-e9a9-11e7-9bcc-0d96f0207ff5.png)
Hit "save" and then "Start". You should be prompted with a login box. Follow the "wizard" (basically hit "exchange", "next" and "verify" until you end up at the bottom of the page with a fully parsed jwt token). Mine looks like this:

![screenshot from 2017-12-25 19-30-59](https://user-images.githubusercontent.com/1747120/34342256-364b95fc-e9aa-11e7-8762-b59488adaa65.png)

From the metadata you can tell than I'm a dude, and that I'm a Kubernetes Admin. This proves that the rule we set up did its job, and exposes the "groups" object as a "top-level-thing" in the decoded jwt token data.

This is all we need for now as far as Auth0 goes. We'll head back here after we've gotten our Kubernetes cluster up and running, so head onwards to step 2!