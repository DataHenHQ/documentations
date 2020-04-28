*******
How-Tos
*******

The following are a list of commonly occuring DataHen use cases and how to do them. It is useful to skim through all the how-tos so that in the future, if you come across any similar need, you can refer back to this section.


Setting a scraper’s scheduler
=============================

You can schedule a scraper’s scheduler in as granular detail as you want. However, we only support granularity down to the Minute.
We currently support the CRON syntax for scheduling.
You can set the ‘schedule’ field or the ‘timezone’ to specify the timezone when the job will be started. If timezone is not specified, it will default to “UTC”. Timezone values are IANA format. Please see the list of available timezones.
The scheduler also has the option to cancel current job, anytime it starts a new one. By default a scraper’s scheduler does not start a new job if there is an already existing job that is active.

.. code-block:: bash

   $ hen scraper create <scraper_name> <git_repo> --schedule "0 1 * * * *" --timezone "America/Toronto" --cancel-current-job
   $ hen scraper update <scraper_name> --schedule "0 1 * * * *" --timezone "America/Toronto"

The following are allowed CRON values:

+--------------+------------+-----------------+----------------------------+
| Field Name   | Mandatory? | Allowed Values  | Allowed Special Characters |
+==============+============+=================+============================+
| Minutes      | Yes        | 0-59            | \* / , -                   |
+--------------+------------+-----------------+----------------------------+
| Hours        | Yes        | 0-23            | \* / , -                   |
+--------------+------------+-----------------+----------------------------+
| Day of month | Yes        | 1-31            | \* / , - L W               |
+--------------+------------+-----------------+----------------------------+
| Month        | Yes        | 1-12 or JAN-DEC | \* / , -                   |
+--------------+------------+-----------------+----------------------------+
| Day of week  | Yes        | 0-6 or SUN-SAT  | \* / , - L #               |
+--------------+------------+-----------------+----------------------------+
| Year         | No         | 1970–2099       | \* / , -                   |
+--------------+------------+-----------------+----------------------------+

| Asterisk ( * )
| The asterisk indicates that the cron expression matches for all values of the field. E.g., using an asterisk in the 4th field (month) indicates every month.

| Slash ( / )
| Slashes describe increments of ranges. For example 3-59/15 in the minute field indicate the third minute of the hour and every 15 minutes thereafter. The form \*/... is equivalent to the form "first-last/...", that is, an increment over the largest possible range of the field.

| Comma ( , )
| Commas are used to separate items of a list. For example, using MON,WED,FRI in the 5th field (day of week) means Mondays, Wednesdays and Fridays.

| Hyphen ( - )
| Hyphens define ranges. For example, 2000-2010 indicates every year between 2000 and 2010 AD, inclusive.

| L
| L stands for "last". When used in the day-of-week field, it allows you to specify constructs such as "the last Friday" (5L) of a given month. In the day-of-month field, it specifies the last day of the month.

| W
| The W character is allowed for the day-of-month field. This character is used to specify the business day (Monday-Friday) nearest the given day. As an example, if you were to specify 15W as the value for the day-of-month field, the meaning is: "the nearest business day to the 15th of the month."

So, if the 15th is a Saturday, the trigger fires on Friday the 14th. If the 15th is a Sunday, the trigger fires on Monday the 16th. If the 15th is a Tuesday, then it fires on Tuesday the 15th. However if you specify 1W as the value for day-of-month, and the 1st is a Saturday, the trigger fires on Monday the 3rd, as it does not 'jump' over the boundary of a month's days.

The W character can be specified only when the day-of-month is a single day, not a range or list of days.

The W character can also be combined with L, i.e. LW to mean "the last business day of the month."

