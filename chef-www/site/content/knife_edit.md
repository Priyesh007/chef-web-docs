+++
title = "knife edit"
description = "DESCRIPTION"
draft = false

aliases = "/knife_edit.html"

[menu]
  [menu.docs]
    title = "knife edit"
    identifier = "knife edit/knife_edit.html"
    parent = "chefdk/knife"
    weight = 150
+++    

[\[edit on
GitHub\]](https://github.com/chef/chef-web-docs/blob/master/chef_master/source/knife_edit.rst)

{{% knife_edit_summary %}}

Syntax
======

This subcommand has the following syntax:

``` bash
$ knife edit (options)
```

Options
=======

<div class="note" markdown="1">

<div class="admonition-title" markdown="1">

Note

</div>

{{% knife_common_see_common_options_link %}}

</div>

This subcommand has the following options:

`--chef-repo-path PATH`

:   The path to the chef-repo. This setting will override the default
    path to the chef-repo. Default: same value as specified by
    `chef_repo_path` in client.rb.

`--concurrency`

:   The number of allowed concurrent connections. Default: `10`.

`--local`

:   Show files in the local chef-repo instead of a remote location.
    Default: `false`.

`--repo-mode MODE`

:   The layout of the local chef-repo. Possible values: `static`,
    `everything`, or `hosted_everything`. Use `static` for just roles,
    environments, cookbooks, and data bags. By default, `everything` and
    `hosted_everything` are dynamically selected depending on the server
    type. Default: `everything` / `hosted_everything`.

<div class="note" markdown="1">

<div class="admonition-title" markdown="1">

Note

</div>

{{% knife_common_see_all_config_options %}}

</div>

Examples
========

The following examples show how to use this knife subcommand:

**Remove a user from /groups/admins.json**

{{% knife_edit_admin_users %}}