# When IDaaS Providers Make Sense

API-driven authentication providers bail us out of the ordeal of having to set up a full-fledged authentication and authorization system on our own. User systems are highly dynamic in that they change frequently, and are also the biggest target for malicious agents. Building our own user system is not only complex and time consuming, but can also lead to security problems now and in the future. By using an IDaaS provider, we can cut down on  development time and trouble, while also ensuring our system is built and secured by industry experts.

When we're looking for a simple drop-in solution that allows users to sign up and log into our application in the least amount of time, both Okta and Auth0 have solutions available. Beyond the simplest use cases, both allow for multiple authentication strategies through an SDK you can customize.

![][compare-logins]

### Drop-in Authentication Solutions

_Okta_ has two drop-in solutions available: their standard sign-in page, which is a hosted redirect authentication solution, and their [Sign-in Widget][okta-signin]. The sign-in widget allows for a well featured login solution that I found easy to integrate. You can customize the styling, and have something up and running fairly quickly.

The Okta Sign-in Widget is easy to add to any web app and can be included with the following code:

```html
<html>
<head>
  <title>Example Okta Sign-In Widget</title>
  <script src='https://ok1static.oktacdn.com/assets/js/sdk/okta-signin-widget/2.5.0/js/okta-sign-in.min.js'></script>
  <link href='https://ok1static.oktacdn.com/assets/js/sdk/okta-signin-widget/2.5.0/css/okta-sign-in.min.css' type='text/css' rel='stylesheet'>
  <link href='https://ok1static.oktacdn.com/assets/js/sdk/okta-signin-widget/2.5.0/css/okta-theme.css' type='text/css' rel='stylesheet'>
</head>
<body>
  <div id='okta-login-container'></div>
  <script>
    const orgUrl = 'https://$OKTA_PREVIEW_INSTANCE.oktapreview.com'
    const oktaSignIn = new OktaSignIn({ baseUrl: orgUrl })

    oktaSignIn.renderEl({ el: '#okta-login-container' }, function (res) {
      if (res.status ==='SUCCESS') {
        res.session.setCookieAndRedirect(orgUrl)
      }
    })
  </script>
</body>
</html>
```

Auth0 also has two drop-in solutions available. Firstly, the [Lock][auth0-lock] widget, which is similar to Okta’s sign-in widget, but allows for more customization options,such as the ability to add custom registration fields that will then be saved into the user's metadata, which is nice. Secondly, theAuth0widget comes with a central login page which allows you to set up a fully-featured user management solution with full interface customization.

The Auth0 Lock widget is also really easy to integrate, and can be included with the following code:

```html
<html>
<head>
  <title>Example Okta Sign-In Widget</title>
</head>
<body>
  <script src='https://cdn.auth0.com/js/lock/10.24.1/lock.min.js'></script>
  <script>
    const lock = new Auth0Lock('$AUTH0_LOCK_SECRET', 'rbin.eu.auth0.com', {
      auth: {
        redirectUrl: 'http://localhost:3000/auth/auth0/callback',
        responseType: 'code',
        params: {
          scope: 'openid email' // more about scopes: https://auth0.com/docs/scopes
        }
      }
    })
  </script>
  <button onclick='lock.show()'>Login</button>
</body>
</html>
```

Okta has [10 quick start guides][okta-docs], while Auth0 has a staggering [56 quick start guides][auth0-guides] and whole sample applications, both seemingly drowning us in tutorial choices to pick from. Thankfully, these vary by the kind of application we're building and the language, so the more the merrier!

### Time to First API Call with Okta

The Express sample tutorial for Okta in Node.js was simple enough to run through and didn't pose any problems. After registering a developer account with Okta and creating a Web application in the admin back-end, I took the Client ID and Secret and placed them into a JSON config file for my application:

```json
{
  "OKTA_ISSUER": "https://$OKTA_PREVIEW_INSTANCE.oktapreview.com/oauth2/default",
  "OKTA_CLIENT_ID": "$CLIENT_ID",
  "OKTA_CLIENT_SECRET": "$CLIENT_SECRET",
  "OKTA_REDIRECT_URL": "http://localhost:8080/authorization-code/callback"
}
```

One thing to note about this specific authentication flow with Okta is that they use a stateful user session cookie to store the authorization details.

