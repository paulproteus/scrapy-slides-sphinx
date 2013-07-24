.. slideconf::
   :theme: single-level

=======================
Scrape the web
=======================

speedily, reliably, and simply with scrapy

Asheesh Laroia

asheesh@asheesh.org

My history with scraping
================================

* 2004: Taught 3-week "Learn Python Through Scraping"

My history with scraping
================================

* 2004: Taught 3-week "Learn Python Through Scraping"

* 2009: "Volunteer opportunity finder" within OpenHatch

My history with scraping
================================

* 2004: Taught 3-week "Learn Python Through Scraping"

* 2009: "Volunteer opportunity finder" within OpenHatch

* 2011: vidscraper. multiprocessing? gevent?

My history with scraping
================================

* 2004: Taught 3-week "Learn Python Through Scraping"

* 2009: "Volunteer opportunity finder" within OpenHatch

* 2011: vidscraper. multiprocessing? gevent?

* 2012: "Volunteer opportunity finder" (aka oh-bugimporters) rewrite w/ Scrapy


My history with scraping
================================

* 2004: Taught 3-week "Learn Python Through Scraping"

* 2009: "Volunteer opportunity finder" within OpenHatch

* 2011: vidscraper. multiprocessing? gevent?

* 2012: "Volunteer opportunity finder" (aka oh-bugimporters) rewrite w/ Scrapy

(thanks)

* Karen Rustad, for diagrams and image selection
* Pablo Hoffman, for starting Scrapy
* Image actual authors: Eric Wilde, April Killingsworth, Jim Davis, Dan Walsh, Steven Depolo
* Stephen Burrows for vidscraper
* Nathan Yergler for slides framework (hieroglyph)

======
Section: Scraping without scrapy
======

Web pages
=========

.. figure:: /_static/rendered.png
   :class: fill

HTML source
===========

.. figure:: /_static/view-source.png
   :class: fill

As diagram
==========

.. figure:: /_static/html-structure.gif
   :class: fill

DOM inspector
=============

.. figure:: /_static/inspector.png
   :class: fill

Scraping in Python (2004)
=========================

.. testcode::

   >>> # get a web page
   >>> page = urllib2.urlopen('http://oscon.com/').read()

.. figure:: /_static/view-source.png

Scraping in Python (2004)
=========================

.. testcode::

   >>> # get a web page
   >>> page = urllib2.urlopen('http://oscon.com/').read()
   >>> # parse it
   >>> soup = BeautifulSoup.BeautifulSoup(page)

.. figure:: /_static/html-structure.gif

Scraping in Python (2004)
=========================

.. testcode::

   >>> # get a web page
   >>> page = urllib2.urlopen('http://oscon.com/').read()
   >>> # parse it
   >>> soup = BeautifulSoup.BeautifulSoup(page)
   >>> # find element we want
   >>> matches = soup('div', {'id': 'location_place'})

.. figure:: /_static/inspector.png

Scraping in Python (2004)
=========================

.. testcode::

   >>> # get a web page
   >>> page = urllib2.urlopen('http://oscon.com/').read()
   >>> # parse it
   >>> soup = BeautifulSoup.BeautifulSoup(page)
   >>> # find element we want
   >>> matches = soup('div', {'id': 'location_place'})

Finish extraction and save:


.. testcode::

   >>> # pull out text
   >>> first = matches[0]
   >>> date_range = r[0].find(text=True)
   >>> print date_range
   u'July 22-26, 2013'
   >>> # store results somehow
   >>> save_results({'conference': 'oscon', 'date_range': date_range})

What could be better
====================

.. testcode::

   >>> # get a web page
   >>> page = urllib2.urlopen('http://oscon.com/').read()

This bloc

What could be better
====================

.. testcode::

   >>> # get a web page
   >>> page = urllib2.urlopen('http://oscon.com/').read()

This blocks until the remote site responds.

What could be better
====================

.. testcode::

   >>> # get a web page
   >>> page = urllib2.urlopen('http://oscon.com/').read()

This blocks until the remote site responds.

