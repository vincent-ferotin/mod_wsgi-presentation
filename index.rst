A (very) short intro. to `mod_wsgi`
===================================

| 

| Vincent Férotin, 2013-03-01
| UPR76, CNRS
| for the `ARTFL` project & `PhiloLogic`


Preliminaries
-------------

**Outline**:
    1.  a bit of context on serving `Python` web applications
    2.  some `mod_wsgi` configuration options
    3.  current state with `PhiloLogic4`

**Disclaimer**:
    .. note::

        I only use `gunicorn` for development,
        and a monolithic `mod_wsgi` configuration for prod. that "works for me"!


Serving a `Python` web app.
---------------------------

There are several ways to serve a `Python` web application:

*   standard `Python` library `wsgiref.simple_server`_
*   a dedicated `WSGI` server (`gunicorn`_, `waitress`_, `meinheld`_, etc.)
*   a full web server (`Apache`, `nginx`, etc.)

Most tools require that you application conforms to the `WSGI`_ interface.


.. _wsgiref.simple_server:
        http://docs.python.org/2/library/wsgiref.html#module-wsgiref.simple_server
.. _gunicorn: http://gunicorn.org/
.. _waitress: http://docs.pylonsproject.org/projects/waitress/
.. _meinheld: http://meinheld.org/
.. _WSGI: http://www.wsgi.org/


`WSGI`
------

*   `WSGI`: "`Python Web Server Gateway Interface`", described in `PEP 3333`_
    This is the current standard in `Python`'s world:
    conform to it, and let use your preferred tools!
*   Running `PhiloLogic4` under `Apache httpd mod_wsgi` is a pragmatic goal.
    Probably, there is a bigger one which afford us this previous goal
    "for free": **allowing its full installation in a virtualenv**:

    *   allowing multiple installation of `PhiloLogic4` in same O.S.,
        e.g. at different versions
    *   decoupling web templates installation from databases,
        e.g. for engine upgrade without touching them


.. _PEP 3333: http://www.python.org/dev/peps/pep-3333/


System-wide VS `virtualenv`
---------------------------

| 

.. image:: sys-venv.png

Default relations numbers are ``1``.


Serving a `WSGI` app.
---------------------

Given a `WSGI`_ application, it is possible to serve it in multiple ways:

*   *development* and tests:
    a standalone `WSGI` server should suffice (e.g. `gunicorn`_, `waitress`_)

    .. code-block:: sh

        $ cd /var/www/philologic/mydb
        $ gunicorn dispatcher:philo_dispatcher

*   *production*: choose your stack, e.g.

    *   nginx + (gunicorn | uwsgi)
    *   [nginx +] apache + mod_wsgi


`Apache httpd` and `Python`
---------------------------

There are several ways to run some `Python` web application
on a `Apache` web server:

*   `CGI`, already used by `PhiloLogic4`
*   `mod_python`_
*   `mod_wsgi`_
*   `uwsgi`_
*   others?


.. _mod_python: http://www.modpython.org/
.. _mod_wsgi: http://code.google.com/p/modwsgi/
.. _uwsgi: http://projects.unbit.it/uwsgi/


`mod_wsgi`
----------

*   `Apache httpd` module
*   created and maintained by `Graham Dumpleton`_
*   feature full:

    *   mature + decent performance
    *   refresh app. without restart `Apache`, by touching `WSGI` file
    *   daemon mode, which could work with `virtualenv`_
    *   numerous configuration options

**=>** http://code.google.com/p/modwsgi/


.. _Graham Dumpleton: http://blog.dscpl.com.au/
.. _virtualenv: http://www.virtualenv.org/


Configuration (1)
-----------------

*   `WSGIScriptAlias`_, to link an URL to a `WSGI` app.

    .. code-block:: apache

        WSGIScriptAlias /myapp /path/to/my/app/application.wsgi

    should offers access to app. from ``http://my.domain.tld/myapp``

*   (probably needs) access to `WSGI` file:

    .. code-block:: apache

        <Directory /path/to/my/app>
            Order deny,allow
            Allow from all
        </Directory>


.. _WSGIScriptAlias:
        http://code.google.com/p/modwsgi/wiki/ConfigurationDirectives#WSGIScriptAlias


Configuration (2)
-----------------

*   `WSGIDaemonProcess`_, to run app. in its own process:

    .. code-block:: apache

        WSGIDaemonProcess mydomain-tld-myapp (...)

    .. note:: recommended by G.D. instead of default `embedded` mode

    Some of its options:

    *   ``processes`` & ``threads``:

        .. code-block:: apache

            WSGIDaemonProcess (...) processes=2 threads=5 (...)


.. _WSGIDaemonProcess:
    http://code.google.com/p/modwsgi/wiki/ConfigurationDirectives#WSGIDaemonProcess


Configuration (3)
-----------------

*   (`WSGIDaemonProcess` continued)

    *   ``user`` & ``group``:

        .. code-block:: apache

            WSGIDaemonProcess (...) user=work group=www-data (...)

    *   ``maximum-requests``:

        .. code-block:: apache

            WSGIDaemonProcess (...) maximum-requests=1000 (...)


Configuration (4)
-----------------

*   (`WSGIDaemonProcess` continued)

    *   ``python-path``:

        .. code-block:: apache

            WSGIDaemonProcess (...) \
                python-path=/usr/lib/python2.7,/usr/local/lib/python2.7/dist-packages,(...)

        which allows using a `virtualenv`!

        Given a fresh `virtualenv` (e.g. called ``myappvenv``),
        it is possible to set ``python-path`` to its path value:

        .. code-block:: apache

            WSGIDaemonProcess (...) \
                python-path=/path/to/myappvenv/lib/python2.7/site-packages


Configuration (5)
-----------------

