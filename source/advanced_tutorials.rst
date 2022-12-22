******************
Advanced Tutorials
******************

How to write a QA script to ingest and parse outputs from multiple scrapers
===========================================================================

Step 1: Setup
-------------

In this tutorial we are going to go over the steps to do QA (Quality Assurance) on the output from your scrapers
so you can validate that the data you scrape is correct. We do this by creating another "QA" scraper that will ingest
the output from your initial scraper and perform QA on it using rules that you define.

For this example, we are going to be creating a QA scraper for the, "Simple Ebay scraper" that we previously
created in, "Coding Tutorials." Let's create an empty directory and name it 'ebay-scraper-qa'

.. code-block:: bash

   $ mkdir ebay-scraper-qa

Next let’s go into the directory and initialize it as a Git repository

.. code-block:: bash

   $ cd ebay-scraper-qa
   $ git init .
   Initialized empty Git repository in /Users/workspace/ebay-scraper-qa/.git/

Create a file called Gemfile inside the ebay-scraper-qa directory with the following line.
This will add the dh_easy-qa Gem, which is the Gem that handles the QA.

.. code-block:: bash

   gem 'dh_easy-qa'

Now run the following command in the ebay-scraper-qa directory to install this gem:

.. code-block:: bash

   bundle

Next let's make a yaml file that will contain the configuration for the QA. Create a file called dh_easy.yaml inside
the ebay-scraper-qa directory with the following contents:

.. code-block:: yaml

   qa:
     scrapers:
       ebay-scraper:
         - listings

This is the configuration we use to tell the QA what scrapers to use and which collections within each scraper to
perform QA on. In this example, we are going to be doing QA on our ebay-scraper scraper and, more specifically,
the listings collection. You can add as many scrapers and as many collections here as you want, but keep in mind that
the same QA validation rules will be applied to all of them. Basically, whatever scrapers and collections you add
should have similar output. If you have another scraper that has different output and different validation rules, you
will need to create a separate QA scraper.

Step 2: Seeding
---------------

Now, let’s create a seeder script. This is where we will iterate through the scrapers that we just defined and seed them
to be fetched by DataHen. In this example we will just seed one scraper.

Create the seeder directory:

.. code-block:: bash

   mkdir seeder

Next, let’s create a ruby script called seeder.rb in the seeder directory with the following content:

.. code-block:: ruby

   config_path = File.expand_path('dh_easy.yaml', Dir.pwd)
   config = YAML.load(File.open(config_path))['qa']
   config['scrapers'].each do |scraper_name, collections|
     if collections
       pages << {
         url: "https://fetchtest.datahen.com/?scraper=#{scraper_name}",
         method: "GET",
         page_type: "qa",
         vars: {
           scraper_name: scraper_name,
           collections: collections
         }
       }
     end
   end

This will create a page on DataHen with the scraper (ebay-scraper) and collection (listings) that we want
to validate saved in the vars. We will eventually create a parser that will send these values to the
dh_easy-qa gem which will load the relevant data from DataHen and perform QA on it using rules that we will
eventually define as well.

Let's go back into the root directory of the project

.. code-block:: bash

   $ cd ..
   $ ls -alth
   total 0
   drwxr-xr-x  10 johndoe  staff   320B 26 Nov 16:19 .git
   drwxr-xr-x   3 johndoe  staff    96B 26 Nov 16:15 seeder
   drwxr-xr-x   1 johndoe  staff    57B 26 Nov 16:15 Gemfile
   drwxr-xr-x   4 johndoe  staff   128B 26 Nov 16:15 .
   drwxr-xr-x  10 johndoe  staff   320B 26 Nov 15:59 ..

Now that we’ve created the seeder script, let’s see if there are any syntax error in it by trying the seeder
script.

.. code-block:: bash

   $ hen seeder try ebay-example-qa seeder/seeder.rb
   Trying seeder script
   =========== Seeding Executed ===========
   ----------------------------------------
   Would have saved 1 out of 1 Pages
   [
     {
       "url": "https://fetchtest.datahen.com/?scraper=ebay-scraper",
       "method": "GET",
       "page_type": "qa",
       "force_fetch": true,
       "vars": {
         "scraper_name": "ebay-scraper",
         "collections": [
           "listings"
         ]
       }
     }
   ]