Must test online.

What could be better
====================

.. testcode::

   >>> # get a web page
   >>> page = urllib2.urlopen('http://oscon.com/').read()

This blocks until the remote site responds.

Must test online.

If this fails, the app crashes.

What could be better
====================

.. testcode::

   >>> # pull out text
   >>> first = matches[0]

If this fails, the app crashes.

What could be better
====================

.. testcode::

   >>> # find element we want
   >>> matches = soup('div', {'id': 'location_place'})

That's just a CSS selector!

What could be better
====================

.. testcode::

   >>> # store results somehow
   >>> save_results({'conference': 'oscon', 'date_range': date_range})

No clarity about data format. Code evolves!

======================================
Section: Importing Scrapy components for sanity
======================================

Rewriting some non-scrapy code
================

Task: Get a list of speakers

Rewriting some non-scrapy code
================

Task: Get a list of speakers

.. testcode::

   SCHED_PAGE='https://us.pycon.org/2013/schedule/'

A word about CSS selectors
==========================

CSS and XPath

.. testcode::

    >>> import cssselect
    >>> cssselect.HTMLTranslator().css_to_xpath('span.speaker')
    u"descendant-or-self::span[@class and contains(concat(' ', normalize-space(@class), ' '), ' speaker ')]"

https://github.com/scrapy/scrapy/pull/176


Rewriting some non-scrapy code
================

Task: Get a list of speakers

.. testcode::

   SCHED_PAGE='https://us.pycon.org/2013/schedule/'

   import requests
   import lxml.html

   def main():
       data = requests.get(SCHED_PAGE)
       parsed = lxml.html.fromstring(data.content)
       for speaker in parsed.cssselect('span.speaker'):
           print speaker.text_content()

Rewriting some non-scrapy code
================

Why: **Separate handling from retrieving**

.. testcode::

   SCHED_PAGE='https://us.pycon.org/2013/schedule/'

   import requests
   import lxml.html

   def main():
       data = requests.get(SCHED_PAGE)
       parsed = lxml.html.fromstring(data.content)
       for speaker in parsed.cssselect('span.speaker'):
           print speaker.text_content()
       #   ↑

Rewriting some non-scrapy code
================

Why: **Separate handling from retrieving**

.. testcode::

   SCHED_PAGE='https://us.pycon.org/2013/schedule/'

   import requests
   import lxml.html

   def main():
       data = requests.get(SCHED_PAGE)
       parsed = lxml.html.fromstring(data.content)
       for speaker in parsed.cssselect('span.speaker'):
           print speaker.text_content()
       #   ↑

``UnicodeEncodeError: 'ascii' codec can't encode character u'\xe9' in position 0: ordinal not in range(128)``

Rewriting some non-scrapy code
================

How: **Separate handling from retrieving**

.. testcode::

   SCHED_PAGE='https://us.pycon.org/2013/schedule/'

   import requests
   import lxml.html

   def get_data():
       data = requests.get(SCHED_PAGE)
       parsed = lxml.html.fromstring(data.content)
       data = []
       for speaker in parsed.cssselect('span.speaker'):
            data.append(speaker.text_content())
       return data


Rewriting some non-scrapy code
================

Why: **Clarify the fields you are retrieving**

.. testcode::

   SCHED_PAGE='https://us.pycon.org/2013/schedule/'

   import requests
   import lxml.html

   def get_data():
       data = requests.get(SCHED_PAGE)
       parsed = lxml.html.fromstring(data.content)
       data = []
       for speaker in parsed.cssselect('span.speaker'):
            datum = {}
            datum['speaker_name'] = speaker.text_content()
	    datum['preso_title'] = _ # FIXME
       return data

Rewriting some non-scrapy code
================

Why: **Clarify the fields you are retrieving**

