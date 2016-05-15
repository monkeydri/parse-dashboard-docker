# Parse Dashboard docker

* [Getting Started](#getting-started)
* [Local Installation](#local-installation)
  * [Configuring Parse Dashboard](#configuring-parse-dashboard)
  * [Managing Multiple Apps](#managing-multiple-apps)
  * [Other Configuration Options](#other-configuration-options)
* [Deploying Parse Dashboard](#deploying-parse-dashboard)
  * [Preparing for Deployment](#preparing-for-deployment)
  * [Security Considerations](#security-considerations)
    * [Configuring Basic Authentication](#configuring-basic-authentication)
    * [Separating App Access Based on User Identity](#separating-app-access-based-on-user-identity)
  * [Run with Docker](#run-with-docker)
* [Contributing](#contributing)


## run parse-dashboard with docker-compose


```sh
wget https://github.com/monkeydri/parse-dashboard-docker/raw/master/docker-compose.yml
APP_ID={appId} MASTER_KEY={masterKey} SERVER_URL={http://localhost:1337/parse} docker-compose up -d
```

## Configuring Parse Dashboard with json config file

You can also start the dashboard from the command line with a config file.  To do this, create a new file called `parse-dashboard-config.json` inside your local Parse Dashboard directory hierarchy.  The file should match the following format:

```json
{
  "apps": [
    {
      "serverURL": "http://localhost:1337/parse",
      "appId": "myAppId",
      "masterKey": "myMasterKey",
      "appName": "MyApp"
    }
  ]
}
```

You can then start the dashboard using docker-compose with file volume mounted from host into parse-dashboard container at pat /src/ParseDashboard/dashboard-config.json

```sh
wget https://github.com/monkeydri/parse-dashboard-docker/raw/master/docker-compose.yml
APP_ID={appId} MASTER_KEY={masterKey} SERVER_URL={http://localhost:1337/parse} docker-compose up -d
```

## Managing Multiple Apps

Managing multiple apps from the same dashboard is also possible.  Simply add additional entries into the `parse-dashboard-config.json` file's `"apps"` array.

You can manage self-hosted [Parse Server](https://github.com/ParsePlatform/parse-server) apps, *and* apps that are hosted on [Parse.com](http://parse.com/) from the same dashboard. In your config file, you will need to add the `restKey` and `javascriptKey` as well as the other paramaters, which you can find on `dashboard.parse.com`. Set the serverURL to `http://api.parse.com/1`:

```json
{
  "apps": [
    {
      "serverURL": "https://localhost:1337/v1",
      "appId": "myAppId1",
      "masterKey": "myMasterKey1",
      "javascriptKey": "myJavascriptKey",
      "restKey": "myRestKey",
      "appName": "My Parse Server App 1"
    },
    {
      "serverURL": "http://localhost:1338/v1", 
      "appId": "myAppId2",
      "masterKey": "myMasterKey2",
      "appName": "My Parse Server App 2"
    }
  ]
}
```

## App Icon Configuration

Parse Dashboard supports adding an optional icon for each app, so you can identify them easier in the list. To do so, you *must* use the configuration file, define an `iconsFolder` in it, and define the `iconName` parameter for each app (including the extension). The path of the `iconsFolder` is relative to the configuration file. To visualize what it means, in the following example `icons` is a directory located under the same directory as the configuration file:

```json
{
  "apps": [
    {
      "serverURL": "http://localhost:1337/parse",
      "appId": "myAppId",
      "masterKey": "myMasterKey",
      "appName": "My Parse Server App",
      "iconName": "MyAppIcon.png",
    }
  ],
  "iconsFolder": "icons"
}
```

## Other Configuration Options

You can set `appNameForURL` in the config file for each app to control the url of your app within the dashboard. This can make it easier to use bookmarks or share links on your dashboard. 

To change the app to production, simply set `production` to `true` in your config file. The default value is false if not specified.

# Deploying Parse Dashboard

## Preparing for Deployment

Make sure the server URLs for your apps can be accessed by your browser. If you are deploying the dashboard, then `localhost` urls will not work. URL in dashboard config must be the URL recheable from where the dashboard will be accessed from (web browser).

## Security Considerations
In order to securely deploy the dashboard without leaking your apps master key, you will need to use HTTPS and Basic Authentication. 

The deployed dashboard detects if you are using a secure connection. If you are deploying the dashboard behind a load balancer or proxy that does early SSL termination, then the app won't be able to detect that the connection is secure. In this case, you can start the dashboard with the `--allowInsecureHTTP=1` option. You will then be responsible for ensureing that your proxy or load balancer only allows HTTPS.

Default value for `allowInsecureHTTP` is 1 on parse-dashboard but if not set in command line when using docker-compose it defaults to an empty string which results in `allowInsecureHTTP` to be set to 0.

### Configuring Basic Authentication
You can configure your dashboard for Basic Authentication by adding usernames and passwords your `parse-dashboard-config.json` configuration file:

```json
{
  "apps": [{"...": "..."}],
  "users": [
    {
      "user":"user1",
      "pass":"pass"
    },
    {
      "user":"user2",
      "pass":"pass"
    }
  ]
}
```

### Separating App Access Based on User Identity
If you have configured your dashboard to manage multiple applications, you can restrict the management of apps based on user identity.

To do so, update your `parse-dashboard-config.json` configuration file to match the following format:

```json
{
  "apps": [{"...": "..."}],
  "users": [
     {
       "user":"user1",
       "pass":"pass1",
       "apps": [{"appId": "myAppId1"}, {"appId": "myAppId2"}]
     },
     {
       "user":"user2",
       "pass":"pass2",
       "apps": [{"appId": "myAppId1"}]
     }  ]
}
```
The effect of such a configuration is as follows:

When `user1` logs in, he/she will be able to manage `appId1` and `appId2` from the dashboard.

When *`user2`*  logs in, he/she will only be able to manage *`appId1`* from the dashboard.


