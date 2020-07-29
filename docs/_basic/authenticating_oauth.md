---
title: Authenticating with OAuth
lang: en
slug: authenticating-oauth
order: 14
---

<div class="section-content">
Slack apps installed on multiple workspaces will need to implement OAuth, then store installation information (like access tokens) securely. By providing `clientId`, `clientSecret`, `stateSecret` and `scopes` when initializing `App`, Bolt for JavaScript will handle the work of setting up OAuth routes and verifying state. Your app only has built-in OAuth support when using the built-in ExpressReceiver. If you're implementing a custom receiver, you can make use of our [OAuth library](https://slack.dev/node-slack-sdk/oauth#slack-oauth), which is what Bolt for JavaScript uses under the hood.

Bolt for JavaScript will create a **Redirect URL** `slack/oauth_redirect`, which Slack uses to redirect users after they complete your app's installation flow. You will need to add this **Redirect URL** in your app configuration settings under **OAuth and Permissions**. This path can be configured in the `installerOptions` argument described below.

Bolt for JavaScript will also create a `slack/install` route, where you can find an `Add to Slack` button for your app to perform direct installs of your app. If you need any additional authorizations (user tokens) from users inside a team when your app is already installed or a reason to dynamically generate an install URL, manually instantiate an `ExpressReceiver`, assign the instance to a variable named `receiver`, and then call `receiver.installer.generateInstallUrl()`. Read more about `generateInstallUrl()` in the [OAuth docs](https://slack.dev/node-slack-sdk/oauth#generating-an-installation-url).

To learn more about the OAuth installation flow with Slack, [read the API documentation](https://api.slack.com/authentication/oauth-v2).
</div>

```javascript
const app = new App({
  signingSecret: process.env.SLACK_SIGNING_SECRET,
  clientId: process.env.SLACK_CLIENT_ID,
  clientSecret: process.env.SLACK_CLIENT_SECRET
  stateSecret: 'my-state-secret',
  scopes: ['channels:read', 'groups:read', 'channels:manage', 'chat:write', 'incoming-webhook'],
  installationStore: {
    storeInstallation: (installation) => {
      // change the line below so it saves to your database
      return database.set(installation.team.id, installation);
    },
    fetchInstallation: (InstallQuery) => {
      // change the line below so it fetches from your database
      return database.get(InstallQuery.teamId);
    },
  },
});
```

<details class="secondary-wrapper">
<summary class="section-head" markdown="0">
<h4 class="section-head">Customizing OAuth defaults</h4>
</summary>

<div class="secondary-content" markdown="0">
You can override the default OAuth using the `installerOptions` object, which can be passed in during the initialization of `App`. You can override the following:

- `authVersion`: Used to toggle between new Slack Apps and Classic Slack Apps
- `metadata`: Used to pass around session related information
- `installPath`: Override default path for "Add to Slack" button
- `redirectUriPath`: Override default redirect url path
- `callbackOptions`: Provide custom success and failure pages at the end of the OAuth flow
- `stateStore`: Provide a custom state store instead of using the built in `ClearStateStore`

</div>

```javascript
const app = new App({
  signingSecret: process.env.SLACK_SIGNING_SECRET,
  clientId: process.env.SLACK_CLIENT_ID,
  clientSecret: process.env.SLACK_CLIENT_SECRET
  scopes: ['channels:read', 'groups:read', 'channels:manage', 'chat:write', 'incoming-webhook'],
  installerOptions: {
      authVersion: 'v1', // default  is 'v2', 'v1' is used for classic slack apps
      metadata: 'some session data',
      installPath: '/slack/installApp',
      redirectUriPath: '/slack/redirect',
      callbackOptions: {
        success: (installation, installOptions, req, res) => {
          // Do custom success logic here
          res.send('successful!');
        }, 
        failure: (error, installOptions , req, res) => {
          // Do custom failure logic here
          res.send('failure');
        }
      },
      stateStore: {
        // Do not need to provide a `stateSecret` when passing in a stateStore
        // generateStateParam's first argument is the entire InstallUrlOptions object which was passed into generateInstallUrl method
        // the second argument is a date object
        // the method is expected to return a string representing the state
        generateStateParam: (installUrlOptions, date) => {
          // generate a random string to use as state in the URL
          const randomState = randomStringGenerator();
          // save installOptions to cache/db
          myDB.set(randomState, installUrlOptions);
          // return a state string that references saved options in DB
          return randomState;
        },
        // verifyStateParam's first argument is a date object and the second argument is a string representing the state
        // verifyStateParam is expected to return an object representing installUrlOptions
        verifyStateParam:  (date, state) => {
          // fetch saved installOptions from DB using state reference
          const installUrlOptions = myDB.get(randomState);
          return installUrlOptions;
        }
      },
  }
});
```

</details>