.. testcode::

   SCHED_PAGE='https://us.pycon.org/2013/schedule/'

   import requests
   import lxml.html

   def get_data():
       data = requests.get(SCHED_PAGE)
       parsed = lxml.html.fromstring(data.content)
       data = []
       for speaker in parsed.cssselect('span.speaker'):
            datum = {}
            datum['speaker_name'] = speaker.text_content()
	    datum['preso_title'] = _ # FIXME
       return data # ↑

   def handle_datum(datum):
       print datum['title'], 'by', datum['speaker_name']
   #                ↑


scrapy.items.Item
=================

.. testcode::

    class PyConPreso(scrapy.item.Item):
        author = Field()
        preso = Field()

scrapy.items.Item
=================

.. testcode::

    class PyConPreso(scrapy.item.Item):
        author = Field()
        preso = Field()

.. testcode::

    # Similar to...
    {'author': _,
     'preso':  _}

scrapy.items.Item
=================

.. testcode::

    class PyConPreso(scrapy.item.Item):
        author = Field()
        preso = Field()

.. testcode::

    # Similar to...
    {'author': _,
     'preso':  _}

::

   >>> p['title'] = 'Asheesh'
   KeyError: 'PyConPreso does not support field: title'


Better
======

.. testcode::

   def get_data():
       data = requests.get(SCHED_PAGE)
       parsed = lxml.html.fromstring(data.content)
       data = []
       for speaker in parsed.cssselect('span.speaker'):
           author = _ # ...
	   preso_title = _ # ...
	   item = PyConPreso(
               author=author,
	       preso=preso_title)
           out_data.append(item)
       return out_data

scrapy.spider.BaseSpider
========================

.. testcode::

    import lxml.html
    START_URL = '...'

    class PyConSiteSpider(BaseSpider):
        start_urls = [START_URL]

        def parse(self, response):
            parsed = lxml.html.fromstring(
                              response.body_as_unicode)
            slots = parsed.cssselect('span.speaker')
	    results = []
            for speaker in speakers:
                author = _ # placeholder
                preso = _  # placeholder
                results.append(PyConPreso(
		        author=author, preso=preso))
            return results

scrapy.spider.BaseSpider
========================

.. testcode::

    import lxml.html
    START_URL = '...'

    class PyConSiteSpider(BaseSpider):
        start_urls = [START_URL]

        def parse(self, response):
            parsed = lxml.html.fromstring(
                              response.body_as_unicode)
            slots = parsed.cssselect('span.speaker')
            for speaker in speakers:
                author = _ # placeholder
                preso = _  # placeholder
                yield PyConPreso(
		        author=author, preso=preso)

How you run it
==============

::

    $ scrapy runspider your_spider.py


How you run it
==============

::

    $ scrapy runspider your_spider.py
    2013-03-12 18:04:07-0700 [Demo] DEBUG: Crawled (200) <GET ...> (referer: None)
    2013-03-12 18:04:07-0700 [Demo] DEBUG: Scraped from <200 ...>
    {}
    2013-03-12 18:04:07-0700 [Demo] INFO: Closing spider (finished)
    2013-03-12 18:04:07-0700 [Demo] INFO: Dumping spider stats:
    {'downloader/request_bytes': 513,
    'downloader/request_count': 2,
    'downloader/request_method_count/GET': 2,
    'downloader/response_bytes': 75142,
    'downloader/response_count': 2,
    'downloader/response_status_count/200': 1,
    'downloader/response_status_count/301': 1,
    'finish_reason': 'finished',
    'finish_time': datetime.datetime(2013, 3, 13, 1, 4, 7, 567078),
    'item_scraped_count': 1,
    'scheduler/memory_enqueued': 2,
    'start_time': datetime.datetime(2013, 3, 13, 1, 4, 5, 144944)}
    2013-03-12 18:04:07-0700 [Demo] INFO: Spider closed (finished)
    2013-03-12 18:04:07-0700 [scrapy] INFO: Dumping global stats:
    {'memusage/max': 95105024, 'memusage/startup': 95105024}

How you run it
==============

::

    $ scrapy runspider your_spider.py -L ERROR
    $

Customizing output
==================

::

    $ scrapy runspider your_spider.py -s FEED_URI=myfile.out
    $
...
===

.. figure:: /_static/scrapy-diagram-1.png
   :class: fill


