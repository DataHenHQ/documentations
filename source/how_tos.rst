*******
How-Tos
*******

The following are a list of commonly occuring DataHen use cases and how to do them. It is useful to skim through all the how-tos so that in the future, if you come across any similar need, you can refer back to this section.


Setting scraper’s mode
======================

The scraper support 2 different modes: **Single job** and **Multiple jobs**, being **Single job** mode the default.

Single job mode
---------------

On "Single job" mode, an scraper can only have 1 job on either `active` or `pause` status, so you will be required to either wait your job to finish or to manually cancel it before being able to start a new job. perfect for scraper development and most user cases.

Use the following command to explicitly change a scraper to "Single job" mode:

.. code-block:: bash

   $ hen scraper create <scraper_name> --no-multiple-jobs
   $ hen scraper update <scraper_name> --no-multiple-jobs

Multiple jobs mode
------------------

On "Multple jobs" mode, a scraper is not restricted to how many jobs can be active at the same time, which makes it perfect for on demand scraping using input vars.

Use the following command to change a scraper to "Multiple jobs" mode:

.. code-block:: bash

   $ hen scraper create <scraper_name> --multiple-jobs
   $ hen scraper update <scraper_name> --multiple-jobs


Setting a scraper’s scheduler
=============================

You can schedule a scraper’s scheduler in as granular detail as you want. However, we only support granularity down to the Minute.
We currently support the CRON syntax for scheduling.
You can set the ‘schedule’ field or the ‘timezone’ to specify the timezone when the job will be started. If timezone is not specified, it will default to “UTC”. Timezone values are IANA format. Please see the list of available timezones.

By default, a scraper's scheduler will cancel any existing job on active or paused status before it starts a new job.

.. code-block:: bash

   $ hen scraper create <scraper_name> <git_repo> --schedule "0 1 * * * *" --timezone "America/Toronto"
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

Prevent scraper's scheduler from cancel an existing job
-------------------------------------------------------

Scraper's scheduler "cancel current job" behavior can be disabled, so it doesn't start a new job if there is an already existing job that is active or paused.
Use the following command to disable scraper's scheduler "cancel current job" feature:

.. code-block:: bash

   $ hen scraper create <scraper_name> <git_repo> --no-cancel-current-job
   $ hen scraper update <scraper_name> --no-cancel-current-job

This feature can be enabled back at any time by using the follosing command:

.. code-block:: bash

   $ hen scraper create <scraper_name> <git_repo> --cancel-current-job
   $ hen scraper update <scraper_name> --cancel-current-job


Enabling global page cache
==========================

By default, every fetch is downloaded directly from the web, however you can enable your account's global page cache to save every successful fetch and be able to reuse it.

This feature is quite useful to avoid duplicated pages fetched when scraping the same website accross multiple scrapers or when using multiple jobs. It is also quite useful during developoment to debug your scraper's code or replicating a whole scrape as many times as you need. These are just a few examples from many use cases on which having cache is really useful and can significally speed up your development and scrapers.

You can use this command to enable global page cache on a scraper:

.. code-block:: bash

   $ hen scraper update <scraper_name> --enable-global-cache     # enable cache
   $ hen scraper update <scraper_name> --no-enable-global-cache  # disable cache

Or you can use this command to enable global page cache on a job:

.. code-block:: bash

   $ hen scraper job update <scraper_name> -j <job_id> --enable-global-cache     # enable cache
   $ hen scraper job update <scraper_name> -j <job_id> --no-enable-global-cache  # disable cache

Or you can enable global page cache on a single job page when enqueuing it, like this:

.. code-block:: ruby

   pages << {
      url: "http://test.com",
      enable_global_cache: true
   }



Changing a Scraper’s or a Job’s or a JobPage's Proxy Type
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

To change a proxy of an existing job, first pause the job, then change the proxy_type, and finally resume the job:

.. code-block:: bash

   $ hen scraper job pause <scraper_name>
   $ hen scraper job update <scraper_name> --proxy-type sticky1
   $ hen scraper job resume <scraper_name>

While setting proxy_type can be done to the Scraper or the Job, you can choose to override a particular JobPage's proxy_type as well.

Simply add the field `proxy_type` while enqueueing a page:


.. code-block:: ruby

   pages << {
      url: "http://test.com",
      proxy_type: "sticky1" 
   }



Changing a Scraper’s or a Job’s Profiles
========================================

If your scraper job needs more resources such as CPU or Memory, there is a way to add this via resource profiles.
This is especially useful if your particular scraping job requires additional power, such as parsing large pages, processing images, or if you require ability to store large number of pages or outputs at the job.

