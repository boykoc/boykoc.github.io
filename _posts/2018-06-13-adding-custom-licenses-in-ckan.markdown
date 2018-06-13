---
layout: post
title:  "Adding Custom Licenses in CKAN"
date:   2018-06-13 9:34:00 -0400
categories: ckan configuration
---
## Overview

Quick notes on adding a custom list of license to a CKAN build. Really I just wanted to add 1 license to the default CKAN list.

## Notes

First I had to figure out where this was stored and how it was handled. In CKAN 2.7, which is what I'm currently using, the relavent information I found was:

* A bullet in the [User Guide](http://docs.ckan.org/en/latest/user-guide.html#adding-a-new-dataset) telling people to contact the system admin if a new license is needed. This confirmed to me that it was possible.
* Eventually led me to the [Licenses Group URL Config](http://docs.ckan.org/en/ckan-2.7.3/maintaining/configuration.html?highlight=license#licenses-group-url) that hinted at being able to point to a local file if desired.
* Then wanting to know how this was implemented I searched the github repo to find the code [`ckan/ckan/model/license.py`](https://github.com/ckan/ckan/blob/555e0960c43d0ca86066b1954e5c94aad565baa7/ckan/model/license.py)

After this I decided to copy the default ckan.json from [Open Licenses Services](https://licenses.opendefinition.org/) into the main ckan src directory for now and save the response.

`curl https://licenses.opendefinition.org/licenses/groups/ckan.json > ckan.json`

Then modify the `production.ini` file to uncomment the `licenses_group_url` config option and changed the url to a path.

`licenses_group_url = file:///home/ckanopen/ckan/lib/default/src/ckan/ckan.json`

Then edit the new `ckan.json` file to add the new license. I used the UK Open Gov license object as a template to add my new license and saved the changes.

then restarted the server to see the changes.

`sudo service apache2 restart`

Then navigate to the Add Dataset or Edit Dataset pages to see the new option(s) in the license select dropdown.
