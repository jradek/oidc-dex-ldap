# OIDC stack using loginapp, dex and OpenLDAP

This setup demonstrates how to configure and use an OIDC stack to enable token based authentication and authorization.
It uses the following tools

* [Loginapp](https://github.com/fydrah/loginapp) - Web application for Kubernetes CLI configuration with OIDC
* [DEX](https://github.com/dexidp/dex) - A federated OpenID Connect provider
* [OpenLDAP](https://github.com/openldap/openldap) - open source implementation of the **L**ightweight **D**irectory **A**ccess Protocol

The general flow goes like this. In the browser-based Loginapp the users enter their credentials - in this case email and password. This is send to DEX. DEX queries OpenLDAP to check if the user can login (authentication) and what permissions (authorization) they have - in this case group membership. On success, this information is stored in an JWT token which is presented to the user within Loginapp. Afterwards this token can be used for whatever service you need, e.g. access to kubernetes cluster.

## Installation

1. clone the repository
2. run `gencert.sh` to generate certificates (even this setup does not use **TLS**, Loginapp insists to get at least a root certificate)
3. Modify our `/etc/hosts` database

    ```bash
    echo "127.0.0.1 dex.example.com" >> /etc/hosts
    ```

4. Start the deployment

    ```bash
    docker-compose up
    ```

5. What happend

    * LoginApp runs on [http://localhost:5555]()
    * DEX listens on [http://localhost:5556]()
    * OpenLDAP listens on [http://localhost:18081]()

## Demo

1. Start the deployment
2. Connect to the OpenLDAP server with any client you like
   I recommend [Apache Directory Studio](https://directory.apache.org/studio/).
   Use the following connection settings

    ```
    Network
        Hostname: localhost
        Port    : 18081
    Authentication
        Method       : Simple Authentication
        Bind DN      : cn=admin,dc=example,dc=org
        Bind password: admin
   ```

3. Select the root node `dc=example,dc=org` in the LDAP DIT and import the example data from [config-ldap.ldif]()
4. Go to [http://localhost:5555]() and login with

    ```
    Email Address: janedoe@example.org
    Password     : foo
    ```

5. Hit `Grant Access`
6. Copy the `id-token` and decode it with [https://jwt.io/]().
   The payload should be something like this

    ```json
    {
        "iss": "http://dex.example.com:5556",
        "sub": "CiNjbj1qYW5lLG91PVBlb3BsZSxkYz1leGFtcGxlLGRjPW9yZxIEbGRhcA",
        "aud": "loginapp",
        "exp": 1586083939,
        "iat": 1585997539,
        "at_hash": "gtoi3BJD3mI0gPSekoVN5Q",
        "email": "janedoe@example.org",
        "email_verified": true,
        "groups": [
            "admins",
            "developers"
        ],
        "name": "jane"
    }
    ```

    You see some details for jane (email, name) taken from LDAP as well as her group membership: admins and developers

## How it works

Dex is configured in [config-dex.yaml]() and Loginapp in [config-loginapp.yaml](). Both files have inline comments.