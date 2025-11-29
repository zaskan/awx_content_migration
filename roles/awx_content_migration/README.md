# AWX / AAP Content Migration Role

This Ansible role automates **exporting and importing content** between
two Red Hat Ansible Automation Platform (AAP) or AWX instances.\
It supports migration of most major Controller objects including
organizations, teams, users, credentials, templates, and more.

------------------------------------------------------------------------

## Overview

The role processes a list of content categories (users, credentials,
projects, inventories, etc.) and migrates them from a **source
Controller** to a **target Controller**.

Each content type is handled in a dedicated task file under `tasks/`,
making the role modular, maintainable, and easy to extend.

Typical use cases:

-   Migrating AAP configuration between environments (dev → test → prod)
-   Backup and restore of Controller definitions
-   Bulk movement of content across isolated clusters
-   Standardizing Controller configuration across multiple sites

------------------------------------------------------------------------

## Role Structure

    awx_content_migration/
    ├── tasks/
    │   ├── credential_types.yml
    │   ├── credentials.yml
    │   ├── execution_environments.yml
    │   ├── notification_templates.yml
    │   ├── organizations.yml
    │   ├── teams.yml
    │   ├── inventories.yml
    │   ├── projects.yml
    │   ├── inventory_sources.yml
    │   ├── job_templates.yml
    │   ├── workflow_job_templates.yml
    │   ├── schedules.yml
    │   ├── applications.yml
    │   ├── users.yml
    │   └── main.yml   ← orchestrates all migration tasks

The `main.yml` orchestrates migration by conditionally including each
component based on variables provided by the user.

------------------------------------------------------------------------

## How It Works

1.  **Detection Phase:**\
    The role checks which content types (users, orgs, credentials, etc.)
    are defined in the calling playbook.

2.  **Selective Task Inclusion:**\
    Only the task files corresponding to defined content types are
    executed.\
    Example:

    ``` yaml
    - name: Move Users
      ansible.builtin.include_tasks: users.yml
      when: users is defined and users | length > 0
    ```

3.  **Content Processing:**\
    Each task file generally performs:

    -   Export from the **source** Controller\
    -   (Optional) transformation / filtering\
    -   Import to the **target** Controller

4.  **Role Modularity:**\
    Every component is isolated in its own YAML file for clarity and
    maintainability.

------------------------------------------------------------------------

## Required Variables

The role expects connection details for both source and target
Controller instances.

### Source Controller

``` yaml
source_aap_hostname: ""
source_aap_username: ""
source_aap_password: ""
source_aap_validate_certs: 
```

### Target Controller

``` yaml
target_aap_hostname: ""
target_aap_username: ""
target_aap_password: ""
target_aap_validate_certs: 
```

### Content Selection Variables

Each content type must be explicitly defined as a list.\
Undefined or empty lists skip that section.

``` yaml
organizations:
  - Default
  - MyOrg

users:
  - admin
  - user1

credentials:
  - "SSH Credential"
  - "Vault Credential"
```

------------------------------------------------------------------------

## Supported Content Types

  -------------------------------------------------------------------------
  Task File                      Description
  ------------------------------ ------------------------------------------
  `credential_types.yml`         Migrates custom credential types

  `credentials.yml`              Migrates credentials

  `execution_environments.yml`   Migrates execution environments

  `notification_templates.yml`   Migrates notification templates

  `organizations.yml`            Migrates organizations

  `teams.yml`                    Migrates teams

  `inventories.yml`              Migrates inventory definitions

  `projects.yml`                 Migrates project definitions

  `inventory_sources.yml`        Migrates inventory sync sources

  `job_templates.yml`            Migrates job templates

  `workflow_job_templates.yml`   Migrates workflow job templates

  `schedules.yml`                Migrates schedules

  `applications.yml`             Migrates OAuth applications

  `users.yml`                    Migrates users and supports password
                                 overrides
  -------------------------------------------------------------------------

------------------------------------------------------------------------

## Password Replacement

When migrating users, passwords are overrided with "changeme" as original secrets are not exposed by the source api.

------------------------------------------------------------------------

## Example Usage

``` yaml
- name: Migrate content from AAP1 → AAP2
  hosts: localhost
  gather_facts: false

  vars:
    source_aap_hostname: "https://aap1.example.com"
    source_aap_username: "admin"
    source_aap_password: "password"
    source_aap_validate_certs: false

    target_aap_hostname: "https://aap2.example.com"
    target_aap_username: "admin"
    target_aap_password: "password"
    target_aap_validate_certs: false

    organizations:
      - organization1
    users:
      - admin
      - user1
    credentials:
      - "Credential 1"

  roles:
    - awx_content_migration
```

------------------------------------------------------------------------

## Notes

-   Only content explicitly listed in variables will be migrated.
-   Controller API credentials must have full admin access.
-   Job history, artifacts, and analytics data are **not** migrated.
-   This role is designed for configuration/state migration, not runtime
    data.

------------------------------------------------------------------------

## License

MIT --- free to reuse and modify.

------------------------------------------------------------------------

## Support

If you need help extending the role (e.g., decrypting credentials,
adding dependencies, or advanced transforms), feel free to ask!
