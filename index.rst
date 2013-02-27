A (very) short intro. to `mod_wsgi`
===================================

Serve a `Python` web app.
-------------------------

There are several ways to serve a `Python` web application:

*   standard `Python` library `wsgiref.simple_server
    <http://docs.python.org/2/library/wsgiref.html#module-wsgiref.simple_server>`_
*   a dedicated `WSGI` server (`gunicorn <http://gunicorn.org/>`_,
    `waitress <http://docs.pylonsproject.org/projects/waitress/>`_,
    `meinheld <http://meinheld.org/>`_, etc.)
*   a full web server (`Apache`, `nginx`, etc.)


`Apache httpd` and `Python`
---------------------------

There are several ways to run some `Python` web application
on a `Apache` web server:

*   `CGI`, already used by `PhiloLogic4`
*   `mod_python <http://www.modpython.org/>`_
*   `mod_wsgi <http://code.google.com/p/modwsgi/>`_
*   `uwsgi <http://projects.unbit.it/uwsgi/>`_
*   others?