Looks like our seeder script test was successful. Let's now create the config file to tell DataHen about our seeder.
Create a config.yaml file in the root project directory with the following content:

.. code-block:: bash

   seeder:
    file: ./seeder/seeder.rb
    disabled: false # Optional. Set it to true if you want to disable execution of this file

The config above simply tells DataHen where the seeder file is, so that it can be executed.

Let’s now commit our files with git and push them to Github. Add all of the current files with the following commands:

.. code-block:: bash

   $ git add .
   $ git commit -m 'create inital qa files'
   [master (root-commit) 7632be0] create initial qa files
    1 file changed, 5 insertions(+)
    create mode 100644 seeder/seeder.rb
    create mode 100644 Gemfile
    create mode 100644 Gemfile.lock
    create mode 100644 dh_easy.yaml
    create mode 100644 config.yaml

Next, let’s push it to an online git repository provider. In this case let’s push this to Github. In the example below it is
using our git repository, you should push to your own repository.

.. code-block:: bash

   $ git remote add origin https://github.com/DataHenHQ/ebay-scraper-qa.git
   $ git push -u origin master
   Counting objects: 4, done.
   Delta compression using up to 8 threads.
   Compressing objects: 100% (2/2), done.
   Writing objects: 100% (4/4), 382 bytes | 382.00 KiB/s, done.
   Total 4 (delta 0), reused 0 (delta 0)
   remote:
   remote: Create a pull request for 'master' on GitHub by visiting:
   remote:      https://github.com/DataHenHQ/ebay-scraper-qa/pull/new/master
   remote:
   To https://github.com/DataHenHQ/ebay-scraper-qa.git
    * [new branch]      master -> master
   Branch 'master' set up to track remote branch 'master' from 'origin'.

Congratulations, you’ve successfully seeded the scraper collections we want to validate with QA and are ready to
define some validation rules, which we will do in the next step.

Step 3: QA Validation Rules
---------------------------
Now that we have seeded a page to DataHen with info about the scraper that we want to perform QA on, we can define
some validation rules. Add some lines to the dh_easy.yaml file so that it looks like the following:

.. code-block:: yaml

   qa:
     scrapers:
       ebay-example:
         - listings
     individual_validations:
       url:
         required: true
         type: Url
       title:
         required: true
         type: String

Basically this will tell the QA gem to perform the following validations on the listings collection of our ebay-example scraper.
It will make sure the url and the title are both present and it will make sure that the url output
is actually a url and that the title output is a string. If any of these validations fail, the failure will be
returned in a summary and the specific listing will be returned with the corresponding failure.

Next we need to create a parser, which will parse the page that has our scraper (ebay-example) and collection (listings) info
saved in vars, and use the dh_easy-qa gem to perform QA. In our seeder we set the page_type to qa, so lets create a parser
named qa.rb inside a folder named parsers.

First lets create the parsers directory:

.. code-block:: bash

   mkdir parsers

Next create a file called qa.rb inside this parsers directory with the following lines:

.. code-block:: ruby

   require 'dh_easy/qa'
   DhEasy::Qa::Validator.new.validate_internal(page['vars'], outputs)

Here we are telling the QA gem to validate internal scrapers on DataHen and are passing the details of these scrapers inside
the vars. We also pass the outputs array which is a special reserved word in DataHen that is an array of job output to be saved.
This way the QA gem will be able to save the QA output to DataHen for you to see.

Let's retrieve the GID of the page that we seeded earlier so we can try it out locally. Run the following command in the project
root directory.

