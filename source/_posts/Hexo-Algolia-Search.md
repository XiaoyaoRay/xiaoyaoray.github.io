---
title: Hexo-Algolia-Search
copyright: true
date: 2018-05-30 14:07:52
tags:
- Hexo
---

NexT provides Algolia search plugin for index your hexo website content. To use this feature, make sure that the version of NexT you are using is after the v5.1.0 release. What you should note here is that only turn on `enable` of `algolia_search`in `next/_config.yml` cannot let you use the algolia search correctly, you need to install corresponding [Hexo Algolia](https://github.com/oncletom/hexo-algolia) plugin to seach your website with Algolia. Follow the steps described below to complete the installation of Algolia search.
<!--more-->
1. Register at [Algolia](https://www.algolia.com/), you can log in directly using GitHub or Google Account. Upon Customer’s initial sign-up for an Account, Customer will have a free, fourteen (14) day evaluation period (the “Evaluation Period”) for the Algolia Services commencing on the Effective Date, subject to the limitations on Algolia’s website. After that, Algolia offers a free, branded version for up to 10k records and 100k operations per month.

2. If a tutorial pops up, you can skip it. Go straight to create an `Index` which will be used later.

   [![img](https://camo.githubusercontent.com/244ac146bf644651ed0f6341b85d6aa23e0aea73/687474703a2f2f7468656d652d6e6578742e696973736e616e2e636f6d2f75706c6f6164732f616c676f6c69612f616c676f6c69612d737465702d322e706e67)](https://camo.githubusercontent.com/244ac146bf644651ed0f6341b85d6aa23e0aea73/687474703a2f2f7468656d652d6e6578742e696973736e616e2e636f6d2f75706c6f6164732f616c676f6c69612f616c676f6c69612d737465702d322e706e67)

3. Algolia requires users to upload their search index data either manually or via provided APIs. Install and configure [Hexo Algolia](https://github.com/oncletom/hexo-algolia) in your Hexo directory. This plugin will index your site and upload selected data to Algolia.

   ```
   $ cd hexo
   $ npm install --save hexo-algolia
   ```

4. Go to the `API Keys` page and find your credentials. You will need the `Application ID` and the `Search-only API key` in the following sections. The `Admin API key` need to keep confidential. Never store your Admin API Key as apiKey in the`_config.yml` file: it would give full control of your Algolia index to others and you don't want to face the consequences.

   [![img](https://user-images.githubusercontent.com/8521181/35479066-64e35aec-0428-11e8-91f9-1ec3afa45c5c.png)](https://user-images.githubusercontent.com/8521181/35479066-64e35aec-0428-11e8-91f9-1ec3afa45c5c.png)

5. In the `API Keys` page, click the `ALL API KEYS` and the `edit` option in the created APIKEY to activate a pop-up box where you can setup authorizations and restrictions with a great level of precision. Check `Add records`, `Delete records`, `List indices`, `Delete index` features in ACL permissions that will be allowed for the given API key. And then click the `Update` button.

   [![img](https://user-images.githubusercontent.com/8521181/35479064-611aa0b4-0428-11e8-85a1-cfb449b486ec.png)](https://user-images.githubusercontent.com/8521181/35479064-611aa0b4-0428-11e8-85a1-cfb449b486ec.png)[![img](https://user-images.githubusercontent.com/8521181/35479084-d4f7ac02-0428-11e8-95a6-c4e3b1bef47b.png)](https://user-images.githubusercontent.com/8521181/35479084-d4f7ac02-0428-11e8-95a6-c4e3b1bef47b.png)

6. In your site's `_config.yml`, add the following configuration and replace the `applicationID` & `apiKey` & `indexName`with corresponding fields obtained at Algolia.

   ```
   algolia:
     applicationID: 'Application ID'
     apiKey: 'Search-only API key'
     indexName: 'indexName'
     chunkSize: 5000
   ```

7. Run the following command to upload index data, keep a weather eye out the output of the command.

   ```
   $ export HEXO_ALGOLIA_INDEXING_KEY=Search-Only API key # Use Git Bash
   # set HEXO_ALGOLIA_INDEXING_KEY=Search-Only API key # Use Windows command line
   $ hexo clean
   $ hexo algolia
   ```

   [![img](https://camo.githubusercontent.com/747791b2da3308ee2a7fc18c664da9dc9d0e6f5f/687474703a2f2f7468656d652d6e6578742e696973736e616e2e636f6d2f75706c6f6164732f616c676f6c69612f616c676f6c69612d737465702d342e706e67)](https://camo.githubusercontent.com/747791b2da3308ee2a7fc18c664da9dc9d0e6f5f/687474703a2f2f7468656d652d6e6578742e696973736e616e2e636f6d2f75706c6f6164732f616c676f6c69612f616c676f6c69612d737465702d342e706e67)

8. Change dir to NexT directory, and install module to `source/lib` directory.

   ```
   $ cd themes/next
   $ git clone https://github.com/theme-next/theme-next-algolia-instant-search source/lib/algolia-instant-search
   ```

   If you want to use the CDN instead of clone this repo, then need to **set vendors** in NexT `_config.yml` file:

   ```
   vendors:
     ...
     # Internal version: 1
     # https://www.algolia.com
     algolia_instant_js: https://cdn.jsdelivr.net/npm/instantsearch.js@2.4.1/dist/instantsearch.js
     algolia_instant_css: https://cdn.jsdelivr.net/npm/instantsearch.js@2.4.1/dist/instantsearch.min.css
     ...
   ```

9. In `next/_config.yml`, turn on `enable` of `algolia_search`. At the same time, you need to **turn off other search plugins** like Local Search. You can also adjust the text in `labels` according to your needs.

   ```
   # Algolia Search
   algolia_search:
     enable: true
     hits:
       per_page: 10
     labels:
       input_placeholder: Search for Posts
       hits_empty: "We didn't find any results for the search: ${query}"
       hits_stats: "${hits} results found in ${time} ms"
   ```

参考：

[中文参考](http://theme-next.iissnan.com/third-party-services.html#search-system)

https://github.com/LouisBarranqueiro/hexo-algoliasearch

https://github.com/theme-next/hexo-theme-next/blob/master/docs/ALGOLIA-SEARCH.md

https://github.com/theme-next/theme-next-algolia-instant-search
