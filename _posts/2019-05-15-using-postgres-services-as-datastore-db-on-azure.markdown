---
layout: post
title:  "Using postgres service as datastore DB on azure"
date:   2019-05-15 15:00:00 -0400
categories: CKAN datastore azure postgres
---

On a current project I'm running CKAN on Azure. Overall it's been a good
experience.

In a recent update we decided to switch from a VM running postgres as our
DB server and use an Azure DB for PostgreSQL instance instead. It had a
few benefits such as easier maintence, automatic backups and security out
of the box.

There are a few changes with this type of setup compared to using postgres
locally on the same application VM or as a seperate VM that just does your
DB. The main changes are how you interact with the DB, which boils down to
strictly using `psql`, and the connection strings.

The interaction part is relatively straightforward, you just need to readup
on the docs and adapt the CKAN commands accordinly when setting things up
or running some updates.

The connection string is also pretty straightforward for the most part.
Azure has some decent [docs on how to get the inforamation and the format](https://docs.microsoft.com/en-ca/azure/postgresql/quickstart-create-server-database-portal#get-the-connection-information).

However, this did cause an issue when setting up the Datastore.

CKAN will run some checks on the datastore connections and permissions
which is kind of re-assuring to know there is some level of checks in
place. But, to check the read permissions it tries to get the datastore
read connection string username.

https://github.com/ckan/ckan/blob/b5aeb795e0b1624238df1190e24ec6b1228b67d6/ckanext/datastore/backend/postgres.py#L1594
{% highlight python %}
        read_connection_user = sa_url.make_url(self.read_url).username
{% endhighlight %}

To do this it's using `import sqlalchemy.engine.url as sa_url` which is
sqlalchemy's good attempt at extract the database connection information.

Sqlalchemy extracts this information like so:

{% highlight python %}
def _parse_rfc1738_args(name):
    pattern = re.compile(
        r"""
            (?P<name>[\w\+]+)://
            (?:
                (?P<username>[^:/]*)
                (?::(?P<password>.*))?
            @)?
            (?:
                (?:
                    \[(?P<ipv6host>[^/]+)\] |
                    (?P<ipv4host>[^/:]+)
                )?
                (?::(?P<port>[^/]*))?
            )?
            (?:/(?P<database>.*))?
            """,
        re.X,
    )

    m = pattern.match(name)
    
    ...
{% endhighlight %}

But this assumes the username is an ordinary `joe_smith` type username. 
It's not expecting something like the username format that Azure's 
postgres offering requires.

So, this is a CKAN issue because it's limiting users potential setups.
But it's an sqlalchemy problem because it's code assumes simple username
formats. *But, it's a my problem because I'm using Azure DB for postgreSQL
service*.

## Solutions

### Flag for CKAN and hope it gets addressed in source code there

Possibly don't use sqlalchemy's functions to get the db username or implement
something similar to below?

### Flag for sqlalchemy

{% highlight python %}
import re

pattern = re.compile(
    r"""
        (?P<name>[\w\+]+)://
        (?:
            (?P<username>[^:/]*)
            (?::(?P<password>.*))?
        @)?
        (?:
            (?:
                \[(?P<ipv6host>[^/]+)\] |
                (?P<ipv4host>[^/:]+)
            )?
            (?::(?P<port>[^/]*))?
        )?
        (?:/(?P<database>.*))?
        """,
    re.X,
)
{% endhighlight %}

Could change username extract to something like below BUT this is
only accounting for my specific setup.

{% highlight python %}

            (?P<username>[^@:/]*)

{% endhighlight %}

### Modify CKAN core on my instance

This is what I'm currently doing with the following:

{% highlight python %}
        # ckan/ckanext/datastore/backend/postgres.py
        # Line 1594.
        read_connection_user = sa_url.make_url(self.read_url).username
        # If connection user comes back with a "@host" split and get just username.
        read_connection_user = read_connection_user.split('@')[0]
{% endhighlight %}        
