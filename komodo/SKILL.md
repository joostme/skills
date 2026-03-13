---
name: komodo
description: Access a Komodo Core instance through its JSON API using API keys, API secrets, or JWT auth. Use this whenever the user wants to inspect Komodo resources, manage builds, deployments, stacks, repos, servers, syncs, users, or execute operational actions like build, deploy, restart, pull, prune, or fetch logs from Komodo.
compatibility: opencode
---

## What I do

- Access a Komodo Core instance through its HTTP JSON API.
- Work with the main request groups: `auth`, `user`, `read`, `write`, and `execute`.
- Prefer safe operational workflows: inspect first, then update or execute.
- Avoid relying on the websocket terminal API unless the user explicitly asks, because the published docs mark that area as not yet documented.

## Required environment variables

- `KOMODO_ADDRESS`: Base URL for the Komodo Core API, for example `https://komodo.example.com`.
- Preferred machine auth:
  - `KOMODO_API_KEY`
  - `KOMODO_API_SECRET`
- Optional token auth:
  - `KOMODO_JWT`

## Authentication rules

- Do not proceed with authenticated calls unless `KOMODO_ADDRESS` is set and one of these auth modes is available:
  - `KOMODO_JWT`, or
  - both `KOMODO_API_KEY` and `KOMODO_API_SECRET`
- Normalize the base URL by removing a trailing slash.
- Use exactly one auth mode per request.
- Prefer API key and secret for automation and scripted sessions.
- Use JWT only when it is already available or when the user explicitly wants a login-based flow.

## Base conventions

- All documented Komodo API calls use `POST`.
- Request paths are grouped by module:
  - `/auth`
  - `/user`
  - `/read`
  - `/write`
  - `/execute`
- Every request body has this shape:

```json
{
  "type": "RequestType",
  "params": {
    "field": "value"
  }
}
```

- Send `Content-Type: application/json` on every request.
- Authentication headers:
  - JWT: `Authorization: <token>`
  - API key mode: `X-Api-Key: <key>` and `X-Api-Secret: <secret>`
- Error responses use this common shape:

```json
{
  "error": "top level error message",
  "trace": [
    "first traceback message",
    "second traceback message"
  ]
}
```

## Core workflow

- For reads, send a request to `/read` with a `type` from the read module.
- For mutations, send a request to `/write` with a `type` from the write module.
- For operational actions such as build, deploy, restart, pull, and prune, send a request to `/execute`.
- For user self-management such as API key creation, use `/user`.
- For login flows, use `/auth`.

## Read patterns

- List resources with `List...` requests.
- `ListFull...` variants return the complete resource object instead of a summary list item.
- Fetch a specific resource with `Get...` requests.
- Prefer names or ids exactly as accepted by the request type.
- Use summary endpoints when the user wants quick status across a whole resource class.
- Use log endpoints or search endpoints for troubleshooting.
- Most `List...` requests accept an optional `query` object with `tags` (array of tag ids or names) and `tag_behavior` (`"All"` or `"Any"`) for filtering.

### Common read requests

- Core and access
  - `GetVersion`
  - `GetCoreInfo`
  - `GetPermission`
  - `ListPermissions`
- Resources
  - `ListBuilds`, `ListFullBuilds`, `GetBuild`, `GetBuildActionState`, `ListBuildVersions`
  - `ListBuilders`, `ListFullBuilders`, `GetBuilder`, `GetBuildersSummary`
  - `ListDeployments`, `ListFullDeployments`, `GetDeployment`, `GetDeploymentActionState`, `GetDeploymentContainer`, `GetDeploymentStats`, `GetDeploymentLog`, `SearchDeploymentLog`
  - `ListStacks`, `ListFullStacks`, `GetStack`, `GetStackActionState`, `GetStackLog`, `SearchStackLog`, `ListStackServices`
  - `ListRepos`, `ListFullRepos`, `GetRepo`, `GetRepoActionState`
  - `ListServers`, `ListFullServers`, `GetServer`, `GetServerState`, `GetSystemStats`, `GetSystemInformation`, `GetHistoricalServerStats`
  - `ListProcedures`, `ListFullProcedures`, `GetProcedure`, `GetProcedureActionState`
  - `ListActions`, `ListFullActions`, `GetAction`, `GetActionActionState`
  - `ListAlerters`, `ListFullAlerters`, `GetAlerter`, `GetAlertersSummary`
  - `ListResourceSyncs`, `ListFullResourceSyncs`, `GetResourceSync`, `GetResourceSyncActionState`
- Docker and runtime inspection
  - `ListDockerContainers` (requires `server` param), `InspectDockerContainer`
  - `ListDockerImages`, `InspectDockerImage`, `ListDockerImageHistory`
  - `ListDockerNetworks`, `InspectDockerNetwork`
  - `ListDockerVolumes`, `InspectDockerVolume`
  - `ListComposeProjects`
