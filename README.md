# Trellis Matomo

> Installing Matomo for Trellis (via Ansible)

## Installation

This Ansible-role for [Trellis](https://roots.io/trellis) installs the latest on-premise version of [Matomo](https://matomo.org/), the GDPR- and CCPA-compliant web analytics tool.

Add the role to the `galaxy.yml` file of Trellis:

```yaml
- name: trellis-matomo
  src: https://github.com/E-VANCE/trellis-matomo
  type: git
  version: '0.3.1'
```

Run `ansible-galaxy install -r galaxy.yml` (or `trellis galaxy install` is you have [trellis-cli](https://github.com/roots/trellis-cli)) to install the new role.

Then, add the role into `server.yml` and/or `dev.yml`:

```yaml
roles:
    ... other Trellis roles ...
    - { role: trellis-matomo, tags: [matomo]}
```

After adding the role to the above file(s) and running the install, provision your Vagrant box via `vagrant reload --provision` (if it's running) or `vagrant provision` (if it's not). If you haven't provisioned the box yet simply run `vagrant up`.

If you have [trellis-cli](https://github.com/roots/trellis-cli) installed – which is highly recommended – then just use `trellis up` / `trellis vm start` or  `trellis provision {ENV}`.

## Configuration

### Group vars

You define the following credentials for Matomo in your `group_vars/*/wordpress_sites.yml` & `group_vars/*/vault.yml`:

**wordpress_sites.yml**

Variable | Value / Comment
--- | ---
`db.user` | The database-user you want to create for the Matomo DB
`paths.addons` | The path where Matomo should be installed within your Trellis project – will resolve to `/srv/www/website.com/{{ addons_path }}`
`paths.matomo` | The path where Matomo should be reachable within your website, e.g. **website.com/{{ matomo_path }}**

Example:

```yaml
[...]
wordpress_sites:
  website.com:
    site_hosts: [...]
    [...]
    matomo:
      db:
        user: matomo
      paths:
        addons: addons
        matomo: analytics # Used for accessing Matomo via /your-endpoint
```

**vault.yml**

Variable | Value / Comment
--- | ---
`db.password` | The database-password you want to use for the Matomo DB

Example:

```yaml
[...]
vault_wordpress_sites:
  website.com:
    env:
      [...]
    matomo:
      db:
        password: XXXXXX
```

### Deployments

In order to make sure that every new release has a corresponding symlink set that points towards the Matomo installation, add the following [deploy hook](https://roots.io/trellis/docs/deployments/#hooks):

`deploy-hooks/share-after.yml`

```yaml
- name: Create symlink to Matomo
  file:
    path: "{{ deploy_helper.new_release_path }}/{{ item.value.public_path | default('web') }}/{{ item.value.matomo.paths.matomo }}"
    src: "{{ project_root }}/{{ item.value.matomo.paths.addons }}/matomo"
    state: link
  loop: "{{ wordpress_sites | dict2items }}"
  loop_control:
    label: "{{ item.key }}"
```

## Setup & Connection

Follow the installation instructions that are being output via **trellis-matomo: Explain next steps**.

In order to hook up the newly created Matomo-instance to your WordPress (multi-)site, make sure to install the [Connect Matomo](https://wordpress.org/plugins/wp-piwik/)-plugin:

`composer require wpackagist-plugin/wp-piwik`

### Known limitations

- This role currently does not cater for subdomain installations (matomo.example.com)