.. code-block:: bash

   hen scraper page list ebay-example-qa
   [
    {
     "gid": "fetchtest.datahen.com-1767f1fa6b7302b4a618b16b470fc1d2",
     "job_id": 9793,
     "job_status": "active",
     "status": "parsing_failed",
     "fetch_type": "standard",
     "page_type": "qa",
     "priority": 0,
     "method": "GET",
     "url": "https://fetchtest.datahen.com/?scraper=ebay-example",
     "effective_url": "https://fetchtest.datahen.com/?scraper=ebay-example",
     "headers": null,
     "cookie": null,
     "body": null,
     "created_at": "2019-08-09T21:44:18.709737Z",
     "no_redirect": false,
     "ua_type": "desktop",
     "freshness": "2019-08-09T21:44:18.735754Z",
     "fresh": true,
     "parsing_at": null,
     "parsing_failed_at": "2019-08-09T22:05:30.684121Z",
     "parsed_at": null,
     "parsing_try_count": 3,
     "parsing_fail_count": 3,
     "fetched_at": "2019-08-09T21:45:10.312099Z",
     "fetching_try_count": 1,
     "to_fetch": "2019-08-09T21:44:18.73264Z",
     "fetched_from": "web",
     "response_checksum": "9d650deb8d3fd908de452f27e148293d",
     "response_status": "200 OK",
     "response_status_code": 200,
     "response_proto": "HTTP/1.1",
     "content_type": "text/html; charset=utf-8",
     "content_size": 555,
     "vars": {
      "collections": [
       "listings"
      ],
      "scraper_name": "ebay-example"
     },
     "failed_response_status_code": null,
     "failed_response_headers": null,
     "failed_response_cookie": null,
     "failed_effective_url": null,
     "failed_at": null,
     "failed_content_type": null,
     "force_fetch": false
    }
   ]

We can see the scraper name and collection are both present in "vars," but what we are interested in is the gid which
will look something like, "fetchtest.datahen.com-1767f1fa6b7302b4a618b16b470fc1d2."
We can use the gid to try out our parser, which will perform the QA, on this page.

Run the following command, replacing the <gid> part with your gid value:

.. code-block:: bash

   hen parser try ebay-example-qa parsers/qa.rb <gid>

The output should look something like:

.. code-block:: bash

   Trying parser script
   getting Job Page
   1
   2
   validating scraper: ebay-example
   Validating collection: listings
   data count 42
   =========== Parsing Executed ===========
   ----------------------------------------
   Would have saved 1 out of 1 Outputs
   [
     {
       "pass": "true",
       "_collection": "ebay-example_listings_summary",
       "total_items": 42
     }
   ]

This means that our validation rules in dh_easy.yaml have passed for each of the 42 items. This also means that you have
successfully performed basic QA on your scraper! We will look at more advanced settings in the next section.

Additional Validation Rules
---------------------------

These are examples of all the available validation rules. You use them by adding them to dh_easy.yaml nested under 'individual_validations.'
For example, here is a validation rule that makes sure a field named, "Title" is present, is a string, and has a length of 10.

.. code-block:: yaml

  qa:
    individual_validations:
      Title:
        required: true
        type: String
        length: 10

Here are all the possible validation rules and values:

------------

.. code-block:: yaml

   length: 5

Validates the length of a field value. The field value can be an Integer, Float, or a String.

------------

.. code-block:: yaml

   type: String

Validates that a value is a String.

------------

.. code-block:: yaml

   type: Integer

Validates that a value is an Integer. The value can be a number in a String. Examples that would pass: 10, '10', '1,000'.

------------

.. code-block:: yaml

   type: Float

Validates that a value is a Float (a number with a decimal point). The value can be a number with decimal points in a String.
Examples that would pass: 2.0, '3.14', '99.99'.

------------

.. code-block:: yaml

   type: Date
   format: '%d-%m-%Y'

Validates that a value is a date. A format is required using `Ruby strftime <https://apidock.com/ruby/DateTime/strftime>`_.

------------

.. code-block:: yaml

   type: Url

Validates that a value is a valid url.

------------

.. code-block:: yaml

   value:
     equal: 'test'

You can also add validations that validate the value of a field itself. For example, the above validation will validate
that a field is equal to the string, 'test'.

------------

.. code-block:: yaml

   value:
     less_than: 5

You can also verify that a field is less than something.

------------

.. code-block:: yaml

   value:
     greater_than: 100

You can also verify that a field is more than something.

------------

.. code-block:: yaml

   value:
     regex: "^(Monday|Tuesday|Wednesday|Thursday|Friday|Saturday|Sunday)$"

Validates the value of a field using a regular expression. For example, you could validate that a value is a
phone number or a day of the week like the above example. Regex uses case ignore by default.

------------

.. code-block:: yaml

   title:
     value:
       equal: 'Test title'
       if:
         search_input:
           value:
             equal: 'Search'

