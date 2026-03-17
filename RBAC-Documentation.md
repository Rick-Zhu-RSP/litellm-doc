# LiteLLM Role-Based Access Control (RBAC) Documentation

## Overview

LiteLLM's Role-Based Access Control (RBAC) system allows you to manage permissions for users, teams, and organizations within the LiteLLM Proxy. It defines what actions different roles can perform and what resources they can access, such as API keys, spend data, and specific LLM models or routes. The system works by assigning roles to users, which then dictate their access to various functionalities and data within the proxy.

## How RBAC Works

The RBAC system in LiteLLM operates by checking a user's assigned role against a set of defined permissions for routes and models. When a request is made to the LiteLLM Proxy, the system identifies the user's role, typically from a JWT token. It then verifies if that role has the necessary permissions to access the requested route or model.

### Key Components

- **JWTAuthManager.can_rbac_role_call_route**: Method that checks if a user's role is allowed to access a specific route
- **JWTAuthManager.can_rbac_role_call_model**: Method that verifies if the role can access a particular model
- **HTTPException (403)**: Raised when a role is not permitted to access a resource

### Enforcement

If `enforce_rbac` is set to `True` in the `litellm_jwtauth` configuration, an unmatched token (one not belonging to a proxy admin, team, or user) will result in a 403 Forbidden error.

### Scope-Based Access Control

LiteLLM also supports scope-based access control, where JWT scopes are mapped to allowed models. The `JWTAuthManager.check_scope_based_access` method ensures that the requested model is within the scopes granted to the user.

## Available Roles

LiteLLM defines several roles, categorized into Global Proxy Roles and Organization/Team Specific Roles. These roles are enumerated in the `LitellmUserRoles` class.

### Global Proxy Roles

- **PROXY_ADMIN**: Has full control over the entire platform, including managing organizations, teams, users, spend, and API keys
- **PROXY_ADMIN_VIEW_ONLY**: Can view all keys and spend across the platform but cannot make any modifications
- **INTERNAL_USER**: Can login, view, create, and delete their own keys, and view their own spend
- **INTERNAL_USER_VIEW_ONLY**: (Deprecated) Can view their own keys and spend but cannot create or delete keys

### Organization/Team Specific Roles

These are premium features:

- **ORG_ADMIN**: Admin over a specific organization, able to create teams and users within that organization
- **TEAM**: Used for JWT authentication and represents a team scope
- **CUSTOMER**: Represents external users

## Configuration

RBAC is primarily configured within the `general_settings` section of the LiteLLM Proxy's `config.yaml` file, specifically under `litellm_jwtauth`.

### Enabling RBAC

To enforce RBAC, you must set `enforce_rbac: true` within the `litellm_jwtauth` configuration.

### Role-Based Permissions

You can define default model and endpoint permissions for each role using `role_permissions` in `general_settings`. This is validated during config loading by creating `RoleBasedPermissions` objects.

#### Example Configuration

Example configuration for a `team` role allowing access to `anthropic-claude` model and `/v1/chat/completions` route:

```yaml
general_settings:
  enable_jwt_auth: True
  litellm_jwtauth:
    object_id_jwt_field: "oid"
    roles_jwt_field: "roles"
    role_mappings:
      - role: litellm.api.consumer
        internal_role: "team"
    enforce_rbac: true
  role_permissions:
    - role: team
      models: ["anthropic-claude"]
      routes: ["/v1/chat/completions"]
```

### JWT Field Configuration

LiteLLM uses fields within JWT tokens to determine user and team identities and roles. Key fields include:

- **object_id_jwt_field**: Specifies the JWT field containing the user or team ID
- **roles_jwt_field**: Specifies the JWT field containing the user's roles
- **team_ids_jwt_field**: Field containing a list of team IDs the user belongs to
- **user_email_jwt_field**: Field containing the user's email
- **end_user_id_jwt_field**: Field for end-user ID for cost tracking

These fields support dot notation for nested claims.

### Role Mappings

You can map external JWT roles to internal LiteLLM roles using `role_mappings`.

```yaml
litellm_jwtauth:
  role_mappings:
    - role: litellm.api.consumer
      internal_role: "team"
```

Supported internal roles for mapping are `team`, `internal_user`, and `proxy_admin`.

### Scope-Based Model Access

To control model access based on JWT scopes, set `enforce_scope_based_access: true` and define `scope_mappings`.

```yaml
litellm_jwtauth:
  scope_mappings:
    - scope: litellm.api.consumer
      models: ["anthropic-claude"]
    - scope: litellm.api.gpt_3_5_turbo
      models: ["gpt-3.5-turbo-testing"]
  enforce_scope_based_access: true
```

### Syncing User Roles and Teams with IDP

LiteLLM can automatically sync user roles and team memberships from an Identity Provider (IDP) to its database. This is enabled by setting `sync_user_role_and_teams: true` and configuring `jwt_litellm_role_map`.

```yaml
litellm_jwtauth:
  user_id_jwt_field: "sub"
  team_ids_jwt_field: "groups"
  roles_jwt_field: "roles"
  user_id_upsert: true
  sync_user_role_and_teams: true
  jwt_litellm_role_map:
    - jwt_role: "ADMIN"
      litellm_role: "proxy_admin"
    - jwt_role: "USER"
      litellm_role: "internal_user"
```

This feature ensures that LiteLLM roles and team memberships remain consistent with the IDP.

## Implementation Notes

- The RBAC system is primarily implemented within the LiteLLM Proxy and relies heavily on JWT authentication for identifying user roles and permissions
- While the `Router` class handles load balancing and routing, the RBAC logic is distinct and focuses on authorization before routing occurs
- The `LitellmUserRoles` enum defines the available roles, and the `JWTAuthManager` class contains the core logic for checking role-based access to routes and models
- Some advanced RBAC features, such as organization/team-specific roles and IDP synchronization, are premium features

## Related Resources

- [Router and Load Balancing (BerriAI/litellm)](https://deepwiki.com/wiki/BerriAI/litellm#2.3)
- [View this search on DeepWiki](https://deepwiki.com/search/what-is-the-rolebased-access-c_35bd33e1-41b0-40a7-8e88-2e4cf334210c)

---

*This documentation was generated from the LiteLLM repository using DeepWiki.*