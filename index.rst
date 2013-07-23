.. slideconf::
   :theme: single-level

=======================
Scrapy: it GETs the Web
=======================

Asheesh Laroia

asheesh@asheesh.org

Part 0. My history with scraping
================================

* 2004: Taught 3-week mini-class on mechanize + BeautifulSoup

* 2009: "Volunteer opportunity finder" within OpenHatch

* 2011: vidscraper. multiprocessing? gevent?

* 2012: oh-bugimporters rewrite w/ Scrapy

======
Part I: Scraping without scrapy
======

Web pages
=========

.. figure: /_static/rendered.png
   :class: fill

HTML source
===========

.. figure: /_static/view-source.png
   :class: fill

As diagram
==========

.. figure:: /_static/html-structure.gif
   :class: fill

DOM inspector
=============

.. figure: /_static/inspector.png
   :class: fill

Scraping in Python (2004)
=========================

.. testcode::

   >>> # get a web page
   >>> page = urllib2.urlopen('http://oscon.com/').read()

Scraping in Python (2004)
=========================

.. testcode::

   >>> # get a web page
   >>> page = urllib2.urlopen('http://oscon.com/').read()
   >>> # parse it
   >>> soup = BeautifulSoup.BeautifulSoup(page)

Scraping in Python (2004)
=========================

.. testcode::

   >>> # get a web page
   >>> page = urllib2.urlopen('http://oscon.com/').read()
   >>> # parse it
   >>> soup = BeautifulSoup.BeautifulSoup(page)
   >>> # find element we want
   >>> matches = soup('div', {'id': 'location_place'})

Scraping in Python (2004)
=========================

.. testcode::

   >>> # get a web page
   >>> page = urllib2.urlopen('http://oscon.com/').read()
   >>> # parse it
   >>> soup = BeautifulSoup.BeautifulSoup(page)
   >>> # find element we want
   >>> matches = soup('div', {'id': 'location_place'})
   >>> # pull out text
   >>> first = matches[0]
   >>> date_range = r[0].find(text=True)
   >>> print date_range
   u'July 22-26, 2013'

Scraping in Python (2004)
=========================

.. testcode::

   >>> # get a web page
   >>> page = urllib2.urlopen('http://oscon.com/').read()
   >>> # parse it
   >>> soup = BeautifulSoup.BeautifulSoup(page)
   >>> # find element we want
   >>> matches = soup('div', {'id': 'location_place'})
   >>> # pull out text
   >>> first = matches[0]
   >>> date_range = r[0].find(text=True)
   >>> print date_range
   u'July 22-26, 2013'
   >>> # store results somewhere
   >>> save_results(date_range)

* **Download** with urllib2, or requests, or mechanize, or ...

* **Examine** with browser inspectors

* **Parse** pages with lxml/BeautifulSoup

* **Select** with XPath or CSS selectors

Part II. Rewriting some non-scrapy code
================

Task: Get a list of speakers

Part II. Rewriting some non-scrapy code
================

Task: Get a list of speakers

.. testcode::

   SCHED_PAGE='https://us.pycon.org/2013/schedule/'

Part II. Rewriting some non-scrapy code
================

Task: Get a list of speakers

.. testcode::

   SCHED_PAGE='https://us.pycon.org/2013/schedule/'

   import requests
   import lxml.html

   data = requests.get(SCHED_PAGE)
   parsed = lxml.html.fromstring(data.content)
   print [x.text_content()
          for x in parsed.cssselect('span.speaker')]

Part II. Rewriting some non-scrapy code
================

Task: **Get a list of speakers and talk titles**

.. testcode::

   SCHED_PAGE='https://us.pycon.org/2013/schedule/'

   import requests
   import lxml.html

   data = requests.get(SCHED_PAGE)
   parsed = lxml.html.fromstring(data.content)
   print [x.text_content()
          for x in parsed.cssselect('span.speaker')]

Now capture preso titles
======================================

You **could**

.. testcode::

   def store_datum(author, preso_title):
       pass # actual logic here...

Now capture preso titles
========================

.. testcode::

   def store_datum(author, preso_title):
       pass # actual logic here...

   def main():
       data = requests.get(SCHED_PAGE)
       parsed = lxml.html.fromstring(data.response)
       for speaker_span in parsed.cssselect('span.speaker'):
           text = speaker_span.text_content()
           store_datum(author, preso_title)


Now capture preso titles
========================

.. testcode::

   def store_datum(author, preso_title):
       pass # actual logic here...

   def main():
       data = requests.get(SCHED_PAGE)
       parsed = lxml.html.fromstring(data.response)
       for speaker_span in parsed.cssselect('span.speaker'):
           text = speaker_span.text_content()
           store_datum(author, preso_title)


.. figure:: /_static/failure3.jpg
   :class: fill

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
    {'author': None,
     'preso':  None}

scrapy.items.Item
=================

.. testcode::

    class PyConPreso(scrapy.item.Item):
        author = Field()
        preso = Field()

.. testcode::

    # Similar to...
    {'author': None,
     'preso':  None}

::

   >>> p['preso_name'] = 'Asheesh'
   KeyError: 'PyConPreso does not support field: preso_name'


Better
======