We support many types of profiles to use:


+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| Profiles               | Description                                                                                                                             |
+========================+=========================================================================================================================================+
| core                   | Internal configuration settings. This is the default.                                                                                   |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
|                        |                                                                                                                                         |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| account_xs             | Extra small account size, ideal for accounts with few small size scrapers. This is the default.                                         |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| account_s              | Small account size, ideal for accounts with some small size scrapers.                                                                   |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| account_m              | Medium account size, ideal for accounts with some mid size scrapers.                                                                    |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| account_l              | Large account size, ideal for accounts with lots of mid size scrapers.                                                                  |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| account_xl             | Extra large account size, ideal for accounts with lots of large size scrapers.                                                          |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
|                        |                                                                                                                                         |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| account_sp             | Small high processing account size, ideal for accounts with high processing usage.                                                      |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| account_mp             | Medium high processing account size, ideal for accounts with high processing usage.                                                     |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| account_lp             | Large high processing account size, ideal for accounts with high processing usage.                                                      |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
|                        |                                                                                                                                         |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| job_xs                 | Extra small job size, ideal for jobs with few pages. This is the default.                                                               |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| job_s                  | Small job size, ideal for jobs with a few thousand pages and outputs.                                                                   |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| job_m                  | Medium job size, ideal for jobs with a few hundred thousands pages and outputs.                                                         |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| job_l                  | Large job size, ideal for jobs with 1M pages and outputs.                                                                               |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| job_xl                 | Extra large job size, ideal for jobs with millions pages and outputs.                                                                   |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
|                        |                                                                                                                                         |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| job_sp                 | Small high processing job size, ideal for jobs that heavily relies on find_outputs.                                                     |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| job_mp                 | Medium high processing job size, ideal for jobs that heavily relies on find_outputs.                                                    |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| job_lp                 | Large high processing job size, ideal for jobs that heavily relies on find_outputs.                                                     |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
|                        |                                                                                                                                         |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| browser_xs             | Extra small browser fetch size, ideal for browser fetch that uses heavy driver code. This is the default.                               |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| browser_s              | Small browser fetch size, ideal for browser fetch that uses heavier driver code.                                                        |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| browser_m              | Medium browser fetch size, ideal for browser fetch that uses heavier driver code.                                                       |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| browser_l              | Large browser fetch size, ideal for browser fetch that uses heavier driver code.                                                        |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| browser_xl             | Extra large browser fetch size, ideal for browser fetch that uses heavier driver code.                                                  |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| fetcher_xs             | Extra small standard fetch size, ideal for most websites and small files. This is the default.                                          |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| fetcher_s              | Small standard fetch size, ideal for some heavy functional webpage and average image file size.                                         |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| fetcher_m              | Medium standard fetch size, ideal for medium heavy functional webpage and average image file size.                                      |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| fetcher_l              | Large standard fetch size, ideal for large heavy functional webpage and average image file size.                                        |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| fetcher_xl             | Extra large standard fetch size, ideal for extra large heavy functional webpage and average image file size.                            |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| fetcher_xxl            | Extra extra large standard fetch size, ideal for largest heavy functional webpage and above average image file size.                    |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| parser_xs              | Extra small parser worker, ideal for simple scrapers that parses average webpage size. This is the default.                             |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| parser_s               | Small parser worker, ideal for scrapers that parses a little above average webpage size.                                                |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| parser_m               | Medium parser worker, ideal for scrapers that parses above average webpage size.                                                        |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| parser_l               | Large parser worker, ideal for file downloads and heavy webpage HTML parsing.                                                           |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| parser_xl              | Extra large parser worker, ideal for file downloads and heavier webpage HTML parsing.                                                   |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| parser_gemfile_xs      | Extra small parser worker, ideal for simple scrapers that have gemfile.                                                                 |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| parser_gemfile_s       | Small parser worker, ideal for scrapers that have gemfile that need a little more resorces.                                             |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| parser_gemfile_m       | Medium parser worker, ideal for scrapers that have gemfile that need more resorces.                                                     |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| parser_gemfile_l       | Large parser worker, ideal for scrapers that have gemfile that need larger resorces.                                                    |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| parser_gemfile_xl      | Extra large parser worker, ideal for scrapers that have gemfile that need a extra large resorces.                                       |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| parser_libs_xs         | Extra small parser worker, ideal for simple scrapers that have heavy seeder or finisher.                                                |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| parser_libs_s          | Small parser worker, ideal for scrapers that have heavy seeder or finisher that need a little more resorces.                            |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| parser_libs_m          | Medium parser worker, ideal for scrapers that have heavy seeder or finisher that need more resorces.                                    |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| parser_libs_l          | Large parser worker, ideal for scrapers that have heavy seeder or finisher that need larger resorces.                                   |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| parser_libs_xl         | Extra large parser worker, ideal for scrapers that have heavy seeder or finisher that need a extra large resorces.                      |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| search_xs              | Extra small search, ideal for simple scrapers that use find_outputs.                                                                    |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| search_s               | Small search, ideal for small scrapers that use find_outputs.                                                                           |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| search_m               | Medium search, ideal for medium scrapers that use find_outputs.                                                                         |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| search_l               | Large parser worker, ideal for large scrapers that use find_outputs.                                                                    |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| search_xl              | Extra large parser worker, ideal for extra large scrapers that use find_outputs.                                                        |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
|                        |                                                                                                                                         |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| export_xs              | Extra small export, ideal for all normal scrapers.                                                                                      |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| export_s               | Small export, ideal for scrapers that have more data on them.                                                                           |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| export_m               | Medium export, ideal for scrapers that have more data on them.                                                                          |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| export_l               | Large export, ideal for large scrapers that have more data than normal.                                                                 |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| export_xl              | Extra large export, ideal for extra large scrapers that contains massive data.                                                          |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+