| Hash ( # )
| # is allowed for the day-of-week field, and must be followed by a number between one and five. It allows you to specify constructs such as "the second Friday" of a given month.

Changing a Scraper’s or a Job’s Proxy Type
==========================================

We support many types of proxies to use:

+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| Proxy Type             | Description                                                                                                                             |
+========================+=========================================================================================================================================+
| standard               | The standard rotating proxy that gets randomly used per request. This is the default.                                                   |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+

.. code-block:: bash

   $ hen scraper update <scraper_name> --proxy-type sticky1

Keep in mind that the above will only take effect when a new scrape job is created.

To change a proxy of an existing job, first cancel the job, and then change the proxy_type, and then resume the job:

.. code-block:: bash

   $ hen scraper job cancel <scraper_name>
   $ hen scraper job update <scraper_name> --proxy-type sticky1
   $ hen scraper job resume <scraper_name>

Setting a specific ruby version
===============================

By default our ruby version that we use is 2.4.4, however if you want to specify a different ruby version you can do so by creating a .ruby-version file on the root of your project directory.

NOTE: we currently only allow the following ruby versions:

* 2.4.4
* 2.5.3
* If you need a specific version other than these, please let us know

Setting a specific Ruby Gem
===========================

To add dependency to your code, we use Bundler. Simply create a Gemfile on the root of your project directory.

.. code-block:: bash

   $ echo "gem 'roo', '~> 2.7.1'" > Gemfile
   $ bundle install # this will create a Gemfile.lock
   $ ls -alth | grep Gemfile
   total 32
   -rw-r--r--   1 johndoe  staff    22B 19 Dec 23:43 Gemfile
   -rw-r--r--   1 johndoe  staff   286B 19 Dec 22:07 Gemfile.lock
   $ git add . # and then you should commit the whole thing into Git repo
   $ git commit -m 'added Gemfile'
   $ git push origin

Changing a Scraper’s Standard worker count
==========================================

The more workers you use on your scraper, the faster your scraper will be.
You can use the command line to change a scraper’s worker count:

.. code-block:: bash

   $ hen scraper update <scraper_name> --workers N

Keep in mind that this will only take effect when a new scrape job is created.

Changing a Scraper’s Browser worker count
=========================================

The more workers you use on your scraper, the faster your scraper will be.
You can use the command line to change a scraper’s worker count:

.. code-block:: bash

   $ hen scraper update <scraper_name> --browsers N

NOTE: Keep in mind that this will only take effect when a new scrape job is created.

Changing an existing scrape job’s worker count
==============================================

You can use the command line to change a scraper job’s worker count:

.. code-block:: bash

   $ hen scraper job update <scraper_name> --workers N --browsers N

This will only take effect if you cancel, and resume the scrape job again:

.. code-block:: bash

   $ hen scraper job cancel <scraper_name> # cancel first
   $ hen scraper job resume <scraper_name> # then resume

Enqueueing a page to Browser Fetcher’s queue
============================================

You can enqueue a page like so in your script. The following will enqueue a headless browser:

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     fetch_type: "browser" # This will enqueue headless browser
   }

Or use the command line:

.. code-block:: bash

   $ hen scraper page add <scraper_name> <url> --fetch-type browser

You can enqueue a page like so in your script. The following will enqueue a full browser (non-headless):

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     fetch_type: "fullbrowser" # This will enqueue headless browser
   }

Or use the command line:

.. code-block:: bash

   $ hen scraper page add <scraper_name> <url> --fetch-type fullbrowser

Setting fetch priority to a Job Page
====================================

The following will enqueue a higher priority page.
NOTE: You can only create a page that has priority, not update an existing page with a new priority value on the script. Also, updating a priority only works via the command line tool.

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     priority: 1 # defaults to 0. Higher numbers means will get fetched sooner
   }

Or use the command line:

.. code-block:: bash

   $ hen scraper page add <scraper_name> <url> --priority N
   $ hen scraper page update <job> <gid> --priority N

Setting a user-agent-type of a Job Page
=======================================

You can enqueue a page like so in your script:

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     ua_type: "desktop" # defaults to desktop, other available is mobile.
   }

Or use the command line:

.. code-block:: bash

   $ hen scraper page add <scraper_name> <url> --ua-type mobile

Setting the request method of a Job Page
========================================

You can enqueue a page like so in your script:

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     method: "POST" # defaults to GET.
   }

Or use the command line:

.. code-block:: bash

   $ hen scraper page add <scraper_name> <url> --method GET

Setting the request headers of a Job Page
=========================================

