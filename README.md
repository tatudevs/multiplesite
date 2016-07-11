Overview
============
An experiment in how to to handle the "multiplesite" pattern in Drupal 8. This is when you run dozens or hundreds of very similar sites. You want these sites to vary configuration in slight ways, but still easily push out config changes to all. Examples include [one site for each member of Congress](http://buytaert.net/us-house-of-representatives-using-drupal) or [one site for each musician in our portfolio](http://www.warnerbrosrecords.com/artists).

Getting Started
==============
1. Clone this repo
1. Run `composer install`
1. Run `cd web`
1. Run `drush msi -vy`. This creates a top level 'config' directory and one subdirectory for each site in the Drupal multisite (currently 3).
1. view web/sites/settings.allsites.php. The DB is setup for Acquia Dev Desktop. If needed, override that by creating a settings.local.php in each settings subdir.
1. Run `drush @master site-install -vy --config-dir=../config/master/sync`. Do same for alpha and bravo sites, replacing alias name and dir name.
1. Verify that sites are working: `drush @master status`, `drush @alpha status`
1. In 3 new terminal windows, run `drush @master runserver`, `drush @alpha runserver`, `drush @bravo runserver`. This will give you a web site to play with. Drush reports back the URL of the site.
1. Edit your /etc/hosts so that the 'master', 'alpha', 'bravo' domains all point to 127.0.0.1


You now have 3 working Drupal sites, mapped to the right databases and config dirs (see web/sites/settings.allsites.php).

Implementation Discussion
=============
There are two git repos:

1. [multiplesite](https://github.com/weitzman/multiplesite). This repo carries the shared for code for all the sites.
1. [multiplesite-config](https://github.com/weitzman/multiplesite-config). This repo has a master branch, where the "golden" Drupal config is stored. Then we create branches off of master - one for each client site. These branches are cloned into place under a /config directory by the `drush msi` command. This build step seems cleaner than a submodule approach which would embed config repos into the code repo.

We setup a Drupal multisite where the 'master' site carries the 'golden' config, and the client sites merge in golden config periodically. So the workflow is that client sites occasionally change config on their own sites and that config gets exported and committed to their own branch frequently. When the master wants to push out new config, we merge from multisite-config/master (or a tag there) into each client branch.

Notes and Limitations
=============
1. A client could create a config entity whose machine name later conflicts with a name from master. That sort of conflict can happen with content as well.
1. If a client deletes a config entity, it would come back with next import. We would need a "disable" instead. Not sure if this is supported by all config entity types.
1. The 'minimal' profile must be used here as its a requirement of drush site-install --config-dir. One could likely do this with the config_installer profile but a custom profile is not likely to work. A custom distro should not be needed since client can just build upon minimal.
1. The 'default' site is deliberately unused. One must specify a site alias for all commands. See /drush/aliases.drushrc.php.
1. The `drush site-set` command is convenient when sending multiple Drush requests. `drush init` will customize your shell prompt so the current Drupal Site is shown.
1. This experiment uses Drupal multisite is used for convenience only. This technique works with separate docroots as well.


Credits
================
- This repo is based on https://github.com/drupal-composer/drupal-project.