.. code-block:: bash

   $ hen scraper update <scraper_name> --profile "job_l,fetcher_m,parser_l"

Keep in mind that the above will only take effect when a new scrape job is created.

To change a profile of an existing job, first pause the job, then change the profile, and finally resume the job:

.. code-block:: bash

   $ hen scraper job pause <scraper_name>
   $ hen scraper job update <scraper_name> --profile "job_s,fetcher_m"
   $ hen scraper job resume <scraper_name>

Account deploy key
===============================

To check the account deploy key for advanced purposes.

.. code-block:: bash

   $ hen account deploy_key show

To recreate account deploy key if expired or need change.

.. code-block:: bash

   $ hen account deploy_key recreate

Setting a specific ruby version
===============================

By default our ruby version that we use is 2.6.5, however if you want to specify a different ruby version you can do so by creating a .ruby-version file on the root of your project directory.

NOTE: we currently only allow the following ruby versions:

* 2.6.5
* 2.7.2
* 3.0.1
* 3.0.7
* 3.1.5
* 3.2.4
* 3.3.1
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

Changing a Scraper’s Standard workers
==========================================

The more workers you use on your scraper, the faster your scraper will be, however, keep in mind that there are other things like blocks, target server load, etc., that could speed down your scraper even with the worker increase.

Standard workers are diveded into: parser worker and fetcher worker.
Use the parser worker to parse your downloaded content and increase it's speed by increasing the worker count.
Use the fetcher worker to increase your download speed by increasing the worker count.

You can use the command line to change a scraper’s fetcher worker count:

.. code-block:: bash

   $ hen scraper update <scraper_name> --fetchers N
   
You can use the command line to change a scraper’s parser worker count:
   
.. code-block:: bash

   $ hen scraper update <scraper_name> --parsers N

NOTE: Keep in mind that this will only take effect when a new scrape job is created if you set this up at scraper level.

Changing a Scraper’s Browser worker count
=========================================

The more workers you use on your scraper, the faster your scraper will be.
You can use the command line to change a scraper’s worker count:

.. code-block:: bash

   $ hen scraper update <scraper_name> --browsers N

NOTE: Keep in mind that this will only take effect when a new scrape job is created if you set this up at scraper level.

Changing an existing scrape job’s worker count
==============================================

You can use the command line to change a scraper job’s worker count:

.. code-block:: bash

   $ hen scraper job update <scraper_name> --fetchers N --parsers N --browsers N

This will only take effect if you pause, update and resume the scrape job again:

.. code-block:: bash

   $ hen scraper job pause <scraper_name> # pause first
   $ hen scraper job update <scraper_name> --fetchers N --parsers N --browsers N #update workers
   $ hen scraper job resume <scraper_name> # then resume
   
NOTE: Once you update the job changing the workers, job core will stop so it may take a while to start again since need to do a backup first internally.

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

   $ hen scraper page add <scraper_name> '{"url": "http://test.com", "fetch_type": "browser"}'
   $ cat /path/to/my_page_file.json | hen scraper page add <scraper_name>
   $ hen scraper page add <scraper_name> < /path/to/my_page_file.json

You can enqueue a page like so in your script. The following will enqueue a full browser (non-headless):

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     fetch_type: "fullbrowser" # This will enqueue headless browser
   }