.. testcode::

   def store_datum(author, preso_title):
       pass # actual logic here...

   def get_data():
       # ...
       for speaker_span in parsed.cssselect('span.speaker'):
           preso = None # FIXME
           text = speaker_span.text_content()
	   item = PyConPreso(
               author=text.strip(),
	       preso=store_datum(author, preso_title))
           out_data.append(item)
       return out_data

Data is complicated
===================

   >>> p.author
   'Asheesh Laroia, Jessica McKellar, Dana Bauer, Daniel Choi'

Data is complicated
===================

   >>> p.author
   'Asheesh Laroia, Jessica McKellar, Dana Bauer, Daniel Choi'

Scrapy-ify early on.

Maybe you'll need multiple HTTP requests. Maybe you'll just want
testable code.

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
                author = None # placeholder
                preso = None  # placeholder
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

Part III. An aside about Scrapy
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

    telnet localhost 6023

Awesome features...
===================

    telnet localhost 6023

Gives

    >>> est()
    Execution engine status
    time()-engine.start_time              : 21.3188259602
    engine.is_idle()                      : False
    …


Awesome features...
===================

    telnet localhost 6023

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

Semi-complex integration with other pieces of code.

Part IV. Async
==============

.. figure:: /_static/asink.jpg
   :class: fill

If you're not done, say so
==========================

.. testcode::

   def parse(response):
       # do some work...

If you're not done, say so
==========================

.. testcode::

   def parse(response):
       # do some work...
       req = request(new_url)
       yield req

If you're not done, say so
==========================

.. testcode::

   def parse(response):
       # do some work...
       req = request(new_url,
                     callback=next_page_handler)
       yield req

   def next_page_handler(response):
       # do some work...
       yield Item()

If you're not done, say so
==========================

.. testcode::

   def parse(response):
       # do some work...
       req = Request(new_url,
                     callback=next_page_handler)
       req.meta['data'] = 'to keep around'
       yield req

   def next_page_handler(response):
       data = response.meta['data'] # pull data out
       # do some work...
       yield Item()

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

    def test_spider():
        resp = HtmlResponse(url='', body=open('saved-data.html').read())
        spidey = PyconSiteSpider()
        expected = [PyConPreso(author=a, preso=b), ...]
        items = list(spidey.parse(resp))
        assert items == expected

More testing
============

.. testcode::

    def test_spider(self):
        spidey = PyConSiteSpider()
        request_iterable = spider.start_requests()
        url2filename = {'http://example.com/':
                               'path/to/sample.html'}

	expected = [...]

        ar = autoresponse.Autoresponder(
	         url2filename=url2filename,
                 url2errors={})
        items = ar.respond_recursively(request_iterable)

	self.assertEqual(expected, items)

Part VI. Wacky tricks
=====================

A setting for everything
========================

* settings.USER_AGENT

* settings.CONCURRENT_REQUESTS_PER_DOMAIN (= e.g. 1)

* settings.CONCURRENT_REQUEST (= e.g. 800)

* settings.RETRY_ENABLED (= True by default)

* settings.RETRY_TIMES

* settings.RETRY_HTTP_CODES

* Great intro-to-scraping docs

JavaScript
==========

.. testcode::

    import spidermonkey

    def parse(self, response):
       # to get a tag...
       script_content = doc.xpath('//script')[0].text_content()
       # to run the JS...
       r = spidermonkey.Runtime()
       ctx = r.new_context()
       n = cx.eval_script("1 + 2") + 3
       # n == 6


JavaScript
==========

.. testcode::

    import spidermonkey

    def parse(self, response):
       script_content = doc.xpath('//script')[0].text_content() # get tag
       r = spidermonkey.Runtime()
       ctx = r.new_context()
       n = cx.eval_script(script_content) # execute script

    import selenium
    class MySpider(BaseSpider):
        def __init__(self):
            self.browser = selenium.selenium(...) # configure
            self.browser.start() # synchronously launch

	def parse(self, response):
            self.browser.open(response.url) # GET by browser
	    self.browser.select('//ul') # in-browser XPath

Django
======

.. testcode::

   from scrapy.contrib.djangoitem import DjangoItem

Django
======

.. testcode::

   from scrapy.contrib.djangoitem import DjangoItem
   from myapp.models import Poll

   # in scrapy items.py
   class PollItem(DjangoItem):
       django_model = Poll

Django
======

.. testcode::

   from scrapy.contrib.djangoitem import DjangoItem
   from myapp.models import Poll

   # in scrapy items.py
   class PollItem(DjangoItem):
       django_model = Poll

   # in scrapy pipelines.py
   class PollPipeline(object):
       def process_item(self, item, spider):
           item.save()

Django
======

.. testcode::

   from scrapy.contrib.djangoitem import DjangoItem
   from myapp.models import Poll

   # in scrapy items.py
   class PollItem(DjangoItem):
       django_model = Poll

   # in scrapy pipelines.py
   class PollPipeline(object):
       def process_item(self, item, spider):
           item.save()

Or just write a Django management command to deal with the JSON.

Best-case integration
=====================

* Leave your HTTP to Scrapy.

* Impatient? Item Pipeline.

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

Asheesh Laroia