*   link process group to  `WSGI` parent dir. by its *name*:

    .. code-block:: apache

        WSGIDaemonProcess mydomain-tld-myapp (...)

        <Directory /path/to/my/app>
            WSGIProcessGroup mydomain-tld-myapp
            WSGIApplicationGroup %{GLOBAL}
            (...)
        </Directory>


Full example
------------

.. code-block:: apache

    WSGIDaemonProcess mydomain-tld-myapp \
        processes=2 threads=5 \
        python-path=/path/to/myappvenv/lib/python2.7/site-packages \
        user=work group=www-data \
        maximum-requests=1000 \
        display-name=%{GROUP}

    <Directory /path/to/my/app>
        WSGIProcessGroup mydomain-tld-myapp
        WSGIApplicationGroup %{GLOBAL}
        Order deny,allow
        Allow from all
    </Directory>

    WSGIScriptAlias /myapp /path/to/my/app/application.wsgi


Addendum
--------

There should be some tricky additional steps, such as:

*   setting good rights to paths;
*   setting path for default daemon process -- but could not remember :-(
*   others? be careful...


RTFantasticMaintainer's
-----------------------

*   `configuration guidelines
    <http://code.google.com/p/modwsgi/wiki/ConfigurationGuidelines>`_
*   `configuration directives
    <http://code.google.com/p/modwsgi/wiki/ConfigurationDirectives>`_
*   `on virtualenv
    <http://code.google.com/p/modwsgi/wiki/VirtualEnvironments>`_
*   `FAQ
    <http://code.google.com/p/modwsgi/wiki/FrequentlyAskedQuestions>`_
*   `config issues
    <http://code.google.com/p/modwsgi/wiki/ConfigurationIssues>`_
    and `application issues
    <http://code.google.com/p/modwsgi/wiki/ApplicationIssues>`_
*   Graham Dumpleton:
    `its blog
    <http://blog.dscpl.com.au/search/label/mod_wsgi>`_,
    `some of its conferences
    <http://pyvideo.org/search?models=videos.video&q=graham+dumpleton>`_


`PhiloLogic4` and `mod_wsgi`
----------------------------

*   It currently does not work (out of the box)!
*   It should, easily (already `WSGI` aware :-):
    it's probably almost a application configuration problem (?).
    Pb closely related to succeeding in installing app. into a `virtualenv`?
*   Quick tests:
    putting a `WSGI` module into ``/var/www/philologic/mydb/``,
    and trying to serve it either by `gunicorn` or `mod_wsgi`...


Quick test (0) `WSGI` file
--------------------------

Given the following `WSGI` module, put into ``/var/www/philologic/mydb/app.py``,
next ``dispatcher.py`` and its friends (``data/``, ``templates/``, etc.):

.. code-block:: python

    import sys

    sys.path.append('/var/www/philologic/mydb')
    from dispatcher import philo_dispatcher as application

and its following link ``app.wsgi``:

.. code-block:: sh

    /var/www/philologic/mydb $ ln -s app.py app.wsgi


Quick test (1) `gunicorn` (``app.py``)
--------------------------------------

.. code-block:: sh

    /var/www/philologic/mydb $ gunicorn app
    (...)
    [ERROR] Error handling request
    Traceback (most recent call last):
    File "/var/www/philologic/mydb/dispatcher.py", line 20, in philo_dispatcher
        yield getattr(reports, report or "navigation")(environ,start_response)
    File "/var/www/philologic/mydb/reports/navigation.py", line 17, in navigation
        db, dbname, path_components, q = wsgi_response(environ,start_response)
    File "/var/www/philologic/mydb/functions/wsgi_handler.py", line 18, in wsgi_response
        myname = environ["SCRIPT_FILENAME"]
    KeyError: 'SCRIPT_FILENAME'


Quick test (2) `mod_wsgi` (``app.wsgi``)
----------------------------------------

``Internal Server Error``

.. code-block:: sh

    /var/log/apache2 $ tail error.log
    (...)
    mod_wsgi: Exception occurred processing WSGI script '/var/www/philologic/mydb/app.wsgi'.
    Traceback (most recent call last):
      File "/var/www/philologic/mydb/dispatcher.py", line 24, in philo_dispatcher
        yield reports.form(environ,start_response)
      File "/var/www/philologic/mydb/reports/form.py", line 11, in form
        return render_template(db=db,dbname=dbname,form=True, template_name='form.mako')
      File "/var/www/philologic/mydb/reports/render_template.py", line 12, in render_template
        template = Template(filename="templates/%s" % data['template_name'], lookup=templates)
      (...)
    IOError: [Errno 2] No such file or directory: 'templates/form.mako'


`virtualenv` installation test
------------------------------

Given `virtualenvwrapper`_ installed:

.. code-block:: sh

    $ mkvirtualenv philologic
    $ # virtualenv 'philologic' activated
    $ # install libphilo
    $ cd libphilo
    $ make install exec_prefix=/path/to/virtualenvs/philologic
    $ # install python bindings
    $ cd ../python
    $ python setup.py install
    $ # install web application
    $ cd ../www
    $ pip install Mako BeautifulSoup

But... how ``pip install philologic-webapp``?


ToDo?
-----

1.  make `PhiloLogic4` runnable under `mod_wsi`, and let web app.
    closed to a specific database, which probably only needs:

    *   fix environment variables and paths

2.  and/or create an installable package for web app.

    *   create a true `Python` package namespace (e.g. :mod:`philologic.web`),
        and use this namespace anywhere, instead of tweaking ``sys.path``
    *   write a dedicated ``setup.py``, or merge into already existing
        :mod:`philologic`


.. _virtualenvwrapper: http://virtualenvwrapper.readthedocs.org/en/latest/