...
===

.. figure:: /_static/scrapy-diagram-2.png
   :class: fill

===============================
Section: Pros and Cons of Scrapy
===============================

   >>> 'Pablo Hoffman' > 'Asheesh Laroia'
   True

Part III. An aside about Scrapy
===============================

* Scrapy: 9,000

Part III. An aside about Scrapy
===============================

* Scrapy: 9,000

* Mechanize: 20,000

Part III. An aside about Scrapy
===============================

* Scrapy: 9,000

* Mechanize: 20,000

* Requests: 475,000

Scrapy wants you to make a project
==================================

::

  $ scrapy startproject tutorial

Scrapy wants you to make a project
==================================

::

  $ scrapy startproject tutorial

creates

::

  tutorial/
      scrapy.cfg
      tutorial/
          __init__.py
          items.py
          pipelines.py
          settings.py
          spiders/
              __init__.py

Awesome features
================

.. figure:: /_static/cloud.png
   :class: fill

Awesome features...
===================

    $ scrapy runspider your_spider.py &

    $ telnet localhost 6023

Awesome features...
===================

    $ scrapy runspider your_spider.py &

    $ telnet localhost 6023

Gives

    >>> est()
    Execution engine status
    time()-engine.start_time              : 21.3188259602
    engine.is_idle()                      : False
    …


Awesome features...
===================

    $ scrapy runspider your_spider.py &

    $ telnet localhost 6023

Gives

    >>> est()
    Execution engine status
    time()-engine.start_time              : 21.3188259602
    engine.is_idle()                      : False
    …
    >>> import os; os.system('eject')
    0
    >>> # Hmm.

Awesome features...
===================

  $ scrapy runspider your_spider.py -s TELNETCONSOLE_ENABLED=0 -s WEBSERVICE_ENABLED=0

Awesome features...
===================

  $ scrapy runspider your_spider.py -s TELNETCONSOLE_ENABLED=0 -s WEBSERVICE_ENABLED=0

Semi-complex integration with other pieces of code.

Section: Async
==============

.. figure:: /_static/asink.jpg
   :class: fill

If you're not done, say so
==========================

.. testcode::

   def parse(self, response):
       # ...
       for speaker in speakers:
           partial_item = PyConPreso(author=author)
	   # need more data!


If you're not done, say so
==========================

.. testcode::

   def parse(self, response):
       # ...
       for speaker in speakers:
           partial_item = PyConPreso(author=author)
	   # need more data!
	   # ...
           request = scrapy.http.Request(other_url)

If you're not done, say so
==========================

.. testcode::

   def parse(self, response):
       # ...
       for speaker in speakers:
           partial_item = PyConPreso(author=author)
	   # need more data!
	   # ...
           request = scrapy.http.Request(other_url)

Relevant snippet:

.. testcode::

    >>> import urlparse
    >>> urlparse.urljoin('http://example.com/my/site', '/newpath')
    'http://example.com/newpath'
    >>> urlparse.urljoin('http://example.com/my/site', 'subpath')
    'http://example.com/my/subpath'
If you're not done, say so
==========================

.. testcode::

   def parse(self, response):
       # ...
       for speaker in speakers:
           partial_item = PyConPreso(author=author)
	   # need more data!
	   # ...
           request = scrapy.http.Request(other_url)
	   request.meta['partial_item'] = partial_item
           yield request

If you're not done, say so
==========================

.. testcode::

   def parse(self, response):
       # ...
       for speaker in speakers:
           partial_item = PyConPreso(author=author)
	   # need more data!
	   # ...
           request = scrapy.http.Request(other_url,
                               callback=extract_next_part)
	   request.meta['partial_item'] = partial_item
           yield request

   def extract_next_part(response):
       partial_item = response.meta['partial_item']
       # do some work...
       partial_item['preso'] = _
       yield partial_item # now not partial!

If you're not done, say so
==========================

