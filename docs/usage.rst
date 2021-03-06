.. _usage:

=====
Usage
=====

Django Correlation Ids have a number of options for usage covering logging,
SQL comments, and a template context processor.

All of these rely on the usage of a piece of middleware. To configure the
middleware simply add the following to your list of MIDDLEWARE_CLASSES in
your settings file:

.. code-block:: python

    MIDDLEWARE_CLASSES = (
        'cid.middleware.CidMiddleware',
        # other middleware
    )

By default the middleware will look for the correlation id in an incoming
request header called ``X_CORRELATION_ID``. An alternative header name can be
used by putting a value into settings for the ``CID_HEADER``. e.g.:

.. code-block:: python

    CID_HEADER = 'X_MY_CID_HEADER'

You can also configure Django Correlation Id to generate it's own correlation
id if one if not found in the header. For this set ``CID_GENERATE`` to true in
you settings file:

.. code-block:: python

    CID_GENERATE = True


SQL Comments
------------

To make use of the SQL comments support you will need to change your database
backend to one of the cid wrapped database backends. e.g. for sqlite3 you will
need to use the following:

.. code-block:: python

    DATABASES = {
        'default': {
            'ENGINE': 'cid.backends.sqlite3',
            'NAME': location('db.sqlite3'),
        }
    }

The are database backend wrappers for all the currently support database
backends found in Django.

mysql
    cid.backends.mysql
oracle
    cid.backends.oracle
postgresql
    cid.backends.postgresql
sqlite3
    cid.backends.sqlite3


Logging
-------

To make use of the correlation id in logs you will need to add the cid log
filter to your settings and apply it to each logger.

e.g.

.. code-block:: python

    LOGGING = {
        'version': 1,
        'disable_existing_loggers': False,
        'formatters': {
            'verbose': {
                'format': '[cid: %(cid)s] %(levelname)s %(asctime)s %(module)s %(process)d %(thread)d %(message)s'
            },
            'simple': {
                'format': '[cid: %(cid)s] %(levelname)s %(message)s'
            }
        }
        'handlers': {
            'file': {
                'level': 'DEBUG',
                'class': 'logging.FileHandler',
                'filename': '/path/to/django/debug.log',
                'formatter': 'verbose',
            },
        },
        filters: {
            'correlation': {
                (): 'cid.log.CidContextFilter'
            }
        }
        'loggers': {
            'django.request': {
                'handlers': ['file'],
                'level': 'DEBUG',
                'propagate': True,
                'filters': ['correlation']
            },
        },
    }

You can then use your loggers as normal, safe in the knowledge that you can tie
them all back to the correlation id.


Template Context Processor
--------------------------

Django Correlation Id provides a template context processor which adds the
correlation id to the template context if it is available. To enable this you
will need to add the context processor to your settings:

.. code-block:: python

    TEMPLATE_CONTEXT_PROCESSORS = (
        "django.contrib.auth.context_processors.auth",
        "django.core.context_processors.debug",
        "django.core.context_processors.i18n",
        "django.core.context_processors.media",
        "django.core.context_processors.static",
        "django.core.context_processors.tz",
        "django.contrib.messages.context_processors.messages",
        "cid.context_processos.cid_context_processor",
    )

This will place the context variable ``correlation_id`` in your template
context if a correlation id is available. For example you could add it as a
meta tag in your templates with the follwing snippet:

.. code-block:: django

    {% if correlation_id %}
        <meta name="correlation_id" content="{{ correlation_id }}" />
    {% endif %}