You can enqueue a page like so in your script:

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     headers: {"Cookie": "name=value; name2=value2; name3=value3"} # set this
   }

Or use the command line:

.. code-block:: bash

   $ hen scraper page add <scraper_name> <url> --headers '{"Cookie": "name=value; name2=value2; name3=value3"}'

Setting the request body of a Job Page
======================================


You can enqueue a page like so in your script:

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     body: "your request body here" # set this field
   }

Or use the command line:

.. code-block:: bash

   $ hen scraper page add <scraper_name> <url> --body 'your request body here'

Setting the page_type of a Job Page
===================================

You can enqueue a page like so in your script:

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     page_type: "page_type_here" # set this field
   }

Or use the command line:

.. code-block:: bash

   $ hen scraper page add <scraper_name> <url> --page-type page_type_here

Reset a Job Page
================

You can reset a scrape-job page’s parsing and fetching from the command line:

.. code-block:: bash

   $ hen scraper page reset <scraper_name> <gid>

You can also reset a page from any parser or seeder script by setting the `reset` field to true while enqueueing it, like so:

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     reset: true # set this field
   }

Handling cookies
================

There are two ways to handle cookies in DataHen, at a lower level via the Request and Response Headers, or at a higher level via the Cookie Jar.

Low level cookie handling using Request/Response Headers
--------------------------------------------------------

To handle cookie at a lower level, you can set the “cookie” on the request header:

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     headers: {"Cookie": "name=value; name2=value2; name3=value3"},
   }

You can also read cookies by reading the “Set-Cookie” response headers:

.. code-block:: ruby

   page['response_headers']['Set-Cookie']

High level cookie handling using the Cookie Jar
-----------------------------------------------

To handle cookie at a higher level, you can set the “cookie” field directly onto the page, and it will be saved onto the Cookie Jar during that request.

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     cookie: "name=value; name2=value2; name3=value3",
   }

You can also do so from the command line:

.. code-block:: bash

   $ hen scraper page add <scraper_name> <url> --cookie "name=value; name2=value2"

You can then read the cookie from the cookiejar by:

.. code-block:: ruby

   page['response_cookie']

This method above is reading from the cookiejar. This is especially useful when a cookie is set by the target-server during redirection.

Force Fetching a specific unfresh page
======================================

To enqueue a page and have it force fetch, you need to set freshness field, and force_fetch field. Freshness should only be now, or in the past. It cannot be in the future. Basically it is “how much time ago, that you consider this page as fresh”
One thing to keep in mind, that this only resets the page fetch, it does nothing to your parsing of pages, whether the parser has executed or not.
In your parser script you can do the following:

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     freshness: "2018-12-12T13:59:29.91741Z", # has to be this string format
     force_fetch: true
   }

You can do this to find one output result or use the command line to query an output:

.. code-block:: bash

   $ hen scraper page add <scraper_name> <url> --page-type page_type_here --force-fetch --freshness "2018-12-12T13:59:29.91741Z"

Handling JavaScript
===================

To do javascript rendering, please use the Browser Fetcher.
First you need to add a browser worker onto your scraper:

.. code-block:: bash

   $ hen scraper update <scraper_name> --browsers 1

Next, for every page that you add, you need to specify the correct fetch_type:

.. code-block:: bash

   $ hen scraper page add <scraper_name> <url> --fetch-type browser

Or in the script, by doing the following:

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     fetch_type: "browser"
   }

Browser display
===============

We support display size configuration within Browser Fetcher having 1366x768 as default size. This feature is quite useful when interacting with responsive websites and taking screenshots. Only `browser` and `fullbrowser` fetch types support this feature.

First you need to add a browser worker onto your scraper:

.. code-block:: bash

   $ hen scraper update <scraper_name> --browsers 1

This example shows you how to change the browser display size to 1920x1080:

.. code-block:: ruby

   pages << {
     "url": "https://www.datahen.com",
     "page_type": "homepage",
     "fetch_type": "browser",
     "display": {
       "width": 1920,
       "height": 1080
     }
   }

Browser interaction
===================

