********************
Datahen
********************

Fetch is a platform where you can scrape data from the internet quickly and easily without incurring significant costs.
We do this by allowing you to scrape data from the shared-cache of contents that Fetch has collectively downloaded from the Internet for other users to scrape.
Your scraping from the cached-content does not necessarily prevent you from getting the freshest content. In fact, you can specify how fresh the contents you want to scrape are by specifying your freshness-type and also specifying that the content should be “force fetched” or not.

Keep in mind that any page that Fetch downloads for your scraper, will be stored on the shared-cache, and will be available to other Fetch users to use for their own scrapers. This reuse of cached pages allows every Fetch users to collectively save cost and save time as they scrape data from the Internet.
Fetch is like curl, where you have lower level control of HTTP request, such as request method, headers, body, and more.

Getting Started
===============

In this getting started section, we will get you started with installing the necessary requirements, and then deploying and running an existing scraper into AnswerEngine Fetch. Currently we support ruby 2.4.4 and 2.5.3.

Install Datahen Command Line Interface using rubygems
-----------------------------------------------------------

.. code-block:: bash

   $ gem install datahen
   Successfully installed datahen-0.2.3
   Parsing documentation for datahen-0.2.3
   Done installing documentation for datahen after 0 seconds
   1 gem installed

Get your access token
---------------------

You can create “account_admin” or “basic” token.
The difference between the two is an “account_admin” token can create other access tokens, whereas basic account could not.

Set environment variable of your access token
---------------------------------------------

.. code-block:: bash

   $ export DATAHEN_TOKEN=<your_token_Here>

Now you’re ready to go.

Create the scraper
------------------

In this step we will create a scraper on Datahen, by specifying the scraper name, and the git repository where the scraper script comes from:

.. code-block:: bash

   $ datahen scraper create walmart-movies git@git.datahen.com:scrapers/walmart-movies.git --workers 1 --browsers 1
   {
    "name": "walmart-movies",
    "id": 54,
    "account_id": 1,
    "force_fetch": false,
    "freshness_type": "any",
    "created_at": "2019-03-12T10:28:22.037768Z",
    "git_repository": "git@git.datahen.com:scrapers/walmart-movies.git",
    "git_branch": "master",
    "deployed_git_repository": "git@git.datahen.com:scrapers/walmart-movies.git",
    "deployed_git_branch": "master",
    "deployed_commit_hash": "e7d77d7622e7b71c32300eafd2d44a8429142fe3",
    "deployed_at": "2019-03-12T10:28:22.037768Z",
    "worker_count": 1,
    "config": {
     "parsers": [
      {
       "file": "./parsers/part.rb",
       "page_type": "part"
      }
     ],
     "seeder": {
      "file": "./seeder/seeder.rb"
     }
    }
   }

Let’s see if your scraper has been created.
Let’s look at the list of scrapers that you have now:

.. code-block:: bash

   $ datahen scraper list
   [
    {
     "name": "ebay",
     "id": 20,
     "account_id": 1,
     "force_fetch": false,
     "freshness_type": "any",
     "created_at": "2018-11-26T22:00:43.007755Z",
     "git_repository": "https://github.com/datahen/ebay-scraper.git",
     "git_branch": "master",
     "deployed_git_repository": "https://github.com/datahen/ebay-scraper.git",
     "deployed_git_branch": "master",
     "deployed_commit_hash": "7bd6091d97a17cf8ee769e00ac285123c41aaf4f",
     "deployed_at": "2018-11-28T06:13:56.571052Z",
     "worker_count": 1,
   ...

Or if you’d like to see your specific scraper, you can do:

.. code-block:: bash

   $ datahen scraper show walmart-movies
   {
    "name": "walmart-movies",
    "id": 18,
    "account_id": 1,
    "force_fetch": false,
    "freshness_type": "any",
    "created_at": "2019-03-12T10:28:22.037768Z",
    "git_repository": "git@git.datahen.com:scrapers/walmart-movies.git",
    "git_branch": "master",
    "deployed_git_repository": "git@git.datahen.com:scrapers/walmart-movies.git",
   ...

Now that we have created the scraper, we need to deploy.

Deploying the scraper
---------------------

Once we have created the scraper, let’s deploy it from the git repo that you have specified.

.. code-block:: bash

   $ datahen scraper deploy walmart-movies
   Deploying scraper. This may take a while...
   {
    "id": 135,
    "scraper_id": 18,
    "commit_hash": "e7d77d7622e7b71c32300eafd2d44a8429142fe3",
    "git_repository": "git@git.datahen.com:scrapers/walmart-movies.git",
    "git_branch": "master",
    "errors": null,
    "success": true,
    "created_at": "2019-03-12T10:48:22.037768Z",
    "config": {
     "parsers": [
      {
       "file": "./parsers/part.rb",
       "page_type": "part"
      }
     ],
     "seeder": {
      "file": "./seeder/seeder.rb"
     }
    }
   }

Let’s see if the list of deployments, if you’re curious to know your deployment history.

.. code-block:: bash

   $ datahen scraper deployment list walmart-movies
   [
    {
     "id": 135,
     "scraper_id": 18,
     "commit_hash": "e7d77d7622e7b71c32300eafd2d44a8429142fe3",
     "git_repository": "git@git.datahen.com:scrapers/walmart-movies.git",
     "git_branch": "master",
   ...

Run the scraper
---------------

Now that the scraper codes has been deployed, let’s run it.

.. code-block:: bash

   $ datahen scraper start walmart-movies
   Starting a scrape job...
   {
    "id": 135,
    "scraper_id": 18,
    "created_at": "2019-03-12T10:52:22.037768Z",
    "freshness": null,
    "force_fetch": false,
    "status": "active",
    "seeding_at": null,
    "seeding_failed_at": null,
    "seeded_at": null,
    "seeding_try_count": 0,
    "seeding_fail_count": 0,
    "seeding_error_count": 0,
    "worker_count": 1
   }

This will now then create a scraping job, which will start fetching pages for you, and parsing them into the outputs.

You can also see all jobs that was created on the scraper.

.. code-block:: bash

   $ datahen scraper job list walmart-movies
   [
    {
     "id": 135,
     "scraper_name": "walmart-movies",
     "scraper_id": 18,
     "created_at": "2019-03-12T10:48:22.037768Z",
   ...

To view the current job on the scraper.

.. code-block:: bash

   $ datahen scraper job show walmart-movies
   {
    "id": 135,
    "scraper_name": "walmart-movies",
    "scraper_id": 18,
    "created_at": "2019-03-12T10:48:22.037768Z",
   ...

Viewing the Job Stats
---------------------

While the job is running, let’s look how the job is doing by looking at the stats. You’ll first need to get the ID form the job list command above.

.. code-block:: bash

   $ datahen scraper stats walmart-movies
   {
    "job_id": 135,
    "pages": 822,
    "fetched_pages": 822,
    "to_fetch": 0,
    "fetching_failed": 0,
    "fetched_from_web": 0,
    "fetched_from_cache": 822,
    "parsed_pages": 0,
    "to_parse": 822,
    "parsing_failed": 0,
    "outputs": 0,
    "output_collections": 0,
    "workers": 1,
    "time_stamp": "2019-03-12T10:48:22.037768Z"
   }

Viewing the Job Pages
---------------------

Let’s see the pages that has been added by the seeder script into this job.

.. code-block:: bash

   $ datahen scraper page list walmart-movies
   [
    {
     "gid": "www.walmart.com-4aa9b6bd1f2717409c22d58c4870471e", # Global ID
     "job_id": 135,
     "page_type": "listings",
     "method": "GET",
     "url": "https://www.walmart.com/browse/movies-tv-shows/4096?facet=new_releases:Last+90+Days",
     "effective_url": "https://www.walmart.com/browse/movies-tv-shows/4096?facet=new_releases:Last+90+Days",
     "headers": "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36",
   ...

Viewing a Global Page Content
-----------------------------

You may be wondering what is a Global Page.
A Global Page acts like a shared-cache that Datahen fetches for all their users as they perform scraping. This shared-cache allows every users to collectively benefit from lower cost and higher performance of extracting data from the Internet.

Now that you’ve seen the pages that has been added into this job, let’s see the content of the page by copying and pasting a page’s GID(Global ID) into the following command.

.. code-block:: bash

   $ datahen globalpage content www.walmart.com-4aa9b6bd1f2717409c22d58c4870471e
   Preview content url: "https://fetch.datahen.com/public/global_pages/preview/HS2RNNi0uKe2YQ3tlU-cedGCWhRHgLcm5PWTwTVx0VLs5yjlOt6bE8qma7lzv6oCfUSYBNHu3IpXK70961lRhcqruPg5xa29OmuSJvolz_ONcVV2nmeMfJx8tSe_jRi8JW1qIfD7O8Rchf3XdO10pfjgICiV_FBczWPGYmg3rNLGcHMk5UGseJcl7maAGvN5bhvrwesscrODp_mni894gKz8a9v3GTFtjVGUgexS-dEu2DKTfe6SNb1ZKHj08SUCTM61P_Umg6XzF-bJBePMZuoX2b8nkXQ3mDw1-bdMJ-WPFUfQ01T5gtkoCBDuSFBg-T8YGETNEPNm0usglfWzsq4="

View the scraper output
-----------------------

Job Outputs are stored in collections. If none is specified, it will be stored in the “default” collection.
Let’s view the outputs of a scraper job by first seeing what collections the scraper outputs to:

.. code-block:: bash

   $ datahen scraper output collection walmart-movies
   [
    {
     "collection": "products",
     "count": 72
    }
   ]

In the result of the command line above, you will see the collection called “products.” Let’s look at the outputs inside the “products” collection:

.. code-block:: bash

   $ datahen scraper output list walmart-movies --collection products
   [
    {
     "_collection": "products",
     "_created_at": "2019-03-12T10:50:44.037768Z",
     "_gid": "www.walmart.com-a2232af59a8d52c356136f6674f532c5",
     "_id": "3de2e6b6e16749879f7e9bdd1ea3f0fc",
     "_job_id": 1341,
     "categories": [
      "Movies & TV Shows",
      "Movies",
      "Documentaries",
      "All Documentaries"
     ],
     "current_price": 21.89,
     "img_url": "https://i5.walmartimages.com/asr/5064efdd-9c84-4f17-a107-2669a34b54ff_1.474fdc2d2d1ea64e45def9c0c5afb4c0.jpeg",
     "original_price": null,
     "publisher": "Kino Lorber",
     "rating": null,
     "reviews_count": 0,
     "title": "International Sweethearts of Rhythm (DVD)",
     "walmart_number": "572439718"
    },
   ...

View the scraper logs
---------------------

If there is an error that occured it will be shown in the job log.
Let’s see what’s in the log.

.. code-block:: bash

   $ datahen scraper log walmart-movies

You can view the log of what happens.

Congratulations! You’ve created and ran your first scraper.

Let’s now cleanup from this Getting Started section by canceling that running job.

.. code-block:: bash

   $ datahen scraper job cancel walmart-movies
   {
    "id": 135,
    "scraper_name": "walmart-movies",
    "scraper_id": 18,
    "created_at": "2019-03-12T10:48:22.058468Z",
    "freshness": null,
    "force_fetch": false,
    "status": "cancelled",
    "seeding_at": "2019-03-12T10:49:42.035968Z",
    "seeding_failed_at": null,
    "seeded_at": "2019-03-12T10:50:23.057768Z",
    "seeding_try_count": 1,
    "seeding_fail_count": 0,
    "seeding_error_count": 0,
    "worker_count": 1
   }

You’re now done with the Getting Started section. Next steps are to read the high level concepts, and do the tutorials.
