.. slideconf::
   :theme: single-level

=======================
Scrapy: it GETs the Web
=======================

Asheesh Laroia

asheesh@asheesh.org

Part 0. My history with scraping
======================

* 2001: Discovered Python

* 2004: Taught 3-week mini-class on mechanize + BeautifulSoup

* 2005: ~/bin/en2fr

======================
Part 1. Scraping nowadays, without Scrapy
======================



Scraping is easy: Downloading
=============================

* **Download** with urllib2, or requests, or mechanize, or ...

* **Parse** pages with lxml/BeautifulSoup

* **Examine** with browser inspectors

* **Select** with XPath or CSS selectors

Part I
================

Scraping nowadays, without Scrapy
Typical Uses
============

* Sessions
* Authentication
* CSRF Protection
* GZipping Content

Middleware Example
==================

.. testcode::

   class LocaleMiddleware(object):

       def process_request(self, request):

           if 'locale' in request.cookies:
               request.locale = request.cookies.locale
           else:
               request.locale = None

       def process_response(self, request, response):

           if getattr(request, 'locale', False):
               response.cookies['locale'] = request.locale


Request Middleware
==================

* On ingress, middleware is executed in order
* Request middleware returns ``None`` to continue processing
* Returning an ``HttpResponse`` short circuits additional middleware

Response Middleware
===================

* On egress, middleware is executed in reverse order
* Response middleware is executed *even if corresponding request
  middleware not executed*

Writing Your Own
================

* Simple Python Classes
* Can implement all or part of the interface
* Middleware is long-lived
* The place for storing request-specific information is cunningly
  named ``request``

WSGI Middleware
===============

* WSGI_ also defines a middleware interface
* The two have similar functions, but are **not** the same

.. _WSGI: http://wsgi.org