```js
app.use(cookieParser())
app.use(session({
  secret: 'AlwaysOn',
  cookie: { maxAge: 3600000 },
  resave: false,
  saveUninitialized: false
}))

const oidc = new ExpressOIDC({
  issuer: sampleConfig.OKTA_ISSUER,
  client_id: sampleConfig.OKTA_CLIENT_ID,
  client_secret: sampleConfig.OKTA_CLIENT_SECRET,
  redirect_uri: sampleConfig.OKTA_REDIRECT_URL,
  scope: 'openid profile email'
})
```

With a simple setup and straight-forward tutorial, let's put the time taken from the start of our trial to the first useful API call being made at roughly 17 minutes. Not bad!

![][okta-signed-in]

### Time to First API Call with Auth0

Running through the Node.js quick start tutorial for Auth0 had me drop in their login page on a basic vanilla web application. After signing up for a free Auth0 developer account, I created a Standard Web Client in their management dashboard and copied the Client ID and Domain to my app's variables:

```json
{
  "AUTH0_CLIENT_ID": "$AUTH0_CLIENT_ID",
  "AUTH0_DOMAIN": "$AUTH0_ACCOUNT_DOMAIN.eu.auth0.com",
  "AUTH0_REDIRECT_URL": "http://localhost:3000"
}
```

The main code to include the Auth0 Web Auth module is pretty simple, as is storing the authorization details in local storage:

```js
const webAuth = new auth0.WebAuth({
  domain: sampleConfig.AUTH0_DOMAIN,
  clientID: sampleConfig.AUTH0_CLIENT_ID,
  redirectUri: location.href,
  audience: `https://${ sampleConfig.AUTH0_DOMAIN }/userinfo`,
  responseType: 'token id_token',
  scope: 'openid',
  leeway: 60
})

function setSession(authResult) {
  const expiresAt = JSON.stringify(
    new Date().valueOf() + authResult.expiresIn * 1000
  )

  localStorage.setItem('access_token', authResult.accessToken)
  localStorage.setItem('id_token', authResult.idToken)
  localStorage.setItem('expires_at', expiresAt)
}
```

Running the web application, we can see the session details being stored in local storage in the image below. The encoded JWT `id_token` is highlighted in red. We can also observe the session being destroyed on logout.

![][auth0-session]

Overall, the Auth0 time to first useful API call took just under 9 minutes. Yay! 🎉

# Development Experience

I also wanted to secure an API back-end that powered a single-page web application and had to figure out what OAuth flow to utilize. The [Client Credentials Grant][auth0-credentials-grant] seemed to make the most sense for a back-end API. Both _Auth0_ and _Okta_ offer multiple OAuth 2.0 authentication flows, including the server-to-server flow required to secure an API back-end.

From an API and documentation point of view, both providers are quite nice, and have fairly clear and concise documentation to help you pick the most appropriate flow, aiding with the implementation. In general, though, the _Auth0_ documentation is a bit nicer, with clear explanations and detailed diagrams. For instance, _Auth0_ has an entire page of documentation dedicated to [choosing an OAuth 2.0 flow][auth0-oauth-flow]. _Okta_ does have [a section on choosing flows][okta-oauth-flow], but it is a bit less detailed than the _Auth0_ page.

### Social Login Integrations

Now that I had the API back-end authentication taken care of using the above OAuth2.0 flow, I needed to add social logins to the application.

Naturally, both _Okta_ and _Auth0_ offer social login solutions. _Okta_ offers [an extensive Social Login solution][okta-social] which is feature-rich, well documented, and easy to implement. _Auth0_ also offers [an excellent Social Login solution][auth0-social] with integrations to many providers, and the extensibility options of adding your own [custom providers][auth0-social-custom].

In the case of my recent implementation, we required custom social login providers, and the ability to do so with Auth0 was a deal-breaker. With that, I had all of the authentication taken care of… Or so I thought! 😅

### Multi-Factor Authentication

Realizing that we were storing a fair amount of sensitive data in user profiles, and wishing for an extended level of security, we had to implement Multi-Factor Authentication. MFA allows for an additional layer of security, decreasing the likelihood of unauthorized access to user accounts. MFA is widely recommended for full usage across the web. Both providers have multiple options available for the multi-factor auth process, with Okta presenting more configuration settings.

I have previously written about [setting up 2FA on your own in Node.js][pf-setup-2fa], but given I was already going to use an IDaaS solution, it made much more sense to keep all authentication and authorization concerns in a single place. Both providers have the ability to integrate an MFA solution into our applications.

![][compare-mfa]

Options for Okta include: SMS, Google Authenticator, and several others. Although most cases would be covered here, Okta doesn’t allow for a custom solution to be included. The MFA options for Auth0 include: push notifications ([demo][auth0-mfa-demo]), SMS, Google Authenticator, and, crucially: contextual MFA through rules or custom providers.

### User Management API

Lastly, I needed a simple interface to manage users on the web. I built this using user management REST API commands. _Auth0_ and _Okta_ each provide a Management API for administrative tasks. For instance, if we wish to update a user's profile through the REST API, we can do so with the following API call with _Okta_:

```bash
curl \
  -X POST \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -H "Authorization: SSWS $API_TOKEN" \
  -d '{
    "profile": {
      "firstName": "Vincent",
      "lastName": "Adultman",
      "email": "vincent.adultman@business.com",
      "login": "vincent.adultman@transactions.business.com",
      "mobilePhone": "555-415-1337"
    }
  }' "https://$OKTA_DOMAIN.com/api/v1/users/$USER_ID"