- Providers and registries
  - `ListGitProvidersFromConfig`, `ListGitProviderAccounts`, `GetGitProviderAccount`
  - `ListDockerRegistriesFromConfig`, `ListDockerRegistryAccounts`, `GetDockerRegistryAccount`
- Users and API keys
  - `ListUsers`, `FindUser`, `GetUsername`
  - `ListApiKeys`, `ListApiKeysForServiceUser`
- Audit and metadata
  - `ListUpdates`, `GetUpdate`
  - `ListAlerts`, `GetAlert`
  - `ListTags`, `GetTag`
  - `ListSchedules`
  - `ListSecrets`, `ListVariables`, `GetVariable`
- Export and sync
  - `ExportAllResourcesToToml`
  - `ExportResourcesToToml`

## Write patterns

- Use `Create...`, `Update...`, `Rename...`, `Copy...`, and `Delete...` requests through `/write`.
- Many update requests take partial config objects and merge only the provided fields.
- Prefer partial updates over rewriting full configs when the user only wants a small change.
- Read the current resource first when you need to confirm field names, current values, or avoid accidental drift.

### Common write requests

- Resource lifecycle
  - `CreateBuild`, `UpdateBuild`, `CopyBuild`, `RenameBuild`, `DeleteBuild`
  - `CreateBuilder`, `UpdateBuilder`, `CopyBuilder`, `RenameBuilder`, `DeleteBuilder`
  - `CreateDeployment`, `UpdateDeployment`, `CopyDeployment`, `RenameDeployment`, `DeleteDeployment`
  - `CreateStack`, `UpdateStack`, `CopyStack`, `RenameStack`, `DeleteStack`
  - `CreateRepo`, `UpdateRepo`, `CopyRepo`, `RenameRepo`, `DeleteRepo`
  - `CreateServer`, `UpdateServer`, `CopyServer`, `RenameServer`, `DeleteServer`
  - `CreateProcedure`, `UpdateProcedure`, `CopyProcedure`, `RenameProcedure`, `DeleteProcedure`
  - `CreateAction`, `UpdateAction`, `CopyAction`, `RenameAction`, `DeleteAction`
  - `CreateAlerter`, `UpdateAlerter`, `CopyAlerter`, `RenameAlerter`, `DeleteAlerter`
  - `CreateResourceSync`, `UpdateResourceSync`, `CopyResourceSync`, `RenameResourceSync`, `DeleteResourceSync`
- Meta and config helpers
  - `UpdateResourceMeta`
  - `RefreshBuildCache`
  - `RefreshRepoCache`
  - `RefreshStackCache`
  - `RefreshResourceSyncPending`
  - `WriteBuildFileContents`
  - `WriteStackFileContents`
  - `WriteSyncFileContents`
- Webhooks and providers
  - `CreateBuildWebhook`, `DeleteBuildWebhook`
  - `CreateRepoWebhook`, `DeleteRepoWebhook`
  - `CreateStackWebhook`, `DeleteStackWebhook`
  - `CreateSyncWebhook`, `DeleteSyncWebhook`
  - `CreateDockerRegistryAccount`, `UpdateDockerRegistryAccount`, `DeleteDockerRegistryAccount`
  - `CreateGitProviderAccount`, `UpdateGitProviderAccount`, `DeleteGitProviderAccount`
- Users and permissions
  - `CreateUserGroup`, `RenameUserGroup`, `DeleteUserGroup`
  - `AddUserToUserGroup`, `RemoveUserFromUserGroup`, `SetUsersInUserGroup`, `SetEveryoneUserGroup`
  - `UpdatePermissionOnTarget`, `UpdatePermissionOnResourceType`
  - `CreateLocalUser`, `CreateServiceUser`, `DeleteUser`
  - `CreateApiKeyForServiceUser`, `DeleteApiKeyForServiceUser`
  - `UpdateUserAdmin`, `UpdateUserBasePermissions`, `UpdateServiceUserDescription`
  - `CreateVariable`, `DeleteVariable`, `UpdateVariableValue`, `UpdateVariableDescription`, `UpdateVariableIsSecret`

## Execute patterns

- Use `/execute` for operational actions that cause builds, deploys, restarts, pulls, destroys, syncs, alerts, or docker maintenance.
- Treat execute requests as state-changing and potentially disruptive.
- Confirm destructive actions when ambiguity exists.
- Prefer targeted actions before batch actions unless the user explicitly wants a broad operation.

### Common execute requests

- Build and repo operations
  - `RunBuild`, `CancelBuild`
  - `BuildRepo`, `CloneRepo`, `PullRepo`, `CancelRepoBuild`
  - `BatchRunBuild`, `BatchBuildRepo`, `BatchCloneRepo`, `BatchPullRepo`
- Deployment operations
  - `Deploy`, `PullDeployment`, `StartDeployment`, `StopDeployment`, `RestartDeployment`, `PauseDeployment`, `UnpauseDeployment`, `DestroyDeployment`
  - `BatchDeploy`, `BatchDestroyDeployment`