Or use the command line:

.. code-block:: bash

   $ hen scraper page add <scraper_name> '{"url": "http://test.com", "fetch_type": "fullbrowser"}'
   $ cat /path/to/my_page_file.json | hen scraper page add <scraper_name>
   $ hen scraper page add <scraper_name> < /path/to/my_page_file.json

**Important**

`Host` header is not supported on browser fetch and should be removed from headers. Cookies should be set at page's `cookie` attribute instead of `Cookie` header.

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

   $ hen scraper page add <scraper_name> '{"url": "http://test.com", "priority": "N"}'
   $ cat /path/to/my_page_file.json | hen scraper page add <scraper_name>
   $ hen scraper page add <scraper_name> < /path/to/my_page_file.json
   $ hen scraper page update <job> <gid> --priority N

Setting a user-agent-type of a Job Page
=======================================

You can enqueue a page like so in your script:

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     ua_type: "desktop" # defaults to desktop, other available values are `mobile`, `none`.
   }

Or use the command line:

.. code-block:: bash

   $ hen scraper page add <scraper_name> '{"url": "http://test.com", "ua_type": "desktop"}'
   $ cat /path/to/my_page_file.json | hen scraper page add <scraper_name>
   $ hen scraper page add <scraper_name> < /path/to/my_page_file.json

Disable user agent override
---------------------------

You can disable the user agent override by setting it to `none` value, quite useful when dealing with JS user agent checks on browser fetch.

You can enqueue a page disabling it like this:

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     ua_type: "none" # disable user agent override
   }

Or use the command line:

.. code-block:: bash

   $ hen scraper page add <scraper_name> '{"url": "http://test.com", "ua_type": "none"}'
   $ cat /path/to/my_page_file.json | hen scraper page add <scraper_name>
   $ hen scraper page add <scraper_name> < /path/to/my_page_file.json

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

   $ hen scraper page add <scraper_name> '{"url": "http://test.com", "method": "POST"}'
   $ cat /path/to/my_page_file.json | hen scraper page add <scraper_name>
   $ hen scraper page add <scraper_name> < /path/to/my_page_file.json

Setting the TLS version on a Job Page (only works on standard fetch)
========================================

You can set up the TLS version by setting the min version and max version and to use a fixed version will be the same on the min and max version to use that specific version, currently we support the following versions:

.. code-block:: bash

   TLS v1.0 -> 10
   TLS v1.1 -> 11
   TLS v1.2 -> 12
   TLS v1.3 -> 13

An example of use would be like this:

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     tls: { 
	  	"min_version": 13,
	  	"max_version": 13
	  }
   }


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

   $ hen scraper page add <scraper_name> '{"url": "http://test.com", "headers": {"Cookie": "name=value; name2=value2; name3=value3"}}'
   $ cat /path/to/my_page_file.json | hen scraper page add <scraper_name>
   $ hen scraper page add <scraper_name> < /path/to/my_page_file.json

Disable default request headers
-------------------------------

Datahen adds a the following default headers to every request to help standard and browser fetch success:

.. code-block:: ruby

   Accept-Language: en, en-US;q=0.9, en-CA;q=0.8, en-GB;q=0.7, *;q=0.1
   Accept-Charset: utf-8, *;q=0.1
   Accept-Encoding: gzip
   
All of them are overridable by the user except by `Accept-Encoding: gzip` which is forced by our fetcher to speed up fetching by compressing the response.

However, there are some scenarios (specially on browser fetch and API requests) on which having these default headers leads to failed fetches or weird behavior.

To fix this, you can prevent Datahen from adding these default headers (including the forced ones) when enqueuing your page like this:

.. code-block:: ruby

   pages << {
     url: "https://test.com",
     no_default_headers: true
   }


Use custom headers
-------------------------------

This is useful when you have a case sensitive scenario where the request headers need to be in a specific way, without this will send a Capitalized header like this `Appversion` and lets say you need it to be `appVersion` so if that is the case you will need to set the flag to true and send the headers as you need, but please be careful because using this will send the headers as you send them if the header is case sensitive and is wrong the request may fail:

.. code-block:: ruby

   pages << {
     url: "https://test.com",
     custom_headers: true,
     headers: {
     	  "Accept": "*/*",
   	  "Content-Type": "application/json; charset=UTF-8",
   	  "User-Agent": "Dalvik/2.1.0 (Linux; U; Android 9; Pixel 3 Build/PI)",
   	  "appVersion": "1.1.10",
   	  "locale": "en",
   	  "region": "HK",
   	  "timeZone": "GMT+08:00",
   	  "uuid": "000000000000027EE6C1719344AECA10CC3646524A5A2A170026393340307752"
     }
   }


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

   $ hen scraper page add <scraper_name> '{"url": "http://test.com", "body": "your request body here"}'
   $ cat /path/to/my_page_file.json | hen scraper page add <scraper_name>
   $ hen scraper page add <scraper_name> < /path/to/my_page_file.json

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

   $ hen scraper page add <scraper_name> '{"url": "http://test.com", "page_type": "page_type_here"}'
   $ cat /path/to/my_page_file.json | hen scraper page add <scraper_name>
   $ hen scraper page add <scraper_name> < /path/to/my_page_file.json

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

   $ hen scraper page add <scraper_name> '{"url": "http://test.com", "cookie": "name=value; name2=value2; name3=value3"}'
   $ cat /path/to/my_page_file.json | hen scraper page add <scraper_name>
   $ hen scraper page add <scraper_name> < /path/to/my_page_file.json

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

   $ hen scraper page add <scraper_name> '{"url": "http://test.com", "freshness": "2018-12-12T13:59:29.91741Z", "force_fetch": true}'
   $ cat /path/to/my_page_file.json | hen scraper page add <scraper_name>
   $ hen scraper page add <scraper_name> < /path/to/my_page_file.json

Handling JavaScript
===================

To do javascript rendering, please use the Browser Fetcher.
First you need to add a browser worker onto your scraper:

.. code-block:: bash

   $ hen scraper update <scraper_name> --browsers 1

Next, for every page that you add, you need to specify the correct fetch_type:

.. code-block:: bash

   $ hen scraper page add <scraper_name> '{"url": "http://test.com", "fetch_type": "browser"}'
   $ cat /path/to/my_page_file.json | hen scraper page add <scraper_name>
   $ hen scraper page add <scraper_name> < /path/to/my_page_file.json

Or in the script, by doing the following:

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     fetch_type: "browser"
   }

Max page size
===================

This is a value that sets the max page size validation, default value is 0 that means no limit. This is value represent how many bytes are allowed when the fetch is done and this is validated against content length. This can be set at 3 levels: scraper, job and page.

.. code-block:: bash

   $ hen scraper update <scraper_name> --max-page-size 12345
   $ hen scraper job update <scraper_name> --max-page-size 12345
   $ hen scraper page update <scraper_name> <gid> --max-size 12345

Or in the script per page, by doing the following:

.. code-block:: ruby

   pages << {
     url: "http://test.com",
     max_size: 12345
   }


Browser display
===============

We support display size configuration within Browser Fetcher having 1366x768 as default size. This feature is quite useful when interacting with responsive websites and taking screenshots. Only `browser` and `fullbrowser` fetch types support this feature.

IMPORTANT: For performance purposes, Browser Fetcher ignores images downloaded on the page by default. To enable it, see :ref:`Enabling browser images`.

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

For this browser fetch we have enable Adblocker by default so that can reduce time and remove some things from webpage that are probably not necessary, this feature can be disabled.

We fully support JS puppeteer's `page object <https://pptr.dev/#?product=Puppeteer&version=v5.2.1&show=api-class-page>`_ and provide a predefined `sleep(miliseconds)` async function to allow easy browser interaction and actions.  

IMPORTANT: For performance purposes, Browser Fetcher ignores images downloaded on the page by default. To enable it, see :ref:`Enabling browser images`.


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

Enqueue same page twice with different code
-------------------------------------------

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

Generate GID without Enqueue a page
-----------------------------------

Sometimes you will need to know what GID will be generated given a page definition without enqueuing the page. This example shows you how to get the GID without enqueuing the page by providing the page definition as a JSON string:

.. code-block:: bash

   $ hen scraper page getgid <scraper_name> '{"url": "https://example.com", "page_type": "default"}'

Or you can also send a JSON file with the page definition by using `cat`, for example:

.. code-block:: bash

   $ cat /path/to/my_page_file.json | hen scraper page getgid <scraper_name>

Or by providing the file directly to `stdin`:

.. code-block:: bash

   $ hen scraper page getgid <scraper_name> < /path/to/my_page_file.json

Intercept request
-----------------

Puppeteer provides an event to intercept requests (`page.on("request", ...)`) with the capability to override the request, however, it has the limitation that can't be used multiple times. The problem is that our internal puppeteer function requires the use of this event to override headers along some other features making it unavailble to the user, since if it were to be used, then it will break it's functionality.