We support browser interaction through `Puppeteer <https://pptr.dev/>`_ and Browser Fetcher. Only `browser` and `fullbrowser` fetch types support this feature.

We fully support JS puppeteer's `page object <https://pptr.dev/#?product=Puppeteer&version=v2.1.1&show=api-class-page>`_ and provide a predefined `sleep(miliseconds)` async function to allow easy browser interaction and actions.  

First you need to add a browser worker onto your scraper:

.. code-block:: bash

   $ hen scraper update <scraper_name> --browsers 1

Next you will need to add your puppeteer javascript code to interact with your browser fetch when enqueuing your page inside your seeder or parser scripts.

This example shows you how to click the first footer link and wait 3 seconds after the page has load:

.. code-block:: ruby

   pages << {
     "url": "https://www.datahen.com",
     "page_type": "footer_page",
     "fetch_type": "browser",
     "driver": {
       "code": "await page.click('footer ul > li > a'); await sleep(3000);"
     }
   }

Notice that modifying your driver code will generate the same GID, to change this, assign driver's `name` attribute.

Enqueue same page twice with diffeent code
------------------------------------------

Sometimes, you will need to scrape the same page more than one time but interact with it on a different way, therefore, `driver.code` attribute alone will generate same GID everytime when using the same page configuration.

To fix this, use `driver.name` attribute as a unique identifier to your `driver.code` and change the GID.

This example shows you how to enqueue the same page twice with different browser interaction by using `name` attribute, notice that each enqueued page will now generate it's own unique GID:

.. code-block:: ruby

   pages << {
     "url": "https://www.datahen.com",
     "page_type": "footer_page",
     "fetch_type": "browser",
     "driver": {
       "name": "click first footer link"
       "code": "await page.click('footer ul > li > a'); await sleep(3000);"
     }
   }

   pages << {
     "url": "https://www.datahen.com",
     "page_type": "footer_page",
     "fetch_type": "browser",
     "driver": {
       "name": "click second footer link"
       "code": "await page.click('footer ul > li + li > a'); await sleep(3000);"
     }
   }

Change browser fetch behavior
-----------------------------

We have a 30 seconds default timeout on each browser fetch therefore, you might find that some pages having timeout on Browser Fetcher because of heavy resources taking too much time to load or maybe a heavy loading API response, that will likely cause your pages to fail.

To fix this, change your page browser timeout to be as long as you need by using `driver.goto_options`. This example shows you how to increase your page browser timeout to 50 seconds:

.. code-block:: ruby

   pages << {
     "url": "https://www.datahen.com",
     "page_type": "homepage",
     "fetch_type": "browser",
     "driver": {
       "goto_options": {
         "timeout": 50000
       }
     }
   }


`driver.goto_options` attribute fully supports puppeteer's `page.goto` `options` param, you can learn more about it `here <https://pptr.dev/#?product=Puppeteer&version=v2.1.1&show=api-pagegotourl-options>`_.

Dealing with responsive designs
-------------------------------

Response designs are quite common along websites, which makes it a common problem when comes to browser interaction click actions on elements that would be hidden on smaller or bigger screen sizes.

This example shows you how to use `display` options to set your browser display size to mobile portrait and then click on a menu option from a response website:

.. code-block:: ruby

   pages << {
     "url": "https://www.datahen.com",
     "page_type": "mobile_blog",
     "fetch_type": "browser",
     "display": {
       "width": 320,
       "height": 480
     }
     "driver": {
       "code": "await page.click('hamburger-toggle'); await sleep(3000); page.click('.menu-horizontal > li + li + li+ li + li + li > a')"
     }
   }

Dealing with infinite load timeouts
-----------------------------------

There are some weird scenarios on which a website will just never finish loading becuase a buggy resource or a never ending JS script loop, that will trigger a timeout no matter how much you wait.

A good way to deal with these weird scenarios is to use puppeteer's goto option `domcontentloaded` and our predefined sleep async function.
The next example shows you how to combine these two options into a working solution by manually waiting 3 seconds for the page to load:

.. code-block:: ruby

   pages << {
     "url": "https://www.datahen.com",
     "page_type": "homepage",
     "fetch_type": "browser",
     "driver": {
       "code": "await sleep(3000);",
       "goto_options": {
         "waitUntil": "domcontentloaded"
       }
     }
   }