.. testcode::

   def parse(self, response):
       # ...
       for speaker in speakers:
           partial_item = PyConPreso(author=author)
	   # need more data!
	   # ...
           request = scrapy.http.Request(other_url,
                               callback=extract_next_part)
	   request.meta['partial_item'] = partial_item
           yield request

   def extract_next_part(response):
       partial_item = response.meta['partial_item']
       # do some work...
       partial_item['preso'] = _
       yield partial_item # now not partial!

Rule: Split the function if you need a new HTTP request.

Performance
===========

* Crawl 500 projects' bug trackers:
 * 26 hours

Performance
===========

* Crawl 500 projects' bug trackers:
 * 26 hours

* Add multiprocessing:
 * +1-10 MB * N workers

Performance
===========

* Crawl 500 projects' bug trackers:
 * 26 hours

* Add multiprocessing:
 * +1-10 MB * N workers

* After Scrapy:
 * N=200 simultaneous requests
 * 1 hour 10 min

====
Section: Testing
====


Data is complicated
===================

   >>> p.author
   'Asheesh Laroia, Jessica McKellar, Dana Bauer, Daniel Choi'

Why testing is normally hard
============================


.. testcode::
    ERROR: tests.test_thing

    Traceback (most recent call last):
     ...
     File "/usr/lib/python2.7/urllib2.py", line 1181, in do_open
        raise URLError(err)
    URLError: <urlopen error [Errno -2] Name or service not known>

    Ran 1 test in 0.153s

    FAILED (errors=1)

Why testing is normally hard
============================


.. testcode::
    ERROR: tests.test_thing

    Traceback (most recent call last):
     ...
     File "/usr/lib/python2.7/urllib2.py", line 1181, in do_open
        raise URLError(err)
    urllib2.HTTPError: HTTP Error 403: Exceeded query limit for API key

    Ran 1 test in 0.153s

    FAILED (errors=1)

Why testing is normally hard
============================

.. testcode::
    ERROR: tests.test_thing

    Traceback (most recent call last):
     ...
     File "/usr/lib/python2.7/urllib2.py", line 1181, in do_open
        raise URLError(err)
    URLError: <urlopen error [Errno 110] Connection timed out>

    Ran 1 test in 127.255s

    FAILED (errors=1)

Why testing is normally hard
============================

.. testcode::
    ERROR: tests.test_thing

    Traceback (most recent call last):
     ...
     File "/usr/lib/python2.7/urllib2.py", line 1181, in do_open
        raise URLError(err)
    URLError: <urlopen error [Errno 110] Connection timed out>

    Ran 1 test in 127.255s

    FAILED (errors=1)

mock.patch()?

Why testing is normally hard
============================

.. figure:: /_static/sad-commit.png
   :class: fill

Part V. Testing
===============

.. testcode::

    class PyConSiteSpider(BaseSpider):
        def parse(self, response):
	    # ...
            for speaker in speakers:
	        # ...
                yield PyConPreso(
		        author=author, preso=preso)

Part V. Testing
===============

.. testcode::

    class PyConSiteSpider(BaseSpider):
        def parse(self, response):
	    # ...
            for speaker in speakers:
	        # ...
                yield PyConPreso(
		        author=author, preso=preso)

test:

.. testcode::

    >>> spidey = PyConSiteSpider()
    >>> results = spidey.parse(response)

Part V. Testing
===============

.. testcode::

    class PyConSiteSpider(BaseSpider):
        def parse(self, response):
	    # ...
            for speaker in speakers:
	        # ...
                yield PyConPreso(
		        author=author, preso=preso)

test:

.. testcode::

    >>> spidey = PyConSiteSpider()
    >>> canned_response = HtmlResponse(url='', body=open('saved-data.html').read())
    >>> results = spidey.parse(canned_response)
    >>> assert list(results) == [PyConPreso(author=a, preso=b), ...]


Part V. Testing
===============

.. testcode::

    class PyConSiteSpider(BaseSpider):
        def parse(self, response):
	    # ...
            for speaker in speakers:
	        # ...
                yield PyConPreso(
		        author=author, preso=preso)

test:

