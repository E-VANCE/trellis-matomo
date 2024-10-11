# Trellis Matomo

> Installing Matomo for Trellis (via Ansible)

## Installation

This Ansible-role for [Trellis](https://roots.io/trellis) installs the latest on-premise version of [Matomo](https://matomo.org/), the GDPR- and CCPA-compliant web analytics tool.

Add the role to the `galaxy.yml` file of Trellis:

```yaml
- name: trellis-matomo
  src: https://github.com/E-VANCE/trellis-matomo
  type: git
  version: '0.2'
```

Run `ansible-galaxy install -r galaxy.yml` (or `trellis install galaxy` is you have [trellis-cli](https://github.com/roots/trellis-cli)) to install the new role.

Then, add the role into both `server.yml` **and** `dev.yml`:

```yaml
roles:
    ... other Trellis roles ...
    - { role: trellis-matomo, tags: [matomo]}
```

After adding the role to the above files and running the install, provision your Vagrant box via `vagrant reload --provision` (if it's running) or `vagrant provision` (if it's not). If you haven't provisioned the box yet simply run `vagrant up`.

If you have [trellis-cli](https://github.com/roots/trellis-cli) installed – which is highly recommended – then use `trellis up` / `trellis vm start` or  `trellis provision {ENV}`.

## Configuration

### Group vars

You define the following credentials for Matomo in your `group_vars/*/wordpress_sites.yml`:

Variable | Value / Comment
--- | ---
`db.user` | The database-user you want to create for the Matomo DB
`db.host` | Should be `localhost` if you're running with the Trellis default
`path` | *optional* – The path where Matomo should be reachable within your website, f.ex. **example.com/analytics**

Example:

```yaml
    matomo:
      db:
        user: matomo
        host: localhost
      path: analytics
```

### Deployments

In order to make sure that every new release has a corresponding symlink set that points towards the Matomo installation, add the following [deploy hook](https://roots.io/trellis/docs/deployments/#hooks):

`deploy-hooks/share-after.yml`

```yaml
- name: Create symlink to Matomo
  file:
    path: "{{ deploy_helper.new_release_path }}/{{ item.value.public_path | default('web') }}/{{ item.value.matomo.path | default('matomo') }}"
    src: "{{ addons_dir }}/{{ matomo_dir }}"
    state: link
  with_items: "{{ wordpress_sites }}"
```

## Setup & Connection

Follow the installation instructions that are being output via **trellis-matomo: Explain next steps**.

In order to hook up the newly created Matomo-instance to your WordPress (multi-)site, make sure to install the [Connect Matomo](https://wordpress.org/plugins/wp-piwik/)-plugin:

`composer require wpackagist-plugin/wp-piwik`

### Known limitations

- This role currently does not cater for subdomain installations (matomo.example.com)