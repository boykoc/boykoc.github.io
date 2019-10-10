---
layout: post
title:  "Popular Tags and Search Facets"
date:   2019-10-10 15:00:00 -0400
categories: CKAN
---

**Warning, this is just a brain dump.** 

## Overview

I wanted to add a custom facet to replace the Tags facet in the popular tags on the home page search box.  I ended up coming up with 3 options:

* update CKAN core home.py view to include getting package facets[code](https://github.com/ckan/ckan/blob/02915e0aea933d12d974cc1aa14a4e89640e4b3c/ckan/views/home.py#L29)
* override home.py index() in my own extension [code](https://github.com/ckan/ckan/blob/c30dc8286e45ea9dab93fb7d7d452acf2ed4edac/ckanext/example_flask_iblueprint/plugin.py#L14)
* create custom template helper [code](https://docs.ckan.org/en/2.8/theming/templates.html#custom-template-helper-functions)

But then realized later I could have tried adding some default facets in my [configuration file](https://github.com/ckan/ckan/blob/99bd644eca3d0700c6be2b587917f11526aade2f/ckan/lib/helpers.py#L2597) e.g. `search.facets = u'organization groups tags res_format license_id keywords_en'`.

## Here's my long-winded, hopefully helpful thoughts on what's going on when I started digging in...

The popular tags on the home page are called from the `get_facet_items_dict` helper. This uses the `search_facets` to figure out what facets to display. `search_facets` are set in the views/home.py from the query it calls (through `package_search()`). The queries facets are set from `h.facets()` which will set the default facets from the config or with a constant in the helper file.

The query uses `def package_search` from the action/get.py. 

So, if you're using a `before_search()` to modify the search params you should be able to alter the `fq` or as @wardi said, maybe the `fq_list` but I haven't played with that one. Because, in a round about way this the action that returns the popular tags.

If using `fq` be careful. It's implemented differently depending on object you're modifying (in this case package). **In my experience with package_search it [escapes out some of the `fq` params](https://github.com/ckan/ckan/blob/f717999a1a4e63ecc192cb4dfe5a7506b7c8cff0/ckan/views/dataset.py#L220) which is good but annoying at times.**

In my case for something similar I had to write a function that "customized" the `fq` params how I wanted to keep the proper values to get them to search the way I wanted.  

E.g.

{% highlight python %}
...
v = value.replace('num_resources:"[1 TO *]"', 'num_resources:[1 TO *]') # remove the quotes to get solr to work the way I wanted.
search_params[param] = v
...
{% endhighlight %}

I did the above for a custom facet on the package search page.

However, I also wanted to customize the popular tags for a separate request I had.  In this case I just wrote a custom helper that queried my own "popular tags" and structured them the same way as the core helper.  This let me get all the facets I wanted without having to modify the existing default facets (although thinking about it now, I could have just updated the config file but I wouldn't have went down this rabbit hole).

E.g. of custom helper that I call in my homepage template:

{% highlight python %}
def get_package_keywords(language='en'):
    '''Helper to return a list of the top 3 keywords based on specified
    language.

    List structure matches the get_facet_items_dict() helper which doesn't
    load custom facets on the home page.
    [{
        'count': 1,
        'display_name': u'English Tag',
        'name': u'English Tag'
    },
    {
        'count': 1,
        'display_name': u'Second Tag',
        'name': u'Second Tag'
    }]
    '''
    facet = "keywords_{}".format(language)
    package_top_keywords = toolkit.get_action('package_search')(
        data_dict={'facet.field': [facet],
                   'facet.limit': 3,
                   'rows': 0})
    package_top_keywords = package_top_keywords['search_facets'][facet]['items']
    return package_top_keywords
{% endhighlight %}
