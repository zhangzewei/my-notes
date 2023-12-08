# Practice with using microsoft OAuth in react hook
> Author: markzzw | Date: 2023-12-07

OAuth is helpful to let developers get user information easily, here I will show you the usages with the microsoft OAuth in react hook way, let's start the travel.

## Config in Azure
    
### Register your app
![](https://learn.microsoft.com/en-us/entra/identity-platform/media/single-page-app-tutorial-01-register-app/register-application.png)
![](https://learn.microsoft.com/en-us/entra/identity-platform/media/single-page-app-tutorial-01-register-app/record-identifiers.png)

**Note: remenber the application ID and Directory ID we will use them in the next step**

## Set up your react project

### Node modules
fisrt we need to install the dependencies.

```shell
npm install @azure/msal-browser
npm install @azure/msal-react
```

### Create a msconfig.js file

msconfig.js
```js
export const msalConfig = {
    auth: {
        clientId: "Enter_the_Application_Id_Here", // application id
        authority: "https://login.microsoftonline.com/Enter_the_Tenant_Info_Here", // directory id
        redirectUri: "http://localhost:3000", // after auth success, will redirect in this uri
    },
    cache: {
        cacheLocation: "sessionStorage", // This configures where your cache will be stored
    },
}
```

### Create a microsoft auth provider
If we want to use it in react hook way, we need to create a provider.

MsAuthProvider.jsx
```jsx
import * from "react";
import { MsalProvider } from '@azure/msal-react';
import { PublicClientApplication } from '@azure/msal-browser';
import { msalConfig } from './msconfig.js';

// create msal instance and export it for using.
export const msalInstance = new PublicClientApplication(msalConfig);

export default function MsAuthProvider({ children }) {
    return <MsalProvider instance={msalInstance}>
        {children}
    </MsalProvider>
}
```

### Use MsAuthProvider to warp your react application

In the App.jsx, we need to do the refactor as the follow.

```jsx
import * from 'react';
import { AuthenticatedTemplate, UnauthenticatedTemplate } from '@azure/msal-react';
import App from './App.jsx'
import MsAuthProvider from './msAuthProvider/index.jsx';
import SignIn from './pages/signin/index.jsx';

ReactDOM.createRoot(document.getElementById('root')!).render(
    <MsAuthProvider>
        <AuthenticatedTemplate>
          <App />
        </AuthenticatedTemplate>
        <UnauthenticatedTemplate>
          <SignIn />
        </UnauthenticatedTemplate>
    </MsAuthProvider>
)
```

We can use the `AuthenticatedTemplate` and `UnauthenticatedTemplate` provided by the `@azure/msal-react`, it's helpful to control of the displaying component, we don't need to handle the authrized status, `MsAuthProvider` will handle this for the app.

### Create SignIn component

SignIn component index.jsx

```jsx
import * from 'react';
import { Card, Typography, CardBody, CardFooter, Button } from "@material-tailwind/react";
import { useSelector } from "react-redux";
import { useMsal } from "@azure/msal-react";

export default function SignIn() {
    const handleLogin = () => {
        // this will open a popup for signin, after singin popup will be close, then
        // app will redirect to the redirectUri as the config in msalConfig
        instance.loginPopup({
            scopes: ["User.Read"]
        }).catch((e) => console.log(e));
    };

    return (
        <div className="flex justify-center mt-32">
            <Card className="w-96">
                <CardBody className="flex flex-col gap-4 place-items-center">
                    <Typography variant="h3">
                        Sign In
                    </Typography>
                </CardBody>
                <CardFooter className="pt-0">
                    <Button
                        className="bg-blue-900"
                        fullWidth
                        onClick={handleLogin}
                    >
                        Sign In With MicroSoft
                    </Button>
                </CardFooter>
            </Card>
        </div>
    )
}
```

![](https://learn.microsoft.com/en-us/entra/identity-platform/media/single-page-app-tutorial-04-call-api/pick-account.png)
![](https://learn.microsoft.com/en-us/entra/identity-platform/media/single-page-app-tutorial-04-call-api/enter-code.png)
### Get user information

profile.jsx

```jsx
import *, { useEffect } from "react";
import { useMsal } from '@azure/msal-react';

export default function Profile() {
    const [profileData, setProfileData] = useState(null);
    const { instance, accounts } = useMsal();
    const getProfile = async () => {
        // get access token by instance, this instance is provided by MsAuthProvider
        const accessToken = await instance.acquireTokenSilent({
            scopes: ["User.Read"]
            account: accounts[0],
        }).then((response) => response.accessToken);
        const headers = new Headers();
        const bearer = `Bearer ${accessToken}`;
        headers.append("Authorization", bearer);
        const options = {
            method: "GET",
            headers: headers
        };
        // fetch user information
        fetch("https://graph.microsoft.com/v1.0/me", options)
            .then(res => res.json())
            .then((res) => setProfileData(res))
    }

    useEffect(() => {
        if (!profileData) {
            getProfile();
        }
    }, [profileData]);

    return (
        <div className="flex flex-col items-center">
            {
                profileData && <div className="flex flex-col items-center">
                    <p>
                        <strong>First Name: </strong> {profileData.givenName}
                    </p>
                    <p>
                        <strong>Last Name: </strong> {profileData.surname}
                    </p>
                    <p>
                        <strong>Email: </strong> {profileData.userPrincipalName}
                    </p>
                    <p>
                        <strong>Id: </strong> {profileData.id}
                    </p>
                </div>
            }
            <button onClick={getProfile}>Get user information</button>
        </div>
    );
};
```
Here we need to get access token so we use the `["User.Read"]` as the scpoes param, if you want to get id token, you should use the `['openid']` as the scpoes param.


### Integration with axios
Axios has `interceptors` can help us to add some configuration before request, let's integrate axios for the requests.

request.js
```js
import axios from 'axios'
import { msalInstance }  from './msAuthProvider/index.jsx';
const request = axios.create({
  baseURL: '/'
})
// add 
request.interceptors.request.use(async config => {
    // msalInstance is exported by msAuthProvider file, because we can not use the hook here
    const accounts = msalInstance.getAllAccounts();
    if (accounts.length > 0) {
        const accessToken = await instance.acquireTokenSilent({
            scopes: ["User.Read"]
            account: accounts[0],
        }).then((response) => response.accessToken);
        if (accessToken) {
            config.headers.Authorization = `Bearer ${accessToken}`
        }
    }
    return config;
}, function (error) {
    return Promise.reject(error);
});

export default request;
```

Then we can use axios request like this.

Refactor profile.jsx

```jsx
import *, { useEffect } from "react";
import { useMsal } from '@azure/msal-react';
import request from './request.js'

export default function Profile() {
    const [profileData, setProfileData] = useState(null);
    const { instance, accounts } = useMsal();
    const getProfile = () => {
        // replce fetch by axios request
        request.get("https://graph.microsoft.com/v1.0/me")
            .then((res) => setProfileData(res))
    }

    useEffect(() => {
        if (!profileData) {
            getProfile();
        }
    }, [profileData]);

    return (
        <div className="flex flex-col items-center">
            {
                profileData && <div className="flex flex-col items-center">
                    <p>
                        <strong>First Name: </strong> {profileData.givenName}
                    </p>
                    <p>
                        <strong>Last Name: </strong> {profileData.surname}
                    </p>
                    <p>
                        <strong>Email: </strong> {profileData.userPrincipalName}
                    </p>
                    <p>
                        <strong>Id: </strong> {profileData.id}
                    </p>
                </div>
            }
            <button onClick={getProfile}>Get user information</button>
        </div>
    );
};
```
## Reference
1. [Migration Guide from msal v1 to @azure/msal-react and @azure/msal-browser](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-react/docs/migration-guide.md)
2. [Tutorial: Register a Single-page application with the Microsoft identity platform](https://learn.microsoft.com/en-us/entra/identity-platform/tutorial-single-page-app-react-register-app)
3. [Axios Interceptor](https://axios-http.com/docs/interceptors)