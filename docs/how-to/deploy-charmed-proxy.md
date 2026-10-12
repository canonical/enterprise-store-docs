---
title: Deploy a charmed store in proxy mode
description: Deploy the Enterprise Store charm in its default online proxy mode with PostgreSQL.
---

# Deploy a charmed store in proxy mode

Proxy mode is the charm's default mode. The unit must be able to reach the Snap
Store to install the Enterprise Store snap and proxy device requests. For a
deployment without internet access, see {doc}`Deploy a charmed store in offline
mode <deploy-charmed-offline>`.

## Prerequisites

You need Juju 3.x and a controller on a machine cloud, such as LXD. See the
[Juju documentation](https://documentation.ubuntu.com/juju/3.6/howto/manage-controllers/)
for controller setup. For a guided local deployment, follow
{doc}`Get started with a charmed Enterprise Store
</tutorial/charmed-deployment>`.

Register the store with `store-admin` against the domain that devices will use:

```bash
store-admin register https://store.example.com --output-file registration_bundle.b64
```

Supply the generated `registration_bundle.b64` to the charm. See
{doc}`Registration <register>` for registration concepts.

## Deploy with the Juju CLI

Charmhub currently publishes the Enterprise Store charm in the `latest/edge`
channel. Deploy and configure it as follows:

```bash
juju deploy postgresql --channel 14/stable
juju config postgresql plugin_btree_gist_enable=true
juju deploy enterprise-store --channel edge
juju integrate postgresql:database enterprise-store:database
juju config enterprise-store \
    registration_bundle="$(cat registration_bundle.b64)"
```

.. note::

  The charm requires PostgreSQL through the `database` integration. This is why
  `btree_gist` is enabled on the PostgreSQL charm so.

Run `juju status --watch 5s` until the Enterprise Store unit is active, then use
{doc}`Configure a device to use the Enterprise Store <devices>`.

## Deploy with a Juju bundle

Save the following as `bundle.yaml`, inserting the registration bundle value, and
deploy it with `juju deploy ./bundle.yaml`:

```yaml
applications:
  postgresql:
    charm: postgresql
    channel: 14/stable
    num_units: 1
    options:
      plugin_btree_gist_enable: true
  enterprise-store:
    charm: enterprise-store
    channel: edge
    num_units: 1
    options:
      registration_bundle: <registration_bundle.b64>

relations:
- - postgresql:database
  - enterprise-store:database
```

See the {doc}`charm reference </reference/charm>` for configuration and
integration details.