.. testcode::

    def test_spider(self):
        expected = [PyConPreso(author=a, preso=b), ...]

        spidey = PyConSiteSpider()
	canned_response = HtmlResponse(url='', body=open('saved-data.html').read())
	results = list(spidey.parse(canned_response))
        self.assertEqual(expected, items)

...
===

.. figure:: /_static/scrapy-diagram-1.png
   :class: fill

More testing
============

.. testcode::

    def test_spider(self):
        url2filename = {'https://us.pycon.org/2013/schedule/':
                               'localcopy.html'}

	expected_data = [PyConPreso(author=a, preso=b), ...]


More testing
============

.. testcode::

    def test_spider(self):
        url2filename = {'https://us.pycon.org/2013/schedule/':
                               'localcopy.html'}

	expected_data = [PyConPreso(author=a, preso=b), ...]

        spidey = PyConSiteSpider()
        request_iterable = spider.start_requests()

More testing
============

.. testcode::

    def test_spider(self):
        url2filename = {'https://us.pycon.org/2013/schedule/':
                               'localcopy.html'}

	expected_data = [PyConPreso(author=a, preso=b), ...]

        spidey = PyConSiteSpider()
        request_iterable = spider.start_requests()

        ar = autoresponse.Autoresponder(
	         url2filename=url2filename,
                 url2errors={})

        items = ar.respond_recursively(request_iterable)

More testing
============

.. testcode::

    def test_spider(self):
        url2filename = {'https://us.pycon.org/2013/schedule/':
                               'localcopy.html'}

	expected_data = [PyConPreso(author=a, preso=b), ...]

        spidey = PyConSiteSpider()
        request_iterable = spider.start_requests()

        ar = autoresponse.Autoresponder(
	         url2filename=url2filename,
                 url2errors={})

        items = ar.respond_recursively(request_iterable)

	self.assertEqual(expected, items)

========
Section: Javascript
========

Three approaches
================

* Re-write the Javascript in Python

Three approaches
================

* Re-write the Javascript in Python

* Wrap some of the JS in spidermonkey

Three approaches
================

* Re-write the Javascript in Python

* Wrap some of the JS in spidermonkey

* Run it in an actual browser

JavaScript
==========

.. testcode::

    >>> import spidermonkey
    >>> r = spidermonkey.Runtime()

JavaScript
==========

.. testcode::

    >>> import spidermonkey
    >>> r = spidermonkey.Runtime()
    >>> ctx = r.new_context()

JavaScript
==========

.. testcode::

    >>> import spidermonkey
    >>> r = spidermonkey.Runtime()
    >>> ctx = r.new_context()
    >>> ctx.execute("{} + []")
    0

JavaScript
==========

.. testcode::

    >>> js_src = '''function (x) { return 3 + x; }'''
    >>> r = spidermonkey.Runtime()
    >>> ctx = r.new_context()
    >>> ctx.execute("{} + []")
    0
    >>> js_fn = cx.execute(js_src)
    >>> type(js_fn)
    <type 'spidermonkey.Function'>

JavaScript
==========

.. testcode::

    >>> js_src = '''function (x) { return 3 + x; }'''
    >>> r = spidermonkey.Runtime()
    >>> ctx = r.new_context()
    >>> ctx.execute("{} + []")
    0
    >>> js_fn = ctx.execute(js_src)
    >>> js_fn(3)
    6

JavaScript
==========

.. testcode::

    >>> js_src = '''function (x) { return 3 + x; }'''
    >>> r = spidermonkey.Runtime()
    >>> ctx = r.new_context()
    >>> js_fn = ctx.execute(js_src)
    >>> type(js_fn)
    <type 'spidermonkey.Function'>
    >>> js_fn(3)
    6

Get your source, e.g.

.. testcode::

    def parse(self, response):
       # to get a tag...
       script_content = doc.xpath('//script')[0].text_content()

JavaScript
==========

Also works for non-anonymous functions:

.. testcode::

    >>> js_src = '''function add_three(x) { return 3 + x; }'''
    >>> r = spidermonkey.Runtime()
    >>> ctx = r.new_context()
    >>> js_fn = ctx.execute(js_src)("add_three")
    >>> type(js_fn)
    <type 'spidermonkey.Function'>
    >>> js_fn(3)
    6

Hash cash
=========

.. figure:: /_static/hash-cash.png
   :class: fill

Hash cash
=========

.. figure:: /_static/hash-cash-2.png
   :class: fill

JavaScript
==========

.. testcode::

    import selenium
    class MySpider(BaseSpider):
        def __init__(self):
            self.browser = selenium.selenium(...) # configure
            self.browser.start() # synchronously launch

	def parse(self, response):
            self.browser.open(response.url) # GET by browser
	    self.browser.select('//ul') # in-browser XPath

=======
Section: API clients
=======

Basic non-HTML use
====

.. testcode::

    class WikiImageSpider(BaseSpider):
        START_URLS = ['http://en.wikipedia.org/w/api.php?action=query&titles=San_Francisco&prop=images&imlimit=20&format=json']

        def parse(self, response):
            results = json.loads(response.body_as_unicode)
            for image in results['query']['pages']['images']:
                 item = WikipediaImage(_) # ...
                 yield WikipediaImage

Basic non-HTML use
====

.. testcode::

    class WikiImageSpider(BaseSpider):
        START_URLS = ['http://en.wikipedia.org/w/api.php?action=query&titles=San_Francisco&prop=images&imlimit=20&format=json']

        def parse(self, response):
            results = json.loads(response.body_as_unicode)
            for image in results['query']['pages']['images']:
                 item = WikipediaImage(_) # ...
                 yield WikipediaImage
            if results['query-continue']['images']:
                new_url = response.url + _ # ...
	        yield scrapy.http.Request(new_url, callback=self.parse)

Basic non-HTML use
====

.. testcode::

    class WikiImageSpider(BaseSpider):
        START_URLS = ['http://en.wikipedia.org/w/api.php?action=query&titles=San_Francisco&prop=images&imlimit=20&format=json']

        def parse(self, response):
            results = json.loads(response.body_as_unicode)
            for image in results['query']['pages']['images']:
                 item = WikipediaImage(_) # ...
                 yield WikipediaImage
            if results['query-continue']['images']:
                new_url = response.url + _ # ...
	        yield scrapy.http.Request(new_url, callback=self.parse)

* settings.USER_AGENT = "My Wiki Bot mywikibot@asheesh.org"

* settings.RETRY_HTTP_CODES.append(403)

Basic non-HTML use
====

.. testcode::

    class WikiImageSpider(BaseSpider):
        START_URLS = ['http://en.wikipedia.org/w/api.php?action=query&titles=San_Francisco&prop=images&imlimit=20&format=json']

        def parse(self, response):
            results = json.loads(response.body_as_unicode)
            for image in results['query']['pages']['images']:
                 item = WikipediaImage(_) # ...
                 yield WikipediaImage
            if results['query-continue']['images']:
                new_url = response.url + _ # ...
	        yield scrapy.http.Request(new_url, callback=self.parse)

* settings.USER_AGENT = "My Wiki Bot mywikibot@asheesh.org"

* settings.RETRY_HTTP_CODES.append(403)

* Write a DownloaderMiddleware for custom retry logic

A setting for everything
========================

* settings.USER_AGENT

* settings.CONCURRENT_REQUESTS_PER_DOMAIN (= e.g. 1)

* settings.CONCURRENT_REQUEST (= e.g. 800)

* settings.RETRY_ENABLED (= True by default)

* settings.RETRY_TIMES

* settings.RETRY_HTTP_CODES

* Great intro-to-scraping docs

Best-case integration
=====================

* Leave your HTTP to Scrapy.

* Impatient? Steal data from item pipeline.

* Patient? Feed Exporter.

Twisted minus Twisted
=====================

.. figure:: /_static/garfield-minus.png
   :class: fill

==================================
Separate requesting and responding
==================================

.. figure:: /_static/take-away.jpg
   :class: fill

Asheesh Laroia asheesh.org