- Stack operations
  - `DeployStack`, `DeployStackIfChanged`, `PullStack`, `StartStack`, `StopStack`, `RestartStack`, `PauseStack`, `UnpauseStack`, `DestroyStack`, `RunStackService`
  - `BatchDeployStack`, `BatchDeployStackIfChanged`, `BatchPullStack`, `BatchDestroyStack`
- Procedures, actions, and sync
  - `RunProcedure`, `BatchRunProcedure`
  - `RunAction`, `BatchRunAction`
  - `RunSync`, `CommitSync`
- Docker and server maintenance
  - `StartContainer`, `StopContainer`, `RestartContainer`, `PauseContainer`, `UnpauseContainer`, `DestroyContainer`
  - `StartAllContainers`, `StopAllContainers`, `RestartAllContainers`, `PauseAllContainers`, `UnpauseAllContainers`
  - `DeleteImage`, `DeleteNetwork`, `DeleteVolume`
  - `PruneBuildx`, `PruneDockerBuilders`, `PruneContainers`, `PruneImages`, `PruneNetworks`, `PruneVolumes`, `PruneSystem`
- Admin and utility
  - `BackupCoreDatabase`
  - `GlobalAutoUpdate`
  - `ClearRepoCache`
  - `SendAlert`
  - `TestAlerter`
  - `Sleep`

## User and auth patterns

- `CreateApiKey` creates an API key for the calling user through `/user`.
- The API secret is only returned once, so tell the user to store it immediately.
- `DeleteApiKey` removes a previously created key.
- `PushRecentlyViewed` and `SetLastSeenUpdate` are UI helper endpoints available through `/user`.
- `LoginLocalUser` can be used through `/auth` to obtain a JWT for local auth.
- `SignUpLocalUser` registers a new local user through `/auth` (only when local auth and registration are enabled).
- `ExchangeForJwt` exchanges a single-use token for a JWT through `/auth`.
- `GetLoginOptions` is useful before attempting a login flow (sent to `/auth`, no auth required).
- `GetUser` verifies the authenticated user context (sent to `/auth`, not `/user`).

## Example requests

Each example shows the path and JSON body to `POST`. Include auth headers on every request.

- Get the Komodo version. `POST /read`

```json
{"type": "GetVersion", "params": {}}
```

- List deployments. `POST /read`

```json
{"type": "ListDeployments", "params": {"query": {}}}
```

- Get one deployment by name or id. `POST /read`

```json
{"type": "GetDeployment", "params": {"deployment": "api"}}
```

- Search a deployment log for errors. `POST /read`

```json
{"type": "SearchDeploymentLog", "params": {"deployment": "api", "search": "ERROR"}}
```

- Run a build. `POST /execute`

```json
{"type": "RunBuild", "params": {"build": "backend-image"}}
```

- Deploy with optional stop overrides. `POST /execute`

```json
{"type": "Deploy", "params": {"deployment": "api", "stop_signal": "SIGTERM", "stop_time": 30}}
```

- Update only the version field on a build. `POST /write`

```json
{"type": "UpdateBuild", "params": {"id": "67076689ed600cfdd52ac637", "config": {"version": "1.15.9"}}}
```

- Create an API key for the calling user with no expiry. `POST /user`

```json
{"type": "CreateApiKey", "params": {"name": "automation", "expires": 0}}
```

- Login as a local user to get a JWT. `POST /auth`

```json
{"type": "LoginLocalUser", "params": {"username": "myuser", "password": "mypass"}}
```

## Practical request guidance

- When the user asks to inspect a resource, prefer this order:
  - list matching resources
  - resolve the exact name or id
  - fetch the specific resource
- When the user asks to update configuration, prefer this order:
  - fetch the resource first
  - change only the requested fields
  - use partial config updates when supported
- When the user asks to run an operation such as build or deploy:
  - confirm the exact target resource if ambiguous
  - inspect current action state if the target might already be busy
  - execute the action
  - optionally fetch logs or updates afterward
- When the user asks for broad operational status, prefer summary endpoints such as:
  - `GetBuildsSummary`
  - `GetBuildersSummary`
  - `GetDeploymentsSummary`
  - `GetServersSummary`
  - `GetStacksSummary`
  - `GetReposSummary`
  - `GetAlertersSummary`

## Safety rules

- Be conservative with destructive actions such as `Delete...`, `Destroy...`, prune operations, and broad batch executions.
- If the target is ambiguous, resolve the exact resource first.
- For `Update...` requests, avoid sending unrelated fields when a partial config is supported.
- For API key creation, warn that the secret is returned only once.
- For websocket terminal access, note that the public docs say the calling details are still undocumented.

## When to use me

Use this skill whenever the user wants to inspect, troubleshoot, configure, or operate a Komodo Core instance through its API, especially for builds, deployments, stacks, repos, servers, syncs, permissions, alerts, logs, or API key management.