Taking screenshots
==================

We support browser screenshots within Browser Fetcher by enabling `screenshot.take_screenshot` attirbute. It is important to note that taking a screenshot will replace the page `content` with the screenshot binary contents. Only `browser` and `fullbrowser` fetch types support this feature.

First you need to add a browser worker onto your scraper:

.. code-block:: bash

   $ hen scraper update <scraper_name> --browsers 1

Next you need to enqueue your page with `screenshot.take_screenshot` attribute enabled. This example shows you how to take a screenshot:

.. code-block:: ruby

   # ./seeder/seeder.rb
   pages << {
     "url": "https://www.datahen.com",
     "page_type": "homepage",
     "fetch_type": "browser",
     "screenshot": {
       "take_screenshot": true,
       "options": {
        "fullPage": false,
        "type": "jpeg",
        "quality": 75
      }
     }
   }

This will replace the page's html source code at "content" variable with the screenshot binary.

This example shows you how to save the screenshot to an AWS S3 bucket, but first, let's create our prerequisites, `Gemfile` and `config.yml` files:

.. code-block:: ruby

   # Gemfile
   source 'https://rubygems.org'
   gem 'datahen'
   gem 'aws-sdk-s3'

.. code-block:: yaml

   # config.yml
   seeder:
     file: ./seeder/seeder.rb
     disabled: false
   parsers:
     - file: ./parser/upload_to_s3.rb
       page_type: my_screenshot
       disabled: false

Now we can upload our screenshot to AWS S3 to our `my_bucket` bucket as `my_screenshot.jpeg`:

.. code-block:: ruby

   # ./parser/upload_to_s3.rb
   require 'aws-sdk-s3'
   
   s3 = Aws::S3::Resource.new()
   obj = s3.bucket('your_bucket').object('my_screenshot.jpeg')
   obj.put(body: content)

Screenshot options
------------------

We support all options from puppeteer's `page.screenshot` `options` params other than `path` and `encoding` due internal handling. You can learn more about it `here <https://pptr.dev/#?product=Puppeteer&version=v2.1.1&show=api-pagescreenshotoptions>`_.

This example shows you how to take a full page screenshot as `JPEG`:

.. code-block:: ruby

   pages << {
     "url": "https://www.datahen.com",
     "page_type": "homepage",
     "fetch_type": "browser",
     "screenshot": {
       "take_screenshot": true,
       "options": {
        "fullPage": true,
        "type": "jpeg",
        "quality": 75
      }
     }
   }

And this example shows you how to take a 800x600 display size screenshot as `PNG`:

.. code-block:: ruby

   pages << {
     "url": "https://www.datahen.com",
     "page_type": "homepage",
     "fetch_type": "browser",
     "display": {
       "width": 800,
       "height": 600
     }
     "screenshot": {
       "take_screenshot": true,
       "options": {
        "fullPage": false,
        "type": "png"
      }
     }
   }
   
Notice that `PNG` screenshots doesn't support `screenshot.quality` attribute, more information about it `here <https://pptr.dev/#?product=Puppeteer&version=v2.1.1&show=api-pagescreenshotoptions>`_.

Screenshots and browser interaction
-----------------------------------

Screenshots and Browser Fetch interaction are compatible, so you can use both to interact with your page before taking a screenshot.

This example shows you how to take a screenshot of `duckduckgo.com` homepage after showing it's side menu at 1920x1080:

.. code-block:: ruby

   pages << {
     "url": "https://www.datahen.com",
     "page_type": "homepage",
     "fetch_type": "browser",
     "display": {
       "width": 1920,
       "height": 1080
     }
     "driver": {
       "code": "page.click('.js-side-menu-open'); await sleep(3000);"
     },
     "screenshot": {
       "take_screenshot": true,
       "options": {
        "fullPage": false,
        "type": "png"
      }
     }
   }



Doing dry-run of your script locally
====================================

Using the `try` command will allow you dry-run a parser or a seeder script locally. How it works is, it downloads necessary data from the DataHen cloud, and then executes your script locally, but it does not upload any data back to the DataHen Cloud.