You can also implement conditions on value validations. For instance, the above example validates that the value of
a field named, 'title' has a value equal to 'Test title' only if the value of the field named, 'search_input' has
a value equal to, 'Search.' If statements currently support value checks with the same options as a normal value
check (less_than, greater_than, regex, and equal).

------------

.. code-block:: yaml

   title:
     required: true
     if:
       search_input:
         value:
           regex: '(One|Two|Three)'

You can also implement an if condition on 'required.' The above example will only check if the 'title' field is required
if the field named, 'search_input' has a value equal to: 'One,' 'Two,' or 'Three.'

------------

.. code-block:: yaml

   title:
     required: true
     if:
       and:
         -
          field1:
            value:
              regex: '(One|Two)'
         -
          field2:
            value:
              less_than: 100

If conditions can also take 'and' and 'or' operators. The above example shows a validation that will only check if the 'title'
field is required if the field named, 'field1' has a value equal to: 'One' or 'Two' and the field named, 'field2' has a value
that is less than 100.

Group Validations
-----------------

In addition to these individual validations you can also perform more complicated validations on the data as a whole.
For example, you may want to ensure that a specific field has ranked values and are in order. To add group validations
create a file named group_validations.rb in your QA scraper root directory with the following (data is an array of the
items you are performing QA on):

.. code-block:: ruby

   module GroupValidations
     def validate_count
       errors[:group_validation] = { failure: 'count' } if data.count > 100
     end
   end

This example would create an error if the total number of items is greater than 100. Let's look at another another example.

.. code-block:: ruby

   module GroupValidations
     def validate_count
       errors[:group_validation] = { failure: 'count' } if data.count > 100
     end

     def validate_unique_ids
       ids = data.collect{|item| item['id'] }
       errors[:group_validation] = { failure: 'unique_ids' } if ids.uniq.count != ids.count
     end
   end

This would add another validation that checks to make sure all item 'id' values are unique. You can edit these examples
to create your own.

Thresholds
----------

Thresholds are useful if you want to suppress errors based on frequency. You can suppress errors on the basis of the number of
errors relative to the number of items you are performing QA on. The threshold itself is a number between 0 and 1 where 1
means that if any error occurs, the error will show. A threshold of 0 means we are ignoring all errors. There are multiple
ways you can set a threshold which we will go through below.

We can set a threshold on a per field basis which will apply to all scrapers. This can be done by adding "threshold" to the
dh_easy.yaml file to a specific field just like a rule. For example, the following will add a threshold that will only show
errors on the "Title" field for every scraper if the occurance rate is above 60%.

.. code-block:: yaml

  qa:
    individual_validations:
      Title:
        threshold: 0.6
        required: true
        type: String
        length: 10

We can also set thresholds on a per field and a per scraper scraper basis at the same time. Using the "Title" field example, this means you could
have a threshold of 0.6 for the Title on one scraper and have 1.0 for another scraper. In order to implement different thresholds for individual scrapers
you can create a file called thresholds.yaml in your scraper root directory. Here is an example of a thresholds.yaml file that would apply different
thresholds for a scraper named ebay1 and a scraper named ebay2.

.. code-block:: yaml

   ---
   ebay1:
      Title:
        threshold: 0.6
        required: true
        type: String
   ebay2:
      Title:
        threshold: 1.0
        required: true
        type: String

External sources
----------------

In addition to performing validation on scrapers that run on DataHen (internal sources) you can also perform validation on external sources.
For example, if you have a scraper that runs somewhere else, you can validate it by ingesting a json endpoint. Here is an example seeder
for an external source:

.. code-block:: ruby

   pages << {
      url: "http://dummy.restapiexample.com/api/v1/employees",
      method: "GET",
      force_fetch: true,
      freshness: Time.now.iso8601,
      vars: {
        collection_id: "employees-1"
      }
   }

This seeder could be expanded to seeding multiple endpointpoints by loading a YAML file and iterating through like in Step 2 above. After seeding
our external json endpoint we can now write a parser such as the following:

.. code-block:: ruby

   require 'typhoeus'
   require 'json'
   require 'dh_easy/qa'

   collection_name = page['vars']['collection_id']
   json = JSON.parse(content)
   qa = DhEasy::Qa::Validator.new(json, {})
   qa.validate_external(outputs, collection_name)