To overcome this limitation, we have created a custom function called `intercept` that allows the user to intercept the requests as many times as you need while providing the ability to override everything on the request, being the first interception our own internal overrides.

`intercept` function is commonly used inside `driver.pre_code` (see :ref:`Executing puppeteer code before fetch is done`) to override sub requests and block trackers.

This example shows you how to override all PNG, GIF and JPEG image urls with `https://www.google.com`:

.. code-block:: ruby

   pages << {
     "url": "https://www.datahen.com",
     "page_type": "homepage",
     "fetch_type": "browser",
     "driver": {
       "pre_code": '
         intercept((request, overrides) => {
           if (/.(png|gif|jpe?g)/i.test(request.url)) {
              overrides["url"] = "https://www.google.com";
           }
           return true;
         });
       ' 
     }
   }


Executing puppeteer code before fetch is done
-----------------------

There are scenarios on which you will need to execute code before the page is fetched like disabling Javascript or intercepting a sub request (see :ref:`Intercept request`), so we provide `driver.pre_code` attribute to do this and just like on 'driver.code', `driver.pre_code` uses puppeteer functions.

This example shows you how to override all request's urls:

.. code-block:: ruby

   pages << {
     "url": "https://www.datahen.com",
     "page_type": "homepage",
     "fetch_type": "browser",
     "driver": {
       "pre_code": '
         intercept((request, overrides) => {
           overrides["url"] = "https://www.google.com";
           return true;
         });
       ' 
     }
   }
   
This example shows you how to execute goto url page on queue and refresh it using this function refreshQueuePage():

.. code-block:: ruby

   pages << {
     "url": "https://www.datahen.com",
     "page_type": "homepage",
     "fetch_type": "browser",
     "driver": {
       "pre_code": '
         intercept((request, overrides) => {
           overrides["url"] = "https://www.google.com";
           return true;
         });
         await page.goto("https://www.datahen.com/"); 
         await refreshQueuePage();
       '
     }
   }

Sharing data between `pre_code` and `code`
------------------------------------------

You might find scenarios on which you need to share data between the JS code set on `driver.pre_code` and `driver.code`, so Datahen provides a global hash variable called `codeVars` that can be used to share values between them. It is quite useful when extracting data using `intercept` function (see ).

This example shows you how to use `codeVars` to share data between `driver.pre_code` and `driver.code` scripts:

.. code-block:: ruby

   pages << {
     url: 'https://www.datahen.com',
     fetch_type: 'browser',
     driver: {
       pre_code: "codeVars['foo'] = 'bar'",  # save vars
       code: "
         await page.goto('about:blank')

         await page.evaluate((text) => {
           element = document.createElement('code');
           element.innerHTML = text;

           /* This will append 'bar' to the page HTML  */
           document.body.appendChild(element)
         }, codeVars['foo']);
       "
     }
   }

Enabling browser images
-----------------------

For performance purposes, Browser Fetcher ignores all images downloaded on that page by default. 

To enable images, set `driver.enable_images` to `true`. This example shows you how to do so:

.. code-block:: ruby

   pages << {
     "url": "https://www.datahen.com",
     "page_type": "homepage",
     "fetch_type": "browser",
     "driver": {
       "enable_images": true 
     }
   }

Disabling browser Adblocker feature
-----------------------

For performance purposes, Browser Fetcher uses adblocker on that page by default. 

To disable adblocker, set `driver.disable_adblocker` to `true`. This example shows you how to do so:

.. code-block:: ruby

   pages << {
     "url": "https://www.datahen.com",
     "page_type": "homepage",
     "fetch_type": "browser",
     "driver": {
       "disable_adblocker": true 
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


Distinction of pages() vs newPage()
-----------------------------------

The `pages()` method gets a list of all open pages inside this browser instance. If there ar multiple browser contexts, this returns all pages in all browser contexts.
This is very useful when you are handling several tabs or websites that uses popups and you need to do some stuff with a specific page. For example, whenever a target
website login process opens in a popup window or a new tab and you need to access it in order to login and then the original page reloads.

Code sample:

.. code-block:: ruby

   const myPages = await pages();

The `newPage()` method creates a new page in the default browser context. This is very useful when you need to open a new tab page that does some execution and then
return to the main one. For example, when you need to login within a different domain or you need to visit several pages in parallel in order to perform a certain flow
and extract the data from it.

Code sample: 

.. code-block:: ruby

   const myNewPage = await newPage();


Taking screenshots
==================

We support browser screenshots within Browser Fetcher by enabling `screenshot.take_screenshot` attirbute. It is important to note that taking a screenshot will replace the page `content` with the screenshot binary contents. Only `browser` and `fullbrowser` fetch types support this feature.

IMPORTANT: For performance purposes, Browser Fetcher ignores images downloaded on the page by default. To enable it, see :ref:`Enabling browser images`.

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

We currently support the ability to query a scraper outputs using query selectors that are similar to, and heavily inspired by MongoDB. 

Querying basics
---------------

Querying of outputs can be done via the CLI or the actual scraper code.

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


Querying from another Job or Scraper
------------------------------------

To find output from another job, do the following:

.. code-block:: ruby

   records = find_outputs('foo_collection', {"_id":"123"}, 1, 500, job_id: 1234}

To find output from another scraper, do the following:

.. code-block:: ruby

   records = find_outputs('foo_collection', {"_id":"123"}, 1, 500, scraper_name:'my_scraper'}


Paginations and Performance Considerations
---------------

Due to performance considerations especially when outputs has millions of records, we only support unlimited number of pages when `find_outputs` are used without any query.

If a query is used, the outputs are limited to only 3 pages maximum.

The following command works because, it doesn't use any query.

.. code-block:: ruby

   page = 1234 # you can specify any page number here
   query = {} # empty query here
   records = find_outputs('foo_collection', query, page}

If you use a query, then you can only return the first three pages, like so:

.. code-block:: ruby

   page = 3 # Maximum is page 3
   query = {"bar": {"$ne": "baz"}} # there is a query here
   records = find_outputs('foo_collection', query, page}

So, how do you get all millions of records of records in the collection?

Answer: You would use the record's `_id` with the `$gt` greater than operator.

Consider the following Ruby script where you loop through the `find_outputs` and using `$gt` with record `_id`:

.. code-block:: ruby

   per_page = 500 # you can get up to 500 records per request
   page_counter = 0 # optional: count pages to track the page number
   last_id = '' # last processed output "_id", it is empty by default to get any output
   
   # start a loop
   while true
      # output search query
      query = {
        "bar": {"$eq": "baz"}, # your custom query goes here

        '_id' => {'$gt' => last_id},  # get all outputs with "_id" greater than "last_id"
        '$orderby' => [{'_id' => 1}]  # order by output "_id"
      }
   
      # get all output records matching the search query
      # notice that page number is always 1 since we handle the output batch using "last_id"
      records = find_outputs('foo_collection', query, 1, per_page)
      
      # exit loop when there are no more output records to process
      break if records.nil? || records.count < 1
      
      # optional: keep track of the current page
      page_counter += 1
      
      # iterate the output records
      records.each do |record|
        # save the lastest processed output "_id" so we can use it to request the next page
        last_id = record['_id']

        # process the output record here ...
      end
   end

The previous code will efficiently iterate all outputs given a query on 500 output batches with low RAM usage. We can remove all optional code and comments to have a really small snippet:


.. code-block:: ruby

   per_page = 500
   last_id = ''
   
   while true
      query = {
        "bar": {"$eq": "baz"}, # your custom query goes here

        '_id' => {'$gt' => last_id},
        '$orderby' => [{'_id' => 1}]
      }
      records = find_outputs('foo_collection', query, 1, per_page)
      break if records.nil? || records.count < 1
      
      records.each do |record|
        last_id = record['_id']

        # process the output record here ...
      end
   end

Logical operations
---------------------

We support the following logical operations in the queries:

+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| Operator               | Description                                                                                                                             |
+========================+=========================================================================================================================================+
| $and                   | Joins query clauses with a logical AND returns all records that match the conditions of both clauses.                                 |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| $or                    | Joins query clauses with a logical OR returns all records that match the conditions of either clause.                                 |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+

Example queries:

.. code-block:: javascript

   {
      "$or": [ 
         {"foo": "foo1"},
         {"bar": "bar1"}
      ]
   }

.. code-block:: javascript
   
   // this operation compares the same field on two different values
   {
      "$and": [
         {"foo": {"$ne": null}},
         {"foo": {"$ne": "bar"}}
      ]
   }




Comparison operations
---------------------

We support the following comparison operations in the queries:

+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| Operator               | Description                                                                                                                             |
+========================+=========================================================================================================================================+
| $eq                    | Matches values that are equal to a specified value.                                                                                     |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| $ne                    | Matches all values that are not equal to a specified value.                                                                             |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| $gt                    | Matches values that are greater than a specified value.                                                                                 |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| $gte                   | Matches values that are greater than or equal to a specified value.                                                                     |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| $lt                    | Matches values that are less than a specified value.                                                                                    |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| $lte                   | Matches values that are less than or equal to a specified value.                                                                        |
+------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+

Example queries:

.. code-block:: javascript

   {
     "_id": {"$gt": "abcd123"}
   }

Ordering Outputs by field(s)
------------------------------------

We support the `$orderby` operations to sort by fields of your choice. 
Use the value of `1` for ascending order, and `-1` for descending order.

Example query:

.. code-block:: javascript

   {
      "_id": {"$gt": "abcd123"},
      "$orderby":[{"foo": 1}, {"bar": -1}]
   }


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

**Important**

A Variable can only contain maximum of value of 130,000 characters. If you plan on sending a large texts to the job, consider saving the the text in a file, and storing it in external storage like Amazon S3. And you can then set the URL to that file on the variable. 


Setting Environment Variables and Secrets on your account.
----------------------------------------------------------

You can set any environment variables and secrets in your account that you can then use in any of your scrapers or jobs.


This `example scraper <https://github.com/DataHenHQ/ebay-scraper/tree/env_vars>`_ shows usage of environment variables.

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

This `example scraper <https://github.com/DataHenHQ/ebay-scraper/tree/input_vars>`_ shows usage of input variables.

There are three steps that you need to do in order to use input variables and secrets:

1. Start a new job with input vars.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
To start a new job and set it's input vars right away using command line:

.. code-block:: bash

   $ hen scraper start <scraper_name> --vars '[{"name":"foo", "value":"bar", "secret":false}]'

To start a new job and set it's input vars right away using `datahen` ruby gem:

.. code-block:: ruby

   client = Datahen::Client::ScraperJob.new({})
   client.create("my_scraper_name", {
     vars: [
       {
         "name":"foo",
         "value":"bar",
         "secret":false
       }
     ]
   })

2. Set the input variable or secrets on your scraper.
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


3. Change your config.yaml to use the variables or secrets.
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


4. Access the input variables and secrets in your script.
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

Using browser fetcher custom images
-----------------------------------

These are special docker images built by us to provide different browser versions and driver engines compatible with our system.  Please
contact us for more info regarding available images.

Once you have the right image for you, you can use that custom image by modifying your config.yaml file and include the following line:

.. code-block:: bash

   browser_fetcher_image: <url-to-your-docker-image>

How to use shared code libraries from other Git repositories using Git Submodule
================================================================================

Sometimes you want to have a scraper that has a shared list of libraries that are used by other scrapers in other Git repositories.
Luckily DataHen supports Git Submodules, which enables this scenario.

You simply just deploy a scraper as usual, and DataHen will take care of initating and checking out the submodules recursively.

This is `the documentation on Git Submodules <https://git-scm.com/book/en/v2/Git-Tools-Submodules>`_ that shows the usage in depth.

This `example scraper <https://github.com/DataHenHQ/ebay-scraper/tree/submodule>`_ shows usage of git submodules.

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

`http3: true`
-------------
This change the standard fetch to HTTP/3, which not only makes fetch faster on websites that support it, but also helps to bypass some anti-scrape tech that usually needs this protocol by using quic requests.

**Example:**

.. code-block:: ruby

   pages << {
     'url' => 'https://example.com'
   }
   # => page fetching will use HTTP/1

   pages << {
     'url' => 'https://example.com',
     'http3' => true
   }
   # => page fetching will use HTTP/3


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




Exclude pages from a job
------------------------

There will be times on which you need to exclude a page from the whole job, like a page created just
to test a hotfix or a refetch failed page.

Here is where `limbo` status comes in. Any page sent to `limbo` will be kept exactly as it was at the
moment it is sent there and will be completely ignored by your job scraping flow.

To send a specific page to `limbo` status from your parser scripts, use the following method:

.. code-block:: ruby

   limbo page['gid']           # send current page to limbo
   limbo 'example.com-123abc'  # send a page with a specific GID to limbo,
                               #  replace 'example.com-123abc' with your page's GID

To send a specific page to `limbo` status using CLI, use the following command:

.. code-block:: bash

   $ hen scraper page limbo <scraper_name> --gid <gid>

Check the `help` command to find other ways to use this command:

.. code-block:: bash

   $ hen scraper page help limbo
   Usage:
     hen scraper page limbo <scraper_name>

   Options:
     g, [--gid=GID]         # Move a specific GID to limbo
         [--status=STATUS]  # Move pages with a specific status to limbo.
     j, [--job=N]           # Set a specific job ID

   Description:
     Move pages in a scraper's current job to limbo. You need to specify either a --gid or --status.
