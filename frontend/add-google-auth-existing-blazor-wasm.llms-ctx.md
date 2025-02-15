# Adding Google Authentication to an ASP.NET Core 8.0 Blazor WASM Standalone Application

This guide walks you through adding Google Authentication to an ASP.NET Core 8.0 Blazor WebAssembly (WASM) Standalone application. It covers both the Google API Console setup and the Blazor WASM integration.

## Prerequisites
- .NET 8.0 SDK installed.
- A Blazor WASM Standalone project created.
- A Google account (for API Console access).

## Step 1: Set Up Google API Console

### 1.1 Create a Google API Console Project

1. Go to the Google API Console.
1. Click on Select a project (top-left dropdown) → New Project.
1. Enter a project name (e.g., BlazorWasmGoogleAuth) and click Create.

### 1.2 Configure the OAuth Consent Screen

1. In the left sidebar, go to APIs & Services → OAuth consent screen.
1. Select External (for development) or Internal (for G Suite accounts).
1. Fill in the required fields:
    1. App name: Your app's name (e.g., Blazor WASM App).
    1. User support email: Your email address.
    1. Developer contact information: Your email address.
1. Click Save and Continue.

### 1.3 Create OAuth 2.0 Credentials

1. In the left sidebar, go to Credentials.
1. Click Create Credentials → OAuth 2.0 Client ID.
1. Configure the OAuth client:
    1. Application type: Web application.
    1. Name: Blazor WASM Client.
    1. Authorized JavaScript origins:
        1. Add https://localhost:5001 (for development).
        1. Add your production domain (e.g., https://yourapp.com).
    1. Authorized redirect URIs:
        1. Add https://localhost:5001/authentication/login-callback (for development).
        1. Add https://yourapp.com/authentication/login-callback (for production).
1. Click Create.
1. Note down the Client ID and Client Secret (you’ll need the Client ID for Blazor WASM).

## Step 2: Update Blazor WASM Project

### 2.1 Install Required NuGet Package

1. Open your Blazor WASM project in the terminal.

1. Run the following command to install the authentication package:

    ```bash
    dotnet add package Microsoft.AspNetCore.Components.WebAssembly.Authentication
    ```

### 2.2 Configure Authentication in Program.cs

1. Open the Program.cs file.

1. Add the following code to configure Google Authentication:

    ```csharp
    using Microsoft.AspNetCore.Components.WebAssembly.Hosting;
    using Microsoft.Extensions.DependencyInjection;
    using System;
    using System.Net.Http;
    using System.Threading.Tasks;

    var builder = WebAssemblyHostBuilder.CreateDefault(args);
    builder.RootComponents.Add<App>("#app");

    builder.Services.AddScoped(sp => new HttpClient { BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) });

    // Add authentication services
    builder.Services.AddOidcAuthentication(options =>
    {
        options.ProviderOptions.Authority = "https://accounts.google.com";
        options.ProviderOptions.ClientId = builder.Configuration["Google:ClientId"];
        options.ProviderOptions.ResponseType = "code";
        options.ProviderOptions.DefaultScopes.Add("openid");
        options.ProviderOptions.DefaultScopes.Add("profile");
        options.ProviderOptions.DefaultScopes.Add("email");
    });

    await builder.Build().RunAsync();
    ```

### 2.3 Add Google Client ID to appsettings.json

1. Open the appsettings.json file.
1. Add the Google Client ID:

    ```json
    {
    "Google": {
        "ClientId": "YOUR_GOOGLE_CLIENT_ID"
    }
    }
    ```

1. Replace YOUR_GOOGLE_CLIENT_ID with the Client ID from the Google API Console.

### 2.4 Update index.html

1. Open the wwwroot/index.html file.
1. Add the authentication script before the closing </body> tag:

    ```html
    <script src="_content/Microsoft.AspNetCore.Components.WebAssembly.Authentication/AuthenticationService.js"></script>
    ```


## Step 3: Add Login and Logout Components

### 3.1 Create LoginDisplay.razor

1. Create a new Razor component named LoginDisplay.razor.
1. Add the following code:

    ```razor
    @using Microsoft.AspNetCore.Components.Authorization
    @using Microsoft.AspNetCore.Components.WebAssembly.Authentication
    @inject NavigationManager Navigation

    <AuthorizeView>
        <Authorized>
            <a href="javascript:void(0)" @onclick="BeginSignOut">Log out</a>
        </Authorized>
        <NotAuthorized>
            <a href="authentication/login">Log in</a>
        </NotAuthorized>
    </AuthorizeView>

    @code {
        private void BeginSignOut()
        {
            Navigation.NavigateTo("authentication/logout");
        }
    }
    ```

### 3.2 Add LoginDisplay to MainLayout.razor

1. Open the MainLayout.razor file.
1. Add the LoginDisplay component where you want the login/logout UI to appear:

    ```razor
    <div class="top-row px-4">
        <LoginDisplay />
    </div>
    ```

## Step 4: Add Authentication Routes

### 4.1 Update App.razor

1. Open the App.razor file.
1. Add the following code to handle authentication routes:

    ```razor
    <CascadingAuthenticationState>
        <Router AppAssembly="@typeof(App).Assembly">
            <Found Context="routeData">
                <AuthorizeRouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
            </Found>
            <NotFound>
                <LayoutView Layout="@typeof(MainLayout)">
                    <p>Sorry, there's nothing at this address.</p>
                </LayoutView>
            </NotFound>
        </Router>
    </CascadingAuthenticationState>
    ```

## Step 5: Add "Log In with Google" Button to the / Route

### 5.1 Update the Home Page (Index.razor)

1. Open the Pages/Index.razor file.
1. Add the "Log In with Google" button for unauthenticated users:

```razor
@page "/"
@using Microsoft.AspNetCore.Components.Authorization
@inject NavigationManager Navigation

<AuthorizeView>
    <Authorized>
        <h3>Welcome back!</h3>
        <p>You are logged in. Start using the app.</p>
    </Authorized>
    <NotAuthorized>
        <h3>Welcome</h3>
        <p>Please log in to continue.</p>
        <button class="btn btn-primary" @onclick="RedirectToLogin">Log In with Google</button>
    </NotAuthorized>
</AuthorizeView>

@code {
    private void RedirectToLogin()
    {
        Navigation.NavigateTo("authentication/login");
    }
}
``` 

## Step 6: Run and Test

1. Run the app:

```bash
dotnet run
```

1. Navigate to the app in your browser (e.g., https://localhost:5001).
1. Click Log in and authenticate with your Google account.
1. Verify that you are redirected back to the app and logged in.

## Step 67: Deploy to Production

1. Update the Authorized JavaScript origins and Authorized redirect URIs in the Google API Console to match your production domain.
1. Update the appsettings.json file with the production Client ID.

## Troubleshooting

1. CORS Issues: Ensure the app is running on the correct port (e.g., https://localhost:5001).
1. Invalid Redirect URI: Double-check the redirect URI in the Google API Console.
1. Authentication Errors: Check the browser console for errors and ensure the ClientId is correct.

## Conclusion

You’ve successfully added Google Authentication to your Blazor WASM Standalone application! Share this guide with your coworker, and let me know if you encounter any issues. 😊