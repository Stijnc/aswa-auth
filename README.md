# A simple azure static web app (aswa) behind Azure AD authentication

This repo serves as the basis to put a [full website behind authentication](https://aswauth.callebaut.dev/).

## TL;DR

 Hit 🔗 [https://aswauth.callebaut.dev/](https://aswauth.callebaut.dev/) to see this in action.

The following steps are needed:

1. add a `staticwebapp.config.json` file
2. add your `login` route to your favorite identity provider
3. disable any other provider
4. add a logout route
5. set the `/*` route to `authenticated` allowedRoles
6. add a responseOverride status code `401` to `/login`

> For demonstration purposes, Azure AD is supported, and not limited to a tenant. The static web app is also hosted on a free plan, excluding the use of a custom identity provider.

If role management is desired, users need to be [invited](https://docs.microsoft.com/en-us/azure/static-web-apps/authentication-authorization?tabs=invitations#role-management), or you need to upgrade the plan to standard, to benefit from [custom provider and programmatic role capabilities](https://docs.microsoft.com/en-us/azure/static-web-apps/authentication-authorization?tabs=function#role-management).


## Configuration

The main configuration (routes, disabling authentication providers, etc) can be found on [docs.microsoft.com](https://docs.microsoft.com/en-us/azure/static-web-apps/authentication-authorization?tabs=invitations).

Below are extracts from the `staticwebapp.config.json` file.

### Routes

The route section contains the following elements:

- login
- disable other providers by pointing them to 404
- logout
- set allowedRole for * wildcard to authenticated

``` json

routes: [
    //add the Azure AD provider
    {
        "route": "/login",
        "rewrite": "/.auth/login/aad",
        "allowedRoles": ["anonymous", "authenticated"]
    },
    // disable github
    {
        "route": "/.auth/login/github",
        "statusCode": 404
    },
    //disable twitter
    {
        "route": "/.auth/login/twitter",
        "statusCode": 404
    },
    //add logout
    {
        "route": "/logout",
        "redirect": "/.auth/logout",
        "allowedRoles": ["anonymous", "authenticated"]
    },
    //make everything else only available to authenticated users
    {
        "route": "/*",
        "allowedRoles": ["authenticated"]
    }
]

```

### Overrides

The only thing remaining is to redirect anonymous users to the login page, whenever they hit a page.

```json

//add a response override for not authorized to the login redirect
"responseOverrides": {
    "401": {
        "redirect": "/login",
        "statusCode": 302
    }
}

```

### Basic HTML page

The basic page served makes use of the `./auth/me` endpoint to personalize the greeting.
More information about the `direct access endpoint` can be found in the [official documentation](https://docs.microsoft.com/en-us/azure/static-web-apps/user-information?tabs=javascript#direct-access-endpoint).

*The javascript code might not be the best, but serves its purpose.

``` javascript

 async function getUserInfo() {
            const response = await fetch('/.auth/me');
            const payload = await response.json();
            const {
                clientPrincipal
            } = payload;
            return clientPrincipal.userDetails;
        }

        function setUserDetails(details) {
            const name = document.getElementById('name');
            name.innerHTML = details;
        }

        getUserInfo().then(
            function(value) {
                setUserDetails
                    (value)
            },
            function(error) {
                setUserDetails
                    (error)
            }
        );

```
