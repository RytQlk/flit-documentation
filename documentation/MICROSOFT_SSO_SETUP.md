# Microsoft SSO login setup

Connect Flit to Microsoft Entra ID so employees can sign in with their existing
Microsoft work accounts. Your organization provides the Microsoft application
registration and credentials.

Start with a working Flit installation from the [installation guide](INSTALLATION.md)
and access to an Entra administrator who can register applications and approve
permissions. These instructions use the files in your **extracted Flit release**;
the private source repository is not required.

The screenshots use a separate demo named `my-flit-gateway`. Its localhost port
`4015` is illustrative; use your actual Flit URL (port `4000` for the local
release examples below). When Flit runs on a VM and the browser runs on a client
machine, use the VM's browser-accessible HTTPS hostname, not `localhost`.

## 1. Register the Microsoft application

Sign in to the [Azure portal](https://portal.azure.com), open **Microsoft Entra ID**,
and select your organization's directory. Open **App registrations > New registration**.

![Open Microsoft Entra ID in Azure services](images/sso/01-entra.png)

| Field | Enter |
| --- | --- |
| Name | A recognizable name, such as `my-flit-gateway` |
| Supported account types | **Accounts in this organizational directory only** / **Single tenant** |
| Redirect platform | **Web** |
| Redirect URI | `https://<your-flit-hostname>/auth/sso/callback` |

Replace `<your-flit-hostname>` with the hostname employees use to open Flit.
For local testing, use `http://localhost:4000/auth/sso/callback` instead.

![Register the application: name, single tenant, Web callback, and Register](images/sso/02-registration.png)

Select **Register**. If the redirect field is not shown during registration,
add it under **Authentication > Add Redirect URI > Web** afterward. Leave
implicit grant and public client flows disabled.

![Keep implicit grant and public client flows disabled](images/sso/09-auth-settings.png)

![Confirm the Web callback; the screenshot uses demo port 4015](images/sso/08-callback.png)

On the application's **Overview**, copy the **Directory (tenant) ID** and
**Application (client) ID**. These are the two IDs Flit needs.

![Copy the application client ID and directory tenant ID](images/sso/03-application-ids.png)

Microsoft reference: [App registration](https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app).

## 2. Create the client secret

Open **Certificates & secrets > Client secrets > New client secret**. Enter a
description, select an expiry according to your organization's policy, and
select **Add**.

![Client secret description and expiry fields](images/sso/04-secret-form.png)

Copy the secret's **Value** immediately to your approved secret storage.
**Use the Value, not the Secret ID.** It is displayed only when created.
Flit's current login integration uses this client secret.

Microsoft reference: [Application credentials](https://learn.microsoft.com/en-us/entra/identity-platform/how-to-add-credentials).

## 3. Approve login permissions and access

Under **API permissions > Add a permission > Microsoft Graph > Delegated
permissions**, add the OpenID permissions `openid`, `profile`, `email`, and
`offline_access` where listed.

![Select the four delegated OpenID scopes and add permissions](images/sso/05-graph-scopes.png)

> **Current release requirement:** Flit's login flow also requests
> `https://search.azure.com/user_impersonation`. Add delegated
> **user_impersonation** for **Azure AI Search** (also shown as **Azure Cognitive
> Search**) through **Add a permission > APIs my organization uses**. No search
> resource configuration is needed for this login setup. The extra scope is
> currently required by Flit's login implementation and cannot be removed with
> an environment setting.

![Select Azure Cognitive Search delegated user_impersonation; checkbox shown before selection](images/sso/06-search-scope.png)

Select the checkbox shown above, then **Add permissions**.

Have the authorized administrator review the permissions and select
**Grant admin consent for your organization**.

![Grant admin consent and verify the green Granted status](images/sso/07-consent.png)

To allow only assigned employees to sign in:

1. Open **Entra ID > Enterprise applications** and select your Flit application.
2. Under **Properties**, keep sign-in enabled, set **Assignment required?** to
   **Yes**, and save.
3. Under **Users and groups > Add user/group**, select the existing employees
   who need access and select **Assign**.

Microsoft reference: [Application access](https://learn.microsoft.com/en-us/entra/identity-platform/howto-restrict-your-app-to-a-set-of-users).

## 4. Configure Flit

Open a terminal in the **extracted release root**, the folder containing
`scripts` and `src`. Run:

```powershell
python scripts/setup_runtime_secrets.py
```

On macOS or Linux, use `python3` if needed. Keep the existing database settings,
volume names, and other deployment choices. Answer **Yes** to **Configure
Microsoft Entra SSO**, then enter:

| Setup prompt | Value |
| --- | --- |
| Microsoft tenant ID | Directory (tenant) ID from step 1 |
| Microsoft client ID | Application (client) ID from step 1 |
| Microsoft redirect URI | The exact Web redirect URI registered in Microsoft |
| Post-login URI | `https://<your-flit-hostname>/chat` |
| Microsoft client secret | The secret **Value** from step 2 |

Setup writes the IDs and URLs to `src/deployment/docker/.env` and stores the
secret in `src/deployment/docker/secrets/microsoft_client_secret` by default.
It enables `overlays/docker-compose.sso.yml` to mount that secret into Flit.
Existing secret files are preserved; setup does not replace an old client secret.

**Do not put the client secret or employee passwords in `.env`.** You do not need
to obtain or save ID tokens, access tokens, or refresh tokens yourself.

Open `src/deployment/docker/.env` and check these settings:

```dotenv
MICROSOFT_TENANT_ID=<your-tenant-id>
MICROSOFT_CLIENT_ID=<your-client-id>
MICROSOFT_REDIRECT_URI=https://<your-flit-hostname>/auth/sso/callback
MICROSOFT_POST_LOGIN_REDIRECT_URL=https://<your-flit-hostname>/chat
SSO_COOKIE_SECURE=true
```

For local HTTP testing, use these three values instead:

```dotenv
MICROSOFT_REDIRECT_URI=http://localhost:4000/auth/sso/callback
MICROSOFT_POST_LOGIN_REDIRECT_URL=http://localhost:4000/chat
SSO_COOKIE_SECURE=false
```

Keep each setting on its own line. The callback must exactly match Microsoft's
registered URL. `/chat` is the destination after sign-in, not a second callback.
For HTTPS deployments, keep your existing TLS configuration and required Compose
overlays; the setup tool does not configure certificates.

## 5. Apply the settings and sign in

From the extracted release root:

```powershell
python scripts/setup_runtime_secrets.py --validate-only
cd src/deployment/docker
docker compose config --quiet
docker compose up -d --no-deps --force-recreate --wait web
```

If the installation uses a custom Compose project name, use the same
`-p <project-name>` on each Compose command above. For example, use
`docker compose -p flit-customer-test config --quiet` for that test installation.
Use the same secret directory as your installation if it differs from the default.

Open Flit in a private browser window and select **Continue with Microsoft**.
Sign in with an assigned work account and complete any Microsoft verification
prompts. Confirm that you return to Flit successfully.

Microsoft sign-in does not automatically grant Flit administrator privileges.
No image build or database migration is needed to enable SSO on an already
working installation.