We can also implement thresholds with external sources by loading a thresholds yaml and passing it into the validator options. We can update our
parser so that it looks like the following:

.. code-block:: ruby

   require 'typhoeus'
   require 'json'
   require 'yaml'
   require 'dh_easy/qa'

   collection_name = page['vars']['collection_id']
   file_path = File.expand_path('thresholds.yaml', Dir.pwd)
   thresholds = YAML.load(File.open(file_path))
   options = { 'thresholds' => thresholds[collection_name] }

   json = JSON.parse(content)
   qa = DhEasy::Qa::Validator.new(json, options)
   qa.validate_external(outputs, collection_name)

How to handle duplicate pages from different categories
=======================================================

It is pretty common for a product belongs to several categories and subcategories within a store's website causing the same product to have several different URLs and therefore causing a scraper to download the same product page multiple times.

For example, a simple category page parser to extract and enqueue it's products would look like this:

.. code-block:: ruby

  # ./parsers/category.rb
  
  # extract category name
  html = Nokogiri::HTML(content)
  category = html.at(.h1).text
  
  # iterate products
  products = html.css('.list .item')
  products.each do |product|
    # extract product url
    url = product.css('a').first.attr(:href)
    
    # enqueue product page
    pages << {
      url: url,
      page_type: 'product',
      vars: {
        category: category
      }
    }
    save_pages pages if pages.count > 99
  end

And it's product page parser script would look like this:

.. code-block:: ruby

  # ./parsers/product.rb
  
  require 'uri'
  
  # extract product ID
  url = URI(page['effective_url'])
  id = url.path.scan(/\/([0-9]+)$/).first.first
  
  # save category-product relation
  outputs << {
    _collection: "cat_#{page['vars']['category']}",
    _id: id
  }
  
  # dedup outputs by using product id
  html = Nokogiri::HTML(content)
  outputs << {
    _collection: 'products',
    _id: id,
    name: html.at('h1').text
  }

Usually, these kind of duplicates can avoid to generate duplicated `outputs` by simply checking the canonical URL of the product (usually found on the product page metadata) or by simply using the product ID as the `output`'s `_id` field like we did on the previous example, however, the problem is that the duplicated pages would still be fetched, increasing both the time and cost of a scraper.

To avoid this, we can make use of `about:blank` URL, `driver.name` and `vars` attributes as a gate to prevent duplicated product page URL from being downloaded because `about:blank` pages skips the fetching process all together and it is sent directly into `to_parse` status.

For example, we can adapt the previous example to avoid fetching product page duplicates by creating an `about:blank` page to work as a gate, like this:

.. code-block:: ruby

  # ./parsers/category.rb
  
  require 'uri'
  
  # extract category name
  html = Nokogiri::HTML(content)
  category = html.at(.h1).text
  
  # iterate products
  products = html.css('.list .item')
  products.each do |product|
    # extract product url
    raw_url = product.css('a').first.attr(:href)
    
    # extract product ID
    url = URI(raw_url)
    id = url.path.scan(/\/([0-9]+)$/).first.first
    
    # enqueue product page
    pages << {
      url: 'about:blank',
      page_type: 'product_gate',
      driver: {
        name: "prod_#{id}"
      },
      vars: {
        product_id: id,
        url: raw_url
      }
    }
    save_pages pages if pages.count > 99
    
    # save category-product relation
    outputs << {
      _collection: "cat_#{category}",
      _id: id
    }
    save_outputs outputs if outputs.count > 99
  end

Notice we moved the `save category-product relation` from `./parsers/product.rb` to the `./parsers/category.rb` script since the dudep won't be handled by the product page parser script.
 
Then we enqueue the real product page on the `product_gate` parser, like this:

.. code-block:: ruby

  # ./parsers/product_gate.rb
  
  # enqueue real product page
  vars = page['vars']
  pages << {
    url: vars['url'],
    page_type: 'product',
    vars: {
      product_id: vars['product_id']
    }
  }

And finally, we parse and save the product data:

.. code-block:: ruby

  # ./parsers/product.rb
  
  # save product data
  html = Nokogiri::HTML(content)
  outputs << {
    _collection: 'products',
    _id: page['vars']['product_id'],
    name: html.at('h1').text
  }

This way our system will handle the product page dedup and fetch a single product page instead of several duplicates of the same product page.
