Credentials (API)
=================

This section describes how to create new users, sessions, and API keys, all through the CIMI API.

**Note:** remeber to use *-k* in all API requests below, if you are working with a test deployment without a green server certificate. 

Create a user
-------------

Once the mF2C system is up and running, 

1. create a new user by doing:

.. code-block:: bash

    curl -XPOST -H "Content-type: application/json" \
        https://<server>/api/user -d @addRegularUser.json

where *addRegularUser.json* is


.. code-block:: json

    {
        "userTemplate": {
            "href": "user-template/self-registration",
            "password": "testpassword",
            "passwordRepeat" : "testpassword",
            "emailAddress": "your@email.com",
            "username": "testuser"
        }
    }


2. check your email for a user validation email (if working with a test deployment of mF2C, check the Spam folder). Once you find the 
email, copy the validation URL and paste it in the browser, looking like this "`https://<server>/api/callback/a3a9b6d9-5229-455a-b31b-87fa2b950159/execute`"

3. once the user is validated, you can create a session and login:

.. code-block:: bash

    curl -XPOST -H 'content-type: application/json' \
        https://<server>/api/session -d @regularUser.json \
        --cookie-jar ~/cookies -b ~/cookies -sS

where *regularUser.json* is


.. code-block:: json

    {
        "sessionTemplate": {
            "href": "session-template/internal",
            "username": "testuser",
            "password": "testpassword"
        }
    }

**note** that ~/cookie expire by default after 1 day.

You are now logged in.

Generate an API key for a user
------------------------------

API keys are a safer way to have robots (scripts) interacting with the API on behalf of a user, 
since the same user can issue multiple API keys, and every one of them can be revoked without 
interfering with the original user access.

Basically CIMI distinguishes between internal logins and api_key logins, 
even though they might be associated with the same user account.

Before using API keys, create the session-template for it (only do it once, and if it doesn't exist yet):

.. code-block:: json

    curl -XPOST -H content-type:application/json -d '
        {
        "method": "api-key",
        "instance": "api-key",

        "name" : "Login with API Key and Secret",
        "description" : "Authentication with API Key and Secret",
        "group" : "Login with API Key and Secret",

        "key" : "key",
        "secret" : "secret",

        "acl": {
                    "owner": {"principal": "ADMIN",
                            "type":      "ROLE"},
                    "rules": [{"principal": "ADMIN",
                                "type":      "ROLE",
                                "right":     "ALL"},
                            {"principal": "ANON",
                                "type":      "ROLE",
                                "right":     "VIEW"},
                            {"principal": "USER",
                                "type":      "ROLE",
                                "right":     "VIEW"}]
                }
    }' https://<server>/api/session-template -H 'slipstream-authn-info: super ADMIN'

Then, to create an API key, do the following:

1. login (like demonstrated in step 3. of the previous section)

.. code-block:: bash

    curl -XPOST -H 'content-type: application/json' \
        https://<server>/api/session -d @regularUser.json \
        --cookie-jar ~/cookies -b ~/cookies -sS

2. generate an API key and secret

.. code-block:: bash

    curl -XPOST -H 'content-type: application/json' \
        https://localhost/api/credential -d @generateAPIKey.json \
        --cookie-jar ~/cookies -b ~/cookies -sS

where *generateAPIKey.json* is something like

.. code-block:: json

    {
        "credentialTemplate": {
            "href": "credential-template/generate-api-key",
            "ttl": 0
        }
    }

3. you'll get a server response similar to

.. code-block:: json

    {
        "status" : 201,
        "message" : "credential/4f8b8f66-2e15-4570-a14e-f9d3582425ad created",
        "resource-id" : "credential/4f8b8f66-2e15-4570-a14e-f9d3582425ad",
        "secretKey" : "nehrHa.V9Ppzb.vHf4BG.5vxv3j.DzLtqb"
    }


4. save the "resource-id" and "secretKey"

.. code-block:: bash

    export CIMI_API_KEY=credential/4f8b8f66-2e15-4570-a14e-f9d3582425ad
    export CIMI_API_SECRET=nehrHa.V9Ppzb.vHf4BG.5vxv3j.DzLtqb

5. create another session login, with regularUserAPIKey.json:

.. code-block:: bash

    cat >regularUserAPIKey.json <<EOF
    {
        "sessionTemplate": {
            "href": "session-template/api-key",
            "key": "$CIMI_API_KEY",
            "secret": "$CIMI_API_SECRET"
        }
    }
    EOF

    curl -XPOST -H 'content-type: application/json' \
    https://<server>/api/session -d @regularUserAPIKey.json \
    --cookie-jar ~/cookies -b ~/cookies -sS

And now you are logged in using an API key and secret instead of your internal user credentials.