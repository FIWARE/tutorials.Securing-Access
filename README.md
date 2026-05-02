[![FIWARE Banner](https://fiware.github.io/tutorials.Securing-Access/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Security](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/security.svg)](https://github.com/FIWARE/catalogue/blob/master/security/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Securing-Access.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

This tutorial secures access to a FIWARE application using the entities created in the
[previous tutorial](https://github.com/FIWARE/tutorials.Roles-Permissions). The tutorial explains appropriate use of the
various OAuth2 grant flows, and how to use the **Keycloak** generic enabler as an Authorization Server to identify
users. **Keycloak** is also used as a Policy Decision Point (PDP) to restrict access.

The tutorial discusses code showing how to integrate Keycloak within a web application and demonstrates examples of
Authorization Server interactions using the **Keycloak** GUI. Some [cUrl](https://ec.haxx.se/) commands are also used to
access the **Keycloak** REST API - [Postman documentation](https://fiware.github.io/tutorials.Securing-Access/) is also
available.

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/66d8ba3abaf7319941b1)

-   このチュートリアルは[日本語](README.ja.md)でもご覧いただけます。

## Contents

<details>
<summary><strong>Details</strong></summary>

-   [Securing Access](#securing-access)
    -   [Standard Concepts of Identity Management](#standard-concepts-of-identity-management)
-   [Prerequisites](#prerequisites)
    -   [Docker](#docker)
    -   [Cygwin](#cygwin)
-   [Architecture](#architecture)
    -   [Tutorial Security Configuration](#tutorial-security-configuration)
-   [Start Up](#start-up)
    -   [Dramatis Personae](#dramatis-personae)
-   [OAuth2 Grant Flows](#oauth2-grant-flows)
    -   [User Credentials Grant](#user-credentials-grant)
        -   [Logging-in with a Password](#logging-in-with-a-password)
        -   [Retrieving User Details from an Access Token](#retrieving-user-details-from-an-access-token)
        -   [User Credentials - Sample Code](#user-credentials---sample-code)
        -   [User Credentials - Running the Example](#user-credentials---running-the-example)
    -   [Authorization Code Grant](#authorization-code-grant)
        -   [Authorization Code - Sample Code](#authorization-code---sample-code)
        -   [Authorization Code - Running the Example](#authorization-code---running-the-example)
    -   [Implicit Grant](#implicit-grant)
        -   [Implicit Grant - Sample Code](#implicit-grant---sample-code)
        -   [Implicit Grant - Running the Example](#implicit-grant---running-the-example)
    -   [Client Credentials Grant](#client-credentials-grant)
        -   [Logging in as an Application](#logging-in-as-an-application)
        -   [Client Credentials Grant - Sample Code](#client-credentials-grant---sample-code)
        -   [Client Credentials Grant - Running the Example](#client-credentials-grant---running-the-example)
    -   [Refresh Token](#refresh-token)
        -   [Availability Check](#availability-check)
        -   [Refresh Access Token](#refresh-access-token)
        -   [Refresh Token - Sample Code](#refresh-token---sample-code)
        -   [Refresh Token - Running the Example](#refresh-token---running-the-example)
-   [PDP - Access Control](#pdp---access-control)
    -   [Authentication Access](#authentication-access)
        -   [Authentication Access - Sample Code](#authentication-access---sample-code)
    -   [Basic Authorization](#basic-authorization)
        -   [Basic Authorization - Sample Code](#basic-authorization---sample-code)
    -   [PDP Access Control - Running the Example](#pdp-access-control---running-the-example)

</details>

# Securing Access

> "Don't let the fox guard the henhouse"
>
> — Proverb

In order to secure access to application resources, it is necessary to know two things. Firstly, who is making the
request and secondly is the requestor permitted to access the resource? The **Keycloak** generic enabler uses
[OAuth2](https://oauth.net/2/) to enable third-party applications to obtain limited access to services. **OAuth2** is
the open standard for access delegation to grant access rights. It allows notifying a resource provider (e.g. the
Knowage Generic Enabler) that the resource owner (e.g. you) grants permission to a third-party (e.g. a Knowage
Application) access to their information (e.g. the list of entities).

There are several common OAuth 2.0 grant flows, the details of which can be found below:

-   [Authorization Code](https://oauth.net/2/grant-types/authorization-code)
-   [Implicit](https://oauth.net/2/grant-types/implicit)
-   [Password](https://oauth.net/2/grant-types/password)
-   [Client Credentials](https://oauth.net/2/grant-types/client-credentials)
-   [Device Code](https://oauth.net/2/grant-types/device-code)
-   [Refresh Token](https://oauth.net/2/grant-types/refresh-token)

The primary concept is that both **Users** and **Applications** must first identify themselves using a standard OAuth2
Challenge-Response mechanism. Thereafter a user is assigned a token which they append to every subsequent request. This
token identifies the user, the application and the rights the user is able to exercise. **Keycloak** can then be used
with other enablers can be used to limit and lock-down access. The details of the access flows are discussed below and
in subsequent tutorials.

The reasoning behind OAuth2 is that you never need to expose your own username and password to a third party to give
them full access - you merely permit the relevant access which can be either Read-Only or Read-Write and such access can
be defined down to a granular level. Furthermore there is provision for revoking access at any time, leaving the
resource owner in control of who can access what.

Once the application is able to authenticate users, it is also possible to lock down access using access control
mechanisms. Access control requires having an access policy - in other words defining who can do what. We have already
defined roles and permissions within the [previous tutorial](https://github.com/FIWARE/tutorials.Roles-Permissions), and
now need to programmatically enforce this policy by adding in a simple Policy Decision Point (PDP) – which evaluates and
issues authorization decisions, and then secure access by enforcing the decision using a Policy Enforcement Point (PEP).

## Standard Concepts of Identity Management

The following common objects are found with the **Keycloak** Identity Management database:

-   **User** - Any signed up user able to identify themselves with an eMail and password. Users can be assigned rights
    individually or as a group
-   **Application** - Any securable FIWARE application consisting of a series of microservices
-   **Organization** - A group of users who can be assigned a series of rights. Altering the rights of the organization
    effects the access of all users of that organization
-   **OrganizationRole** - Users can either be members or admins of an organization - Admins are able to add and remove
    users from their organization, members merely gain the roles and permissions of an organization. This allows each
    organization to be responsible for their members and removes the need for a super-admin to administer all rights
-   **Role** - A role is a descriptive bucket for a set of permissions. A role can be assigned to either a single user
    or an organization. A signed-in user gains all the permissions from all of their own roles plus all of the roles
    associated to their organization
-   **Permission** - An ability to do something on a resource within the system

Additionally two further non-human application objects can be secured within a FIWARE application:

-   **IoTAgent** - a proxy between IoT Sensors and the Context Broker
-   **PEPProxy** - a middleware for use between generic enablers challenging the rights of a user.

The relationship between the objects can be seen below - the entities marked in red are used directly within this
tutorial:

![](https://fiware.github.io/tutorials.Securing-Access/img/entities-ld.png)

# Prerequisites

## Docker

To keep things simple both components will be run using [Docker](https://www.docker.com). **Docker** is a container
technology which allows to different components isolated into their respective environments.

-   To install Docker on Windows follow the instructions [here](https://docs.docker.com/docker-for-windows/)
-   To install Docker on Mac follow the instructions [here](https://docs.docker.com/docker-for-mac/)
-   To install Docker on Linux follow the instructions [here](https://docs.docker.com/install/)

**Docker Compose** is a tool for defining and running multi-container Docker applications. A
[YAML file](https://raw.githubusercontent.com/Fiware/tutorials.Identity-Management/master/docker-compose.yml) is used
configure the required services for the application. This means all container services can be brought up in a single
command. Docker Compose is installed by default as part of Docker for Windows and Docker for Mac, however Linux users
will need to follow the instructions found [here](https://docs.docker.com/compose/install/)

## Cygwin

We will start up our services using a simple bash script. Windows users should download [cygwin](http://www.cygwin.com/)
to provide a command-line functionality similar to a Linux distribution on Windows.

# Architecture

This application adds OAuth2-driven security into the existing Farm Management and Sensors-based application created in
[previous tutorials](https://github.com/FIWARE/tutorials.IoT-Agent/) by using the data created in the first
[security tutorial](https://github.com/FIWARE/tutorials.Identity-Management/) and reading it programmatically. It will
make use of three FIWARE components - the [Orion-LD Context Broker](https://fiware-orion.readthedocs.io/en/latest/), the
[IoT Agent for JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/) and integrates the use of
[Keycloak](https://www.keycloak.org/) as the Identity and Access Management server. Usage of the Orion-LD Context Broker
is sufficient for an application to qualify as _“Powered by FIWARE”_.

Both the Orion-LD Context Broker and the IoT Agent rely on open source [MongoDB](https://www.mongodb.com/) technology to
keep persistence of the information they hold. We will also be using the dummy IoT devices created in the
[previous tutorial](https://github.com/FIWARE/tutorials.IoT-Sensors/). **Keycloak** uses its own
[PostgreSQL](https://www.postgresql.org/) database.

Therefore the overall architecture will consist of the following elements:

-   The FIWARE [Orion-LD Context Broker](https://fiware-orion.readthedocs.io/en/latest/) which will receive requests
    using [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
-   The FIWARE [IoT Agent for JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/) which will receive
    southbound requests using [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
    and convert them to [JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
    commands for the devices
-   [Keycloak](https://www.keycloak.org/) Identity and Access Management offering:
    -   An OAuth2 / OIDC authentication system for Applications and Users
    -   A graphical frontend for Identity Management Administration
    -   A REST API for Identity Management via HTTP requests
-   The underlying [MongoDB](https://www.mongodb.com/) database :
    -   Used by the **Orion-LD Context Broker** to hold context data information such as data entities, subscriptions and
        registrations
    -   Used by the **IoT Agent** to hold device information such as device URLs and Keys
-   A [PostgreSQL](https://www.postgresql.org/) database :
    -   Used by **Keycloak** to persist user identities, applications, roles and permissions
-   The **Farm Management Frontend** does the following:
    -   Displays farm building and sensor information
    -   Shows which animals and equipment are present
    -   Allows authorized users to send commands to IoT devices
    -   Allows authorized users into restricted areas
-   A webserver acting as set of [dummy IoT devices](https://github.com/FIWARE/tutorials.IoT-Sensors/) using
    the [JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
    protocol running over HTTP - access to certain resources is restricted.

Since all interactions between the elements are initiated by HTTP requests, the entities can be containerized and run
from exposed ports.

![](https://fiware.github.io/tutorials.Securing-Access/img/architecture.png)

The necessary configuration information for adding security to the **Farm Management Frontend** can be found in the
`tutorial` section of the associated `docker-compose.yml` file - only the relevant variables are shown below:

## Tutorial Security Configuration

```yaml
tutorial:
    image: quay.io/fiware/tutorials.ngsi-ld
    hostname: tutorial
    container_name: fiware-tutorial
    networks:
        default:
            ipv4_address: 172.18.1.11
    expose:
        - "${TUTORIAL_APP_PORT}"
    ports:
        - "${TUTORIAL_APP_PORT}:${TUTORIAL_APP_PORT}"
    environment:
        - DEBUG=tutorial:*
        - WEB_APP_PORT=${TUTORIAL_APP_PORT}
        - SECURE_ENDPOINTS=true
        - OIDC_ISSUER=http://keycloak:8080/realms/farm-management
        - KEYCLOAK_URL=http://localhost:${KEYCLOAK_PORT}/realms/farm-management
        - OIDC_CLIENT_ID=ngsi-ld-farm
        - OIDC_CLIENT_SECRET=1234
        - OIDC_REDIRECT_URI=http://localhost:${TUTORIAL_APP_PORT}/login/callback
        - OIDC_SCOPE=openid profile email
```

The `tutorial` container is driven by environment variables as shown:

| Key                | Value                                                    | Description                                                                         |
| ------------------ | -------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| DEBUG              | `tutorial:*`                                             | Debug flag used for logging                                                         |
| WEB_APP_PORT       | `3000`                                                   | Port used by web-app which displays the login screen etc.                           |
| SECURE_ENDPOINTS   | `true`                                                   | Enables PDP enforcement on protected routes                                         |
| OIDC_ISSUER        | `http://keycloak:8080/realms/farm-management`            | Internal URL of the **Keycloak** realm — used for server-to-server calls            |
| KEYCLOAK_URL       | `http://localhost:3005/realms/farm-management`           | Public URL of **Keycloak** — used for browser redirects (login / logout)            |
| OIDC_CLIENT_ID     | `ngsi-ld-farm`                                           | The Client ID registered in Keycloak for this application                           |
| OIDC_CLIENT_SECRET | `1234`                                                   | The Client Secret registered in Keycloak for this application                       |
| OIDC_REDIRECT_URI  | `http://localhost:3000/login/callback`                   | The callback URL Keycloak will redirect to after a successful authentication        |
| OIDC_SCOPE         | `openid profile email`                                   | OIDC scopes requested during authentication                                         |

The other `tutorial` container configuration values described in the YAML file have been described in previous tutorials.

`OIDC_ISSUER` and `KEYCLOAK_URL` are split because the tutorial runs inside Docker. The `OIDC_ISSUER` value uses the
internal Docker hostname `keycloak` for server-to-server communication (token exchange, introspection). The
`KEYCLOAK_URL` value uses `localhost` — the port that is exposed to the host — so that browser redirects for login and
logout resolve correctly. All of these values should be replaced with appropriate DNS entries for a production
environment, but production deployment is beyond the scope of this tutorial.

# Start Up

To start the installation, do the following:

```console
git clone https://github.com/FIWARE/tutorials.Step-by-Step.git
cd NGSI-LD/tutorials.Securing-Access

./services create
```

> **Note** The initial creation of Docker images can take up to three minutes

Thereafter, all services can be initialized from the command-line by running the
[services](https://github.com/FIWARE/tutorials.Securing-Access/blob/NGSI-LD/services) Bash script provided within the
repository:

```console
./services <command>
```

Where `<command>` will vary depending upon the exercise we wish to activate.

> :information_source: **Note:** If you want to clean up and start over again you can do so with the following command:
>
> ```console
> ./services stop
> ```

## Dramatis Personae

The following people at `fiware.farm` legitimately have accounts within the Farm Management Information System:

-   **Bob**, the Farm Manager - he has full control over the farm and all entities.
-   **Carol**, a Livestock Supervisor - she manages animals and related sensors (water, filling levels).
-   **Jenny**, a Read-Only Consultant - an external auditor who can view all farm data but cannot make changes.
-   **Alice**, the System Administrator - she manages the Keycloak instance but does not have direct access to farm data
    by default.

The following person at `fiware.farm` has signed up for an account but has no reason to be granted access:

-   **Mallory**, the Malicious Attacker - she should be denied access to all farm resources.

### 1. Defined Roles & Capabilities

The following roles are defined within the `farm-management` realm:

| Role                       | Description                          | Access Level                           |
| :------------------------- | :----------------------------------- | :------------------------------------- |
| **`farm-manager`**         | Full control over the farm.          | **Read & Write** (All Entities)        |
| **`livestock-supervisor`** | Manages animals and related sensors. | **Read & Write** (Animal, Water, etc.) |
| **`read-only-consultant`** | External auditor/viewer.             | **Read Only** (All Entities)           |
| **`crop-supervisor`**      | Manages fields and weather data.     | Read & Write (Fields, Soil)            |
| **`equipment-supervisor`** | Manages tractors and machinery.      | Read & Write (Tractors)                |
| **`field-worker`**         | Worker on the ground.                | Read (Domain), Write (Measurements)    |

### 2. User Assignments (Initial Setup)

For the purpose of this tutorial, the following users have been provisioned with the credentials below (password is
always `test`):

| User        | Group                  | Assigned Role      | Effective Rights                              |
| :---------- | :--------------------- | :----------------- | :-------------------------------------------- |
| **Bob**     | `farm-management`      | **`farm-manager`** | **Full Read/Write** access to all entities.   |
| **Carol**   | `livestock-team`       | _None (Directly)_  | **Access Denied** (No role mapping for group) |
| **Jenny**   | `external-consultants` | _None (Directly)_  | **Access Denied** (No role mapping for group) |
| **Alice**   | _None_                 | _None_             | **Access Denied** (No roles assigned)         |
| **Mallory** | _None_                 | _None_             | **Access Denied** (No roles assigned)         |

> [!NOTE] In the initial setup, **Bob** is the only user with functional access to the data because he is the only one
> explicitly assigned a role (`farm-manager`). For Carol or Jenny to have access, their respective groups would need to
> be mapped to the `livestock-supervisor` or `read-only-consultant` roles within Keycloak.

One application (`ngsi-ld-farm`), with appropriate roles and permissions has also been created:

| Key           | Value                         |
| ------------- | ----------------------------- |
| Client ID     | `ngsi-ld-farm`                |
| Client Secret | `1234`                        |
| URL           | `http://localhost:3000`       |
| RedirectURL   | `http://localhost:3000/login` |

To save time, the data creating users and organizations from the
[previous tutorial](https://github.com/FIWARE/tutorials.Roles-Permissions) has been imported and is automatically
persisted to the PostgreSQL database on start-up so the assigned UUIDs do not change and the data does not need to be
entered again.

The **Keycloak** PostgreSQL database deals with all aspects of application security including storing users, passwords
etc.; defining access rights and dealing with OAuth2 / OIDC authorization protocols.

To refresh your memory about how to create users, groups and clients, you can log in to the Keycloak Admin Console at
`http://localhost:3005` using the account `alice` with a password of `test`.

![](https://fiware.github.io/tutorials.Securing-Access/img/tutorial-log-in.png)

and look around.

# OAuth2 Grant Flows

As noted in the documentation, **Keycloak** complies with the OAuth2 standard described in
[RFC 6749](http://tools.ietf.org/html/rfc6749) and supports all four standard grant types defined there.

When making requests the `Authorization` header is built by combining the application Client ID and Client Secret
credentials provided by **Keycloak**, separated by a `:` and base-64 encoded. The value can be generated as shown:

```console
echo -n ngsi-ld-farm:1234 | base64
```

```
bmdzaS1sZC1mYXJtOjEyMzQ=
```

All four major grant flows can be demonstrated within the tutorial application; the actual flow to pick will depend on
your own use case.

## User Credentials Grant

The [User Credentials](https://tools.ietf.org/html/rfc6749#section-1.3.3) grant flow, also known as the _resource owner
password credentials grant_ or password grant should only be used when:

-   A User wants to log into an application via a web-app client
-   The web-app client is absolutely trusted

![](https://fiware.github.io/tutorials.Securing-Access/img/user-credentials.png)

This is the most appropriate usage within the Farm Management Tutorial Application, as the Web-App has been written by
us and we can trust it to pass on credentials to a **Keycloak** instance also owned by us. As you can see from the
diagram, the user must type their own password into the web-app client itself.

### Logging-in with a Password

#### :one: Request

To log in using the User Credentials flow, send a POST request to the Keycloak token endpoint with
`grant_type=password`:

```console
curl -iX POST \
  'http://localhost:3005/realms/farm-management/protocol/openid-connect/token' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data 'username=bob%40fiware.farm&password=test&grant_type=password&client_id=ngsi-ld-farm&client_secret=1234&scope=openid+profile+email'
```

> **Note:** the username in the grant is the email address of the user in this instance.

#### Response

The response returns an `access_token` and `id_token` to identify the user:

```json
{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6Ii4uLiJ9...",
    "expires_in": 300,
    "refresh_expires_in": 1800,
    "refresh_token": "eyJhbGciOiJIUzUxMiIsInR5cCIgOiAiSldUIiwia2lkIiA6Ii4uLiJ9...",
    "token_type": "Bearer",
    "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6Ii4uLiJ9...",
    "scope": "openid profile email"
}
```

### Retrieving User Details from an Access Token

The access token can then be used with a GET request to the Keycloak `/userinfo` endpoint to obtain user details:

#### :two: Request

```console
curl -X GET \
  'http://localhost:3005/realms/farm-management/protocol/openid-connect/userinfo' \
  -H 'Authorization: Bearer <access_token>'
```

#### Response

The username (bob) and associated claims are returned as shown:

```json
{
    "sub": "bbbbbbbb-good-0000-0000-000000000000",
    "email_verified": true,
    "name": "Bob Manager",
    "preferred_username": "bob",
    "given_name": "Bob",
    "family_name": "Manager",
    "email": "bob@fiware.farm",
    "realm_access": {
        "roles": ["farm-manager"]
    }
}
```

### User Credentials - Sample Code

The code delegates all Keycloak interactions to a separate library
[keycloak.js](https://github.com/FIWARE/tutorials.Step-by-Step/blob/master/NGSI-LD/app/lib/keycloak.js). Every request
is wrapped in a promise to simplify the application code. The User Credentials flow is invoked using
`keycloak.getUserCredentials()` - once an `access_token` is received the user details are retrieved using a separate
`keycloak.getUserInfo()` call as shown:

```javascript
function userCredentialGrant(req, res) {
    const email = req.body.email;
    const password = req.body.password;

    keycloak
        .getUserCredentials(email, password)
        .then(({ status, body }) => {
            if (status !== 200 || !body.access_token) {
                throw new Error(body.error_description || 'Password grant failed');
            }
            storeTokens(req, body);
            return keycloak.getUserInfo(body.access_token);
        })
        .then(({ body: user }) => {
            const username = user.preferred_username || user.sub;
            req.session.username = username;
            req.flash('success', username + ' logged in with <strong>Password</strong>');
            return res.redirect('/');
        })
        .catch((error) => {
            req.flash('error', 'Access Denied');
            return res.redirect('/');
        });
}
```

The underlying `getUserCredentials()` function posts the password grant to the Keycloak token endpoint:

```javascript
function getUserCredentials(username, password) {
    const body = querystring.stringify({
        grant_type: 'password',
        client_id: clientId,
        client_secret: clientSecret,
        username,
        password,
        scope
    });
    return post(tokenEndpoint, body);
}
```

### User Credentials - Running the Example

It is possible to invoke the User Credentials grant flow programmatically, by bringing up the page
`http://localhost:3000/` and filling out the username and password form.

![](https://fiware.github.io/tutorials.Securing-Access/img/tutorial-log-in.png)

The response displays the user on the top right of the screen, with a success flash message:

![](https://fiware.github.io/tutorials.Securing-Access/img/tutorial-response.png)

## Authorization Code Grant

The [Authorization Code](https://tools.ietf.org/html/rfc6749#section-1.3.1) grant flow can be used where the client (in
our case the Tutorial Web-application) doesn't need access to any passwords directly - it just needs to know who the
user is. With the Authorization Code grant, the user is redirected to an Authorization Server such as **Keycloak**, logs
in there and permits access. The response returns an access-code which can be exchanged for an access-token which then
identifies the user.

![](https://fiware.github.io/tutorials.Securing-Access/img/authcode-flow.png)

This is an example of the sort of flow used when a third party (such as Travis-CI) asks you to log in using your GitHub
account. Travis never gains access to your password, but does receive details that you are who you claim to be from
GitHub.

### Authorization Code - Sample Code

A user must first be redirected to **Keycloak** using PKCE. The `keycloak.getAuthorizeUrl()` function returns a URL of
the form:
`/protocol/openid-connect/auth?response_type=code&client_id={{client-id}}&state=xyz&redirect_uri={{callback_url}}&code_challenge={{challenge}}&code_challenge_method=S256`

```javascript
function authCodeGrant(req, res) {
    const verifier = keycloak.generateCodeVerifier();
    const challenge = keycloak.generateCodeChallenge(verifier);
    const state = require('crypto').randomBytes(16).toString('hex');

    req.session.pkce_verifier = verifier;
    req.session.oauth_state = state;

    const url = keycloak.getAuthorizeUrl(state, challenge);
    return res.redirect(url);
}
```

After the user authenticates, the response is received at the `redirect_uri` callback. The authorization code is
exchanged for tokens using PKCE:

```javascript
function authCodeGrantCallback(req, res) {
    const code = req.query.code;
    const verifier = req.session.pkce_verifier;

    return keycloak
        .exchangeCode(code, verifier)
        .then(({ status, body }) => {
            storeTokens(req, body);
            return keycloak.getUserInfo(body.access_token);
        })
        .then(({ body: user }) => {
            req.session.username = user.preferred_username || user.sub;
            return res.redirect('/');
        });
}
```

### Authorization Code - Running the Example

It is possible to invoke the Authorization Code grant flow programmatically, by bringing up the page
`http://localhost:3000/` and clicking on the **Authorization Code** link.

The user is redirected to **Keycloak** and must log in:

![](https://fiware.github.io/tutorials.Securing-Access/img/tutorial-log-in.png)

The response displays the user on the top right of the screen, with a success flash message.

> **Note** Unless you deliberately log out of **Keycloak** at `http://localhost:3005`, the existing **Keycloak** session
> which has already permitted access will be used for subsequent authorization requests, so the **Keycloak** login
> screen will not be shown again.

## Implicit Grant

The [Implicit](https://tools.ietf.org/html/rfc6749#section-1.3.2) grant flow is a simplified form of the Authorization
grant flow where **Keycloak** returns an `access_token` directly rather than returning an interim access-code. This is
less secure than the Authcode flow but can be used in some client-side applications

![](https://fiware.github.io/tutorials.Securing-Access/img/implicit-flow.png)

### Implicit Grant - Sample Code

A user must first be redirected to **Keycloak**, requesting a `token` directly. The `keycloak.getImplicitAuthorizeUrl()`
function returns a URL of the form:
`/protocol/openid-connect/auth?response_type=id_token+token&client_id={{client-id}}&state=xyz&redirect_uri={{callback_url}}&nonce={{nonce}}`

```javascript
function implicitGrant(req, res) {
    const state = require('crypto').randomBytes(16).toString('hex');
    req.session.oauth_state = state;

    const url = keycloak.getImplicitAuthorizeUrl(state);
    return res.redirect(url);
}
```

After the user authenticates, **Keycloak** posts the tokens directly to the `redirect_uri` without a code exchange step:

```javascript
function authCodeGrantCallback(req, res) {
    // For the Implicit flow, access_token arrives directly in the POST body
    const accessToken = req.body.access_token;
    return keycloak
        .getUserInfo(accessToken)
        .then(({ body: user }) => {
            req.session.username = user.preferred_username || user.sub;
            return res.redirect('/');
        });
}
```

### Implicit Grant - Running the Example

It is possible to invoke the Implicit grant flow programmatically, by bringing up the page `http://localhost:3000/` and
clicking on the **Implicit Grant** link.

The user is redirected to **Keycloak** and must log in:

![](https://fiware.github.io/tutorials.Securing-Access/img/tutorial-log-in.png)

The response displays the user on the top right of the screen, with a success flash message.

> **Note** Unless you deliberately log out of **Keycloak** at `http://localhost:3005`, the existing **Keycloak** session
> which has already permitted access will be used for subsequent authorization requests.

## Client Credentials Grant

The final grant flow - the [Client Credentials](https://tools.ietf.org/html/rfc6749#section-1.3.4) grant, does not need
a user. It is sometimes necessary for an application to identify itself so that the application (rather than the user)
is granted access to resources. There are no resources secured in such a manner within this tutorial, but the flow has
been included for completeness.

![](https://fiware.github.io/tutorials.Securing-Access/img/client-credentials.png)

### Logging in as an Application

To log in using the Client Credentials flow send a POST request to the Keycloak token endpoint with
`grant_type=client_credentials`:

#### :three: Request:

```console
curl -iX POST \
  'http://localhost:3005/realms/farm-management/protocol/openid-connect/token' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data 'grant_type=client_credentials&client_id=ngsi-ld-farm&client_secret=1234'
```

#### Response:

The response returns an access token to identify the application itself.

```json
{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6Ii4uLiJ9...",
    "expires_in": 300,
    "token_type": "Bearer",
    "scope": "profile email"
}
```

### Client Credentials Grant - Sample Code

The code is similar to the User Credentials Grant, but without an explicit username or password:

```javascript
function clientCredentialGrant(req, res) {
    keycloak
        .getClientCredentials()
        .then(({ status, body }) => {
            storeTokens(req, body);
            req.session.username = 'Application';
            req.flash('success', 'Application logged in with <strong>Client Credentials</strong>');
            return res.redirect('/');
        })
        .catch(() => res.redirect('/'));
}
```

### Client Credentials Grant - Running the Example

It is possible to invoke the Client Credentials grant flow programmatically, by bringing up the page
`http://localhost:3000/` and clicking on the **Client Credentials** link.

The response displays the details of the token. No User is involved.

## Refresh Token

Once a User has identified themselves (using any appropriate grant type), they should not need to log-in again, even
though the `access_token` they are using is time-limited. To provide continued access, an addition
[Refresh Token](https://tools.ietf.org/html/rfc6749#section-1.5) flow has been defined, allowing a User to exchange an
expired token for a new one. Offering this exchange is not mandatory for OAuth2 Authorization Servers, and is not
appropriate for all grant types.

### Availability Check

#### :four: Request

Check to see if the Refresh Token flow is available by logging in using one of the other grant types. For example, to
log in using the User Credentials flow send a POST request to the Keycloak token endpoint with `grant_type=password`:

```console
curl -iX POST \
  'http://localhost:3005/realms/farm-management/protocol/openid-connect/token' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data 'username=bob%40fiware.farm&password=test&grant_type=password&client_id=ngsi-ld-farm&client_secret=1234&scope=openid+profile+email'
```

#### Response

Along with the `access_token` identifying the user, the response also returns a `refresh_token`:

```json
{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6Ii4uLiJ9...",
    "expires_in": 300,
    "refresh_expires_in": 1800,
    "refresh_token": "eyJhbGciOiJIUzUxMiIsInR5cCIgOiAiSldUIiwia2lkIiA6Ii4uLiJ9...",
    "token_type": "Bearer",
    "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6Ii4uLiJ9...",
    "scope": "openid profile email"
}
```

### Refresh Access Token

The `refresh_token` from the response above is stored for later use. To obtain a new `access_token` (for example once
the previous one has expired) the `refresh_token` is used with `grant_type=refresh_token`:

#### :five: Request

```console
curl -iX POST \
  'http://localhost:3005/realms/farm-management/protocol/openid-connect/token' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data 'grant_type=refresh_token&client_id=ngsi-ld-farm&client_secret=1234&refresh_token=<refresh_token>'
```

#### Response

The response is similar to the previous response, but the `access_token` and `refresh_token` have been updated and the
expiry window has been moved forward.

```json
{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6Ii4uLiJ9...",
    "expires_in": 300,
    "refresh_expires_in": 1800,
    "refresh_token": "eyJhbGciOiJIUzUxMiIsInR5cCIgOiAiSldUIiwia2lkIiA6Ii4uLiJ9...",
    "token_type": "Bearer",
    "scope": "openid profile email"
}
```

### Refresh Token - Sample Code

The code delegates the token refresh to
[keycloak.js](https://github.com/FIWARE/tutorials.Step-by-Step/blob/master/NGSI-LD/app/lib/keycloak.js). The Refresh
Token flow is invoked using `keycloak.refreshAccessToken()` — the previously received `refresh_token` is used to obtain
a new `access_token` once the previous one has expired:

```javascript
function refreshTokenGrant(req, res) {
    return keycloak
        .refreshAccessToken(req.session.refresh_token)
        .then(({ status, body }) => {
            storeTokens(req, body);
            req.flash('success', req.session.username + ' <strong>refreshed token</strong>');
            return res.redirect('/');
        })
        .catch(() => res.redirect('/'));
}
```

### Refresh Token - Running the Example

First log in using the username and password form at `http://localhost:3000/`:

![](https://fiware.github.io/tutorials.Securing-Access/img/tutorial-log-in.png)

Once logged in, press the **Refresh Token** button to obtain a new `access_token` and `refresh_token`:

![](https://fiware.github.io/tutorials.Securing-Access/img/tutorial-response.png)

Pressing **Refresh Token** returns a new `access_token` and `refresh_token` for the logged-in user.

# PDP - Access Control

There are three Levels of PDP Access Control:

-   Level 1: Authentication Access - Allow all actions to every signed in user and no actions to an anonymous user.
-   Level 2: Basic Authorization - Check which resources and verbs the currently logged in user should have access to
-   Level 3: Advanced Authorization - Fine grained control through [XACML](https://en.wikipedia.org/wiki/XACML)

**Keycloak** can be used to offer a simple Level 1 and 2 PDP on its own, and can offer level 3 combined with additional
[generic enablers](https://github.com/FIWARE/tutorials.XACML-Access-Rules). This tutorial will only be concerned with
the logged in site itself. Securing other services in conjunction with a
[PEP Proxy](https://fiware-pep-proxy.readthedocs.io/en/latest) will be dealt with in a
[later tutorial](https://github.com/FIWARE/tutorials.PEP-Proxy)

## Authentication Access

If **Keycloak** (or any other OAuth2 provider) has successfully logged in, an `access_token` has been provided saying
that the user exists. This information is sufficient to **authenticate** a User.

Level 1 PDP can be used in conjunction with any OAuth2 provider using any flow.

Because the tutorial uses **Keycloak**-issued JWTs stored server-side, token freshness can be verified locally by
decoding the JWT payload and checking the `exp` claim — no network round-trip required.

#### :six: Request

```console
curl -X GET \
  'http://localhost:3005/realms/farm-management/protocol/openid-connect/userinfo' \
  -H 'Authorization: Bearer <access_token>'
```

A `200 OK` response indicates a valid, non-expired `access_token`.

### Authentication Access - Sample Code

To check that a user has logged in, just store the `access_token` in session when they log in and check for its
existence. The JWT is decoded to read the `exp` claim without a network call:

```javascript
function authenticate(req, res, next) {
    if (!req.session.access_token) {
        res.locals.authorized = false;
        return next();
    }

    const claims = req.session.claims || decodeJwtPayload(req.session.access_token);
    if (!claims) {
        res.locals.authorized = false;
        return next();
    }

    const now = Math.floor(Date.now() / 1000);
    if (claims.exp && claims.exp < now) {
        res.locals.authorized = false;
        return next();
    }

    res.locals.authorized = true;
    return next();
}
```

## Basic Authorization

Level 2 PDP can only be used with our own trusted instance of **Keycloak**, usually via the Password Grant flow.

Once a user has signed in and obtained an `access_token`, the roles embedded in the JWT can be inspected server-side
without any additional network call to Keycloak. The `realm_access.roles` claim in the JWT contains the set of realm
roles assigned to the user.

#### :seven: Request

To introspect a token and confirm its roles, send a POST to the Keycloak introspect endpoint:

```console
curl -X POST \
  'http://localhost:3005/realms/farm-management/protocol/openid-connect/token/introspect' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data 'token=<access_token>&client_id=ngsi-ld-farm&client_secret=1234'
```

#### Response

The response includes an `active` flag and the user's role claims:

```json
{
    "active": true,
    "sub": "bbbbbbbb-good-0000-0000-000000000000",
    "preferred_username": "bob",
    "email": "bob@fiware.farm",
    "realm_access": {
        "roles": ["farm-manager"]
    },
    "client_id": "ngsi-ld-farm"
}
```

### Basic Authorization - Sample Code

**Keycloak** issues JWTs that carry role information directly. The authorization middleware reads `realm_access.roles`
from the stored JWT claims — no Keycloak API call is required at request time:

```javascript
function authorizeBasicPDP(req, res, next) {
    if (!req.session.access_token) {
        res.locals.authorized = false;
        return next();
    }

    const roles = getRoles(req); // reads realm_access.roles from JWT
    const method = req.method.toUpperCase();

    // farm-manager can do everything; read-only-consultant can only GET
    if (roles.includes('farm-manager')) {
        res.locals.authorized = true;
    } else if (method === 'GET' || method === 'HEAD') {
        res.locals.authorized = roles.length > 0;
    } else {
        // POST / PATCH / DELETE require a write-capable role
        res.locals.authorized =
            roles.includes('livestock-supervisor') ||
            roles.includes('crop-supervisor') ||
            roles.includes('equipment-supervisor');
    }

    return next();
}
```

A secured Web Page needs to check if the `authorized` flag has been set, and redirect the user if disallowed. This is an
example of a Policy Enforcement Point (PEP):

```javascript
function priceChange(req, res) {
    if (!res.locals.authorized) {
        req.flash("error", "Access Denied");
        return res.redirect("/");
    }
    /// Continue with the normal flow of execution...
}
```

Similarly a secured command can fail fast and return an error code if the user is not authorized, this is another
example of a Policy Enforcement Point (PEP):

```javascript
function sendCommand(req, res) {
    if (!res.locals.authorized) {
        res.setHeader("Content-Type", "application/json");
        return res.status(403).send({ message: "Forbidden" });
    }
    /// Continue with the normal flow of execution...
}
```

## PDP Access Control - Running the Example

> **Note** The following resources are secured within the FMIS tutorial application:
>
> -   Viewing individual farm building pages (Level 1)
> -   Sending IoT device commands (Level 2 — write-capable roles only)
> -   Accessing restricted farm management areas (Level 2)

#### Anonymous Access

-   Ensure that you are not signed in as any user.

##### Level 1 : Authentication Access

-   Click on any farm building page — access is **denied** for anonymous users
-   Open the Device Monitor on `http://localhost:3000/device/monitor`
    -   Send a device command — access is **denied** for anonymous users

##### Level 2 : Authorization Access

-   All device commands at `http://localhost:3000/device/monitor` — access is **denied**

#### Mallory the Malicious Attacker

Mallory has an account but has been assigned no roles.

-   From `http://localhost:3000`, log in as `mallory@fiware.farm` with the password `test`

##### Level 1 : Authentication Access

-   Click on any farm building page — access is **permitted** for any logged-in user

##### Level 2 : Authorization Access

-   Send any device command — access is **denied** (no roles assigned)

#### Jenny the Read-Only Consultant

Jenny has the **`read-only-consultant`** role, granting read-only access.

-   From `http://localhost:3000`, log in as `jenny@fiware.farm` with the password `test`

##### Level 1 : Authentication Access

-   Click on any farm building page — access is **permitted** for any logged-in user

##### Level 2 : Authorization Access

-   GET requests to farm data — access is **permitted**
-   Sending device commands (POST) — access is **denied** (read-only role)

#### Bob the Farm Manager

Bob has the **`farm-manager`** role, granting full read/write access.

-   From `http://localhost:3000`, log in as `bob@fiware.farm` with the password `test`

##### Level 1 : Authentication Access

-   Click on any farm building page — access is **permitted** for any logged-in user

##### Level 2 : Authorization Access

-   All farm data pages — access is **permitted**
-   All device commands — access is **permitted** (farm-manager role)

---

## License

[MIT](LICENSE) © 2018-2026 FIWARE Foundation e.V.