```

And with _Auth0_:

```bash
curl \
  -X PATCH \
  -H "Accept: application/json" \
  -H "Content-Type: application/json"  \
  -d '{
    "Blocked": false,
    "Email_verified": true,
    "Email": "vincent.adultman@business.com",
    "user_metadata":{
      "profileCode": 1479,
      "addresses": {
        "work_address": "100 Industrial Way",
        "home_address": "742 Evergreen Terrace"
      }
    },
    "App_metadata": {},
    "Connection": "business-db",
    "Username": "vincent_adultman",
    "Client_id": "DaM8bokEXBWrTUFCiJjWn50jei6ardyX"
  }' "https://login.auth0.com/api/v2/users/$USER_ID"
```

Both API calls are similar, although Auth0 provides better handling of custom profile fields, allowing these to be created and modified on the fly. With Okta, custom profile fields can only be specified if first created through either the Admin UI, or the Schemas API, as per the [Okta documentation][okta-custom-profiles]:

> User profiles may be extended with custom properties but the property must first be added to the user profile schema before it can be referenced. You can use the Profile Editor in the Admin UI or the Schemas API to manage schema extensions.

Although you must first define profile fields, this does lead to an advantage where Okta allows for granular event logging and tracking. You can access analytics on a profile attribute level, which is useful in scenarios when you must have a log of every change made to individual fields at any time, such as for a financial, or gambling application. Auth0 cannot track updates at this level as their user profile objects have a flexible schema.

One other point to note with Okta here is the usage of the `PUT` HTTP verb when updating a user profile. If you were to use the `PUT` verb, any unspecified properties would be overridden with `null` values. From the [Okta docs][okta-custom-profiles-put-docs]:

> Note: Use the POST method to make a partial update and the PUT method to delete unspecified properties.

As far as I'm concerned this is a pretty weird API to offer, since virtually nobody would expect this behavior, and it may lead to data loss if we don't test properly and mistakenly use PUT thinking it'd just update the provided fields.

### Extensibility

Okta has a large [list of extensions and integrations][okta-integrations]. They also include the ability to create your own integrations, albeit the options are slightly limited. You can read up on [how to create Okta integrations here][okta-create-integration].

Auth0 has a strong focus on extensibility and offers a platform on which to build extensions and integrations beyond just webhooks: [Auth0 Extend][auth0-extend]. With a focus on a [serverless back-end][auth0-extend-serverless], _Extend_ helps create and deploy extensions in very little time. The extensibility Auth0 provides was a blessing, as was using [Webtask][auth0-webtask] as a modern replacement for cron jobs.

### OpenID and SAML

OpenID Connect (OIDC) is an authentication protocol based on the OAuth 2.0 family of specifications. It uses simple JWT tokens delivered via the OAuth protocol, and its purpose is to enable you to use one login across multiple sites. Again, both providers offer a solution for this, ultimately receiving authorization requests and sending back ID and access tokens.

To get started with Okta's OIDC solution, check [their documentation here][okta-oidc]. To get started with Auth0's OIDC solution, [read their documentation here][auth0-oidc]. Both providers have similar solutions in terms of both functionality and usage.

Both _Okta_ and _Auth0_ offer integrations for SAML as well. Both platforms can be used as both your Service Provider and your Identity Provider. Although both platforms offer similar solutions for this scenario, Auth0 has fewer options in their catalog. Okta has a lot more SAML integration options available, and therefore are the choice provider for this scenario. Okta also offers [a SAML testing tool][okta-saml-testing], which is pretty nifty.

That being said, I did find it easier to navigate the Auth0 resources for information on SAML. Auth0 has [extensive SAML documentation][auth0-saml] whereas Okta has a more sparse [SAML documentation][okta-saml].

# Documentation and Resources

The first step when researching a new product is usually reading its documentation. This leaves API providers with a distinct need to provide excellent, detailed, and concise documentation.

One of the first features of both providers' documentation I would like to mention is the ability with both to export the API documentation to the [Postman REST Client][postman]. Both _Okta_ and _Auth0_ allow developers to test out the API functionality in real-time by installing preset collections in Postman. With _Okta_, you can get an overview of the [Postman testing integration][okta-postman]. For _Auth0_, you can read about using the [Auth0 API with Postman][auth0-postman].

Both the Okta and Auth0 API reference documentations are pretty comprehensive. One of the best examples of [good API documentation][stripe-docs] belongs to Stripe, the payment API provider. While Auth0does provide copy/paste samples in a few formats and Oktadon't really do that, Stripe has nailed the concept, allowing developers to find snippets of API calls that they can copy directly in a multitude of languages, and which actually work without the need for any modifications whatsoever.

# Conclusion

I set out to explore the functionality of two of the leading IDaaS solutions, and in the process I've also gained a deeper understanding of how both services work and what they offer.

It's clear to me that Auth0 is the more developer-friendly choice, with gorgeous design around their products, which is always something the front-end developer in me seeks and naturally gravitates towards. The Auth0 offering was clearly conceived from scratch to support a wide range of scenarios and use cases, while Okta made me work harder to get integrations going. Overall, both solutions mostly get the job done, but I prefer Auth0 for its elegance and simplicity, if not completeness.

[compare-logins]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/image5-f050159b4c8d441abea707d1d774f3ea.png
[compare-mfa]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/image4-1531476595054d7aa19edc76d04f97dd.png
[okta]: https://www.okta.com/
[okta-signin]: https://developer.okta.com/quickstart/#/widget/nodejs/express
[okta-docs]: https://developer.okta.com/documentation/
[okta-signed-in]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/image2-a432bb64dcb64b10beb09e90b2e8ed55-ac153811ef684452924a60d0af01b0cb.png
[okta-oauth-flow]: https://developer.okta.com/authentication-guide/auth-overview/#choosing-an-oauth-20-flow
[okta-social]: https://developer.okta.com/authentication-guide/social-login/
[okta-custom-profiles]: https://developer.okta.com/docs/api/resources/users#custom-profile-properties
[okta-custom-profiles-put-docs]: https://developer.okta.com/docs/api/resources/users#update-profile
[okta-integrations]: https://www.okta.com/resources/find-your-apps/
[okta-create-integration]: https://developer.okta.com/integrate-with-okta/
[okta-oidc]: https://developer.okta.com/standards/OIDC/
[okta-postman]: https://developer.okta.com/docs/api/getting_started/api_test_client.html
[okta-saml]: https://developer.okta.com/authentication-guide/saml-login/
[okta-saml-testing]: http://saml.oktadev.com/
[auth0]: https://auth0.com/
[auth0-lock]: https://auth0.com/docs/libraries/lock/v10
[auth0-guides]: https://auth0.com/docs/quickstarts
[auth0-session]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/image1-a251ef120b0143349ec4955628c8f379.png
[auth0-credentials-grant]: https://auth0.com/docs/api-auth/grant/client-credentials
[auth0-oauth-flow]: https://auth0.com/docs/api-auth/which-oauth-flow-to-use
[auth0-social]: https://auth0.com/docs/identityproviders#social
[auth0-social-custom]: https://auth0.com/docs/extensions/custom-social-extensions
[auth0-mfa-demo]: https://auth0.com/docs/multifactor-authentication/guardian
[auth0-mfa-rules]: https://auth0.com/docs/rules/current
[auth0-mfa-custom]: https://auth0.com/docs/multifactor-authentication/custom
[auth0-extend]: https://auth0.com/extend/docs/#platform-extensibility
[auth0-extend-serverless]: https://auth0.com/extend/developers
[auth0-webtask]: https://webtask.io/
[auth0-postman]: https://auth0.com/docs/api/postman
[auth0-oidc]: https://auth0.com/docs/protocols/oidc
[auth0-saml]: https://auth0.com/docs/protocols/saml
[pf-setup-2fa]: /articles/setting-up-2fa-for-nodejs-applications "Setting up 2FA for Node.js Applications on Pony Foo"
[postman]: https://www.getpostman.com/
[stripe-docs]: https://stripe.com/docs