.. code-block:: bash

   $ hen parser try ebay parsers/details.rb
   $ hen seeder try ebay seeder/seeder.rb

Executing your script locally, and uploading to DataHen
=======================================================

Using the `exec` command will allow you execute a parser or a seeder script locally and upload the result to the DataHen cloud. It works by downloading the necessary data from the DataHen cloud, and executes it locally. When done it will upload the resulting output and pages back onto the DataHen cloud.

.. code-block:: bash

   $ hen parser exec <scraper_name> <parser_file> <gid>
   $ hen seeder exec <scraper_name> <seeder_file>

The `exec` command is really useful to do end-to-end testing on your script, to ensure that not only the execution works, but also if it properly uploads the resulting data to the DataHen cloud.
Any errors that are generated during the exec command, will be logged onto the DataHen cloud’s log, so it is accessible in the following way

.. code-block:: bash

   $  hen scraper log <scraper_name>
   $  hen scraper page log <scraper_name> <gid>

Once you’ve successfully executed the command locally using `exec` you can check your stats, and collection lists and outputs using the command

.. code-block:: bash

   $ hen scraper stats <scraper_name>
   $ hen scraper output collection <scraper_name>
   $ hen scraper output list <scraper_name> --collection <collection_name>

Querying scraper outputs
========================

We currently support the ability to query a scraper outputs based on arbitrary JSON key. Only exact matches are currently supported.
In your parser script you can do the following to find many output results:

.. code-block:: ruby

   # find_outputs(collection='default', query={}, page=1, per_page=30)
   # will return an array of output records
   records = find_outputs('foo_collection', {"_id":"123"}, 1, 500}

Or you can do this to find one output result:

.. code-block:: ruby

   # find_output(collection='default', query={})
   # will return one output record
   record = find_output('foo_collection', {"_id":"123"}}

Or use the command line, to query an output:

.. code-block:: bash

   $ hen scraper output list <scraper_name> --collection home --query '{"_id":"123"}'

You can also query outputs from another scraper or job:
To find output from another job, do the following:

.. code-block:: ruby

   records = find_outputs('foo_collection', {"_id":"123"}, 1, 500, job_id: 1234}

To find output from another scraper, do the following:

.. code-block:: ruby

   records = find_outputs('foo_collection', {"_id":"123"}, 1, 500, scraper_name:'my_scraper'}

Restart a scraping job
======================

To restart a job, you need to cancel an existing job first, then start a new one:

.. code-block:: bash

   $ hen scraper job cancel <scraper_name>
   $ hen scraper start <scraper_name>

Setting Variables and Secrets to your Account, Scrapers, and Jobs
=====================================================

The DataHen platform supports Variables and Secrets that you can store in your account, scrapers, and jobs.

Variables (or Secrets) that are stored in your **account** are called **Environment Variables**.

Variables (or Secrets) that are stored in your **scraper** or **jobs** are called **Input Variables**.

**What is the difference between Environment Variables and Input Variables?**

Environment Variables are useful in sharing variables accross scrapers. For example, if you need to have multiple scrapers to push data to the same external database, instead of setting the same variables over and over again, you can just store them in the account, and your scrapers can access them. 

Input Variables on the other hand, are only settable on the scraper or on the job itself. Input variables also allow the scraper's Web UI to display an input form, so that the users of your scrapers does not need to modify the code anytime they want to specify a variable.

**What is the difference between Variables and Secrets?**

Variables are used to store information to be referenced and manipulated, whereas Secrets are simply Variables that are encrypted.

Secrets are useful for storing passwords, or connection strings to an external Database, which will make your code more secure and more reusable. 

Regardless of whether you store the Variables (or secrets) in your account, scrapers, or jobs, they are all equally accessable in any of your seeder, parser, or finisher scripts, provided that you have modified your config.yaml file to do so.


Setting Environment Variables and Secrets on your account.
----------------------------------------------------------

You can set any environment variables and secrets in your account that you can then use in any of your scrapers or jobs.


