README
======

Get the code via git::

    git clone git://github.com/ecometrica/django-multisite.git django-multisite

Run::

    python setup.py install

Or add the django-multisite/multisite folder to your PYTHONPATH.

If you wish to contribute, instead run::

    python setup.py develop


Quickstart
----------

Replace your SITE_ID in settings.py to::

    from multisite import SiteID
    SITE_ID = SiteID(default=1)

Add these to your INSTALLED_APPS::

    INSTALLED_APPS = [
        ...
        'django.contrib.sites',
        'multisite',
        ...
    ]

Add to your settings.py TEMPLATES loaders in the OPTIONS section::

    TEMPLATES = [
        ...
        {
            ...
            'OPTIONS': {
                'loaders': (
                    'multisite.template_loader.Loader',
                    'django.template.loaders.app_directories.Loader',
                )
            }
            ...
        }
        ...
    ]

Or for Django 1.7 and earlier, add to settings.py TEMPLATES_LOADERS::

    TEMPLATE_LOADERS = ( 
        'multisite.template_loader.Loader',
        'django.template.loaders.app_directories.Loader',
    ) 

Edit to settings.py MIDDLEWARE_CLASSES::

    MIDDLEWARE_CLASSES = (
        ...
        'multisite.middleware.DynamicSiteMiddleware',
        ...
    )

Append to settings.py, in order to use a custom cache that can be
safely cleared::

    # The cache connection to use for django-multisite.
    # Default: 'default'
    CACHE_MULTISITE_ALIAS = 'multisite'
    
    # The cache key prefix that django-multisite should use.
    # Default: '' (Empty string)
    CACHE_MULTISITE_KEY_PREFIX = ''

If you have set CACHE\_MULTISITE\_ALIAS to a custom value, *e.g.*
``'multisite'``, add a separate backend to settings.py CACHES::

    CACHES = {
        'default': {
            ...
        },
        'multisite': {
            'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
            'TIMEOUT': 60 * 60 * 24,  # 24 hours
            ...
        },
    }


Domain fallbacks
----------------

By default, if the domain name is unknown, multisite will respond with
an HTTP 404 Not Found error. To change this behaviour, add to
settings.py::

    # The view function or class-based view that django-multisite will
    # use when it cannot match the hostname with a Site. This can be
    # the name of the function or the function itself.
    # Default: None
    MULTISITE_FALLBACK = 'django.views.generic.base.RedirectView

    # Keyword arguments for the MULTISITE_FALLBACK view.
    # Default: {}
    MULTISITE_FALLBACK_KWARGS = {'url': 'http://example.com/',
                                 'permanent': False}

Create a directory settings.TEMPLATE_DIRS directory with the names of
domains, such as::

    mkdir templates/example.com


Cross-domain cookies
--------------------

In order to support `cross-domain cookies`_,
for purposes like single-sign-on,
prepend the following to the top of
settings.py MIDDLEWARE_CLASSES::

    MIDDLEWARE_CLASSES = (
        'multisite.middleware.CookieDomainMiddleware',
        ...
    )

CookieDomainMiddleware will consult the `Public Suffix List`_
for effective top-level domains.
It caches this file
in the system's default temporary directory
as ``effective_tld_names.dat``.
To change this in settings.py::

    MULTISITE_PUBLIC_SUFFIX_LIST_CACHE = '/path/to/multisite_tld.dat'

By default,
any cookies without a domain set
will be reset to allow \*.domain.tld.
To change this in settings.py::

    MULTISITE_COOKIE_DOMAIN_DEPTH = 1  # Allow only *.subdomain.domain.tld

In order to fetch a new version of the list,
run::

    manage.py update_public_suffix_list

.. _cross-domain cookies: http://en.wikipedia.org/wiki/HTTP_cookie#Domain_and_Path
.. _Public Suffix List: http://publicsuffix.org/


Tests
-----

To run the tests::

    python setup.py test

Before deploying a change, to verify it has not broken anything you should run::

    test_versions

This runs the tests under every supported combination of Django and Python,
isolated by creating virtualenvs. If a test breaks, it will quit, with the
virtualenv intact in .venv-python2, or .venv-python3, depending on what broke. 
You can investigate the broken version manually with::

    . .venv-python2/bin/activate  # or .venv-python3
    python setup.py test

(of course, as new versions are supported and old are retired,
please keep test_versions up to date)
