# Trellis Matomo

> Installing Matomo for Trellis (via Ansible)

## Introduction

This Ansible-role installs the latest on-premise version of [Matomo](https://matomo.org/), the GDPR- and CCPA-compliant web analytics tool.

## Configuration

### Group vars

You define the following credentials for Matomo in your `group_vars/*/wordpress_sites.yml`:

Variable | Value / Comment
--- | ---
`db.user` | The database-user you want to create for the Matomo DB
`db.host` | Should be `localhost` if you're running with the Trellis default
`path` | *optional* – The path where Matomo should be reachable within your website, f.ex. **example.com/analytics**

Example:

```YML
    matomo:
      db:
        user: matomo
        host: localhost
      path: analytics
```

### Deployments

In order to make sure that every new release has a corresponding symlink set that points towards the Matomo installation, add the following [deploy hook](https://roots.io/trellis/docs/deployments/#hooks):

`deploy-hooks/share-after.yml`

```YML
- name: Create symlink to Matomo
  file:
    path: "{{ deploy_helper.new_release_path }}/{{ item.value.public_path | default('web') }}/{{ item.value.matomo.path | default('matomo') }}"
    src: "{{ addons_dir }}/{{ matomo_dir }}"
    state: link
  with_items: "{{ wordpress_sites }}"
```

## Setup & Connection

Follow the installation instructions that are being output via **trellis-matomo : Explain next steps**.

In order to hook up the newly created Matomo-instance to your WordPress (multi-)site, make sure to install the [Connect Matomo](https://wordpress.org/plugins/wp-piwik/)-plugin:

`composer require wpackagist-plugin/wp-piwik`

### Known limitations

- This role currently does not cater for subdomain installations (matomo.example.com)