This `example scraper <https://github.com/DataHenOfficial/ebay-scraper/tree/env_vars>`_ shows usage of environment variables.

There are three steps that you need to do in order to use environment variables and secrets:

1. Set the environment variable or secrets on your account.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
To set an environment variable using command line:

.. code-block:: bash

   $ hen var set <var_name> <value>

To set a secret environment variable using command line:

.. code-block:: bash

   $ hen var set <var_name> <value> --secret


2. Change your config.yaml to use the variables or secrets.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Add the following to your config.yaml file.

.. code-block:: yaml

   env_vars:
    - name: foo
      global_name: bar # Optional. If specified, refers to your account's environment variable of this name.
      disabled: false # Optional
    - name: baz
      default: bazvalue

In the example above, this will search for your account's environment variable of ``bar`` and then make it available to your script as ``ENV['foo']``.
The above example also will search for ``baz`` variable on your account, and make it available to your script as ``ENV['baz']``.

IMPORTANT: The name of the env var must be the same as the env var that you have specified in your account in step 1. If You intend to use a different variable name in the scraper vs in the account, use ``global_name``.



3. Access the environment variables and secrets in your script.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Once you've done step 1 and 2 above, you can then access the environment variables or secrets from any of your seeder, parser, finisher scripts, by doing so:

.. code-block:: ruby

   ENV['your_env_var_here']



Setting Input Variables and Secrets on your scraper and scrape job.
-------------------------------------------------------------------

You can set any input variables and secrets on your scraper, similar to how you use environment variables.

When you've specified your input variables on your scraper, any jobs that gets created will contain the variables that are copied from the scraper.

This `example scraper <https://github.com/DataHenOfficial/ebay-scraper/tree/input_vars>`_ shows usage of input variables.

There are three steps that you need to do in order to use input variables and secrets:

1. Set the input variable or secrets on your scraper.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
To set an input variable on a scraper using command line:

.. code-block:: bash

   $ hen scraper var set <var_name> <value>

To set a secret input variable on a scraper using command line:

.. code-block:: bash

   $ hen scraper var set <var_name> <value> --secret

To set an input variable on a scrape job using command line:

.. code-block:: bash

   $ hen scraper job var set <var_name> <value>

IMPORTANT: For this to take effect. You must pause and resume the job


To set a secret input variable on a scraper job using command line:

.. code-block:: bash

   $ hen scraper job var set <var_name> <value> --secret

IMPORTANT: For this to take effect. You must pause and resume the job


2. Change your config.yaml to use the variables or secrets.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Add the following to your config.yaml file.

.. code-block:: yaml

   input_vars:
    - name: starting_url
      title: Starting URL # Optional
      description: Enter the starting URL for the scraper to run # optional
      default: https://www.ebay.com/sch/i.html?_nkw=macbooks # optional.
      type: text # Available values include: string, text, secret, date, datetime. This will display the appropriate input on the form.
      required: false # Optional. This will make the input field in the form, required
      disabled: false # Optional
    - name: baz

In the example above, this will search for your scrape job's input variable of ``starting_url`` and then make it available to your script as ``ENV['starting_url']``.
The above example also will search for ``baz`` variable on your scrape job, and make it available to your script as ``ENV['baz']``.


3. Access the input variables and secrets in your script.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Once you've done step 1 and 2 above, you can then access the input variables or secrets from any of your seeder, parser, finisher scripts, by doing so:

.. code-block:: ruby

   ENV['your_input_var_here']



Using a custom docker image for the scraper
===========================================

We support docker image where the scraper will run on. What this means, is that you can install any dependencies that you’d like on it. Please let the DataHen support know so that this can be created for you.


IMPORTANT: Only docker images that are compatible with DataHen can be run. Please contact us for more info.

Our base Docker image is based on Alpine 3.7:

.. code-block:: ruby

   FROM alpine:3.7

So, if you want a package to be installed, make sure that it builds correctly on your local machine first.

Once correctly built, please let us know what dockerfile commands to add to the custom image.
The following format would be preferable:

.. code-block:: bash

   RUN apk add --update libreoffice

Once we have built the image for you, you can use this custom image by modifying your config.yaml file and include the following line:

.. code-block:: bash

   scraper_image: <url-to-your-docker-image>

When you have modified this and deploy this, you need to restart your job.

How to use shared code libraries from other Git repositories using Git Submodule
================================================================================

Sometimes you want to have a scraper that has a shared list of libraries that are used by other scrapers in other Git repositories.
Luckily DataHen supports Git Submodules, which enables this scenario.

You simply just deploy a scraper as usual, and DataHen will take care of initating and checking out the submodules recursively.

This is `the documentation on Git Submodules <https://git-scm.com/book/en/v2/Git-Tools-Submodules>`_ that shows the usage in depth.

This `example scraper <https://github.com/DataHenOfficial/ebay-scraper/tree/submodule>`_ shows usage of git submodules.

How to debug page fetch
=======================
Debugging page fetch can be both easy and hard, depending on how much work you need to find the cause of the problem. You will find here some common and uncommon page fetching issues that happens on websites along it's fixes:

`no_url_encode: true`
---------------------
This option forces a page to keep it's url as is, since DataHen decode and re-encode the url so it fix any error on it by default, useful to standardize the url for cache.

**Example:**

.. code-block:: ruby

   pages << {
     'url' => 'https://example.com/?my_sensitive_value'
   }
   # => url is re-encoded as "https://example.com/?my_sensitive_value="

   pages << {
     'url' => 'https://example.com/?my_sensitive_value',
     'no_url_encode' => true
   }
   # => url is left as is "https://example.com/?my_sensitive_value"

`http2: true`
-------------
This change the standard fetch from HTTP/1 to HTTP/2, which not only makes fetch faster on websites that support it, but also helps to bypass some anti-scrape tech that usually blocks HTTP/1 requests.

**Example:**

.. code-block:: ruby

   pages << {
     'url' => 'https://example.com'
   }
   # => page fetching will use HTTP/1

   pages << {
     'url' => 'https://example.com',
     'http2' => true
   }
   # => page fetching will use HTTP/2


response headers and request headers are different
--------------------------------------------------
There has been a few times on that a dev includes a response header within  `headers: {}`  causing the fetch to fail on websites that validates the headers it receives, so try to check which headers your browser shows on dev tools to see if an extra header is being used by mistake.

**Example:**
let's say a page enqueues this way

.. code-block:: ruby

   pages << {'url' => '[https://www.example.com](https://www.example.com/)'}

then it will fetch, and then got response_headers like

.. code-block:: ruby

   response_headers: {
     'content-type' => 'json'
   }

so on next page you enqueue the following page adding one or more response headers by mistake

.. code-block:: ruby

   pages << {
     'url' => 'https://www.example.com/abc'
     'headers' => {
       'content-type' => 'json'
     }
   }

On this example, using `content-type` is fine on request as long as it is POST method, but this one is GET, so on this case this would be invalid and a website that validates the headers will fail.

bzip compression headers
------------------------
Most browsers will include a headers indicating to compress the page to bzip or other compression format, most of the times it will not affect anything, but there are a few on which including these headers, will cause the content to fail.



Advanced Usage
==============

Parsing Failed Responses
------------------------

DataHen comes with a lot of safety harnesses to make scraping easy and delightful for developers. What this means is, we only allow for successfully (200 HTTP Status) fetched pages to be parsed.
However, if you do need to go down into the detail and deal with your own failed pages, or other type of responses, we allow you to do so.
On your config.yaml, add the following:

.. code-block:: yaml

   parse_failed_pages: true

After doing the above, don’t forget to deploy your scraper, and restart your job.

We have now removed your safety harnesses.
From now on, you have to deal with your own page reset, and page response statuses.
Typically, you should have your parser deal with two kinds of responses, successful and failed ones.
Look at the following example parser file on how we deal with the different responses in the same parser:

.. code-block:: ruby

   if page['response_status_code'] # if response is successful
      body = Nokogiri.HTML(content)
   elsif page['failed_response_status_code'] # if response is not successful
      body = Nokogiri.HTML(failed_content)
   end

   doc = {
      text: body.text,
      url: page['url']
   }

   outputs << doc
