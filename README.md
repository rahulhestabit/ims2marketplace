# ims2marketplace


### Initial Vue Storefront import

Now, You're ready to run the importer.

We'll use in the following example - the ENV variables. The simplest command sequence to perform full reindex is:

```bash
export TIME_TO_EXIT=2000
export MAGENTO_CONSUMER_KEY=byv3730rhoulpopcq64don8ukb8lf2gq
export MAGENTO_CONSUMER_SECRET=u9q4fcobv7vfx9td80oupa6uhexc27rb
export MAGENTO_ACCESS_TOKEN=040xx3qy7s0j28o3q0exrfop579cy20m
export MAGENTO_ACCESS_TOKEN_SECRET=7qunl3p505rubmr7u1ijt7odyialnih9

echo 'Default store - in our case United States / en'
export MAGENTO_URL=http://demo-magento2.vuestorefront.io/rest
export INDEX_NAME=vue_storefront_catalog

node --harmony cli.js categories --removeNonExistent=true
node --harmony cli.js productcategories --partitions=1
node --harmony cli.js attributes --removeNonExistent=true
node --harmony cli.js taxrule --removeNonExistent=true
node --harmony cli.js products --removeNonExistent=true --partitions=1
node --harmony cli.js reviews
```


**Please note:**
- `--removeNonExistent` option means - all records that were found in the index but currently don't exist in the API feed - will be removed. Please use this option ONLY for the full reindex!
- `INDEX_NAME` by default is set to the `vue_storefront_catalog` but You may set it to any other elastic search index name.
- The `categories` importer option `--generateUniqueUrlKeys` is by default set to true. This is due the fact that in Magento2, the `category.url_key` field is not mandatory unique and from v. 1.7 Vue Storefront uses the `category.url_key` to display the category details without any client's side modification.
- `PRODUCTS_EXCLUDE_DISABLED` by default is set to `false`. To only import enabled products set this to `true`.

**Cache invalidation:** Recent version of Vue Storefront do support output caching. Output cache is being tagged with the product and categories id (products and categories used on specific page). ims2marketplace can invalidate cache of product and category pages if You set the following ENV variables:

```bash
export VS_INVALIDATE_CACHE_URL=http://localhost:3000/invalidate?key=aeSu7aip&tag=
export VS_INVALIDATE_CACHE=1
```

- `VS_INVALIDATE_CACHE_URL` is a cache to the Vue Storefront instance - used as a webhook to clear the output cache.

## Advanced usage

Start Elasticsearch and Redis:
- `docker-compose up`

Install:
- `npm install`
- `cd src`

Config - see: config.js or use following ENV variables: 
- `MAGENTO_URL`
- `MAGENTO_CONSUMER_KEY`
- `MAGENTO_CONSUMER_SECRET`
- `MAGENTO_ACCESS_TOKEN`
- `MAGENTO_ACCESS_TOKEN_SECRET`
- `DATABASE_URL` (default: 'elasticsearch://localhost:9200/vue_storefront_catalog')


Run:
- `cd src/` and then:
- `node --harmony cli.js fullreindex` - synchronizes all the categories, products and links between products and categories

Other commands supported:
- `node --harmony cli.js products --partitions=10`
- `node --harmony cli.js products --partitions=10 --initQueue=false` - run the products sync worker (product sync jobs should be populated eslewhere - it's used to run multi-tenant environment of workers)
- `node --harmony cli.js products --partitions=10 --delta=true` - check products changed since last run; compared by updated_at field
- `node --harmony cli.js productcategories` - to synchronize the links between products and categories it *should be run before* products synchronization because it populates Redis cache assigments for product-to-category link
- `node --harmony cli.js categories`
- `node --harmony cli.js products --adapter=magento --partitions=1 --skus=24-WG082-blue,24-WG082-pink`  - to pull out only selected SKUs
- `node --harmony cli.js productsworker --adapter=magento --partitions=10`  - run queue worker for pulling out individual products (jobs can be assigned by webapi.js microservice triggers; it can be called by webhook for example from within Magento2 plugin)
- `node --harmony webapi.js` - run localhost:3000 service endpoint for adding queue tasks

WebAPI:
- `node --harmony webapi.js`
- `curl localhost:8080/api/magento/products/pull/WT09-XS-Purple` - to schedule data refresh for SKU=WT09-XS-Purple
- `node --harmony cli.js productsworker` - to run pull request processor 

Available options:
- `partitions=10` - number of concurent processes, by default number of CPUs core given
- `adapter=magento` - for now only Magento is supported
- `delta` - sync products changed from last run
- command names: `products` / `attributes` / `taxrule` / `categories` / `productsworker` / `productcategories` / `productsdelta`


