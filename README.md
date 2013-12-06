Java M2X API Client
=====================
 
The AT&T M2X API provides all the needed operations and methods to connect your devices to AT&T's M2X service. This
client provides an easy to use interface for your favorite language, Java.
 
 
Getting Started
------------
1. Signup for an M2X Account: https://m2x.att.com/signup
2. Obtain your *Master Key* from the Master Keys tab of your Account Settings: https://m2x.att.com/account
3. Create your first Data Source Blueprint and copy its *Feed ID*: https://m2x.att.com/blueprints
4. Review the M2X API Documentation: https://m2x.att.com/developer/documentation/overview
 
If you have questions about any M2X specific terms, please consult the M2X glossary: https://m2x.att.com/developer/documentation/glossary
 
 
Setup
-----
 
At the present time client is not submitted to any of existing Maven repositories. To start using it now please build
and install it locally.
 
[Maven 2](http://maven.apache.org/) is required.
 
Add the following to pom.xml to start using M2XClient.
 
```
<dependency>
	<groupId>com.att.m2x</groupId>
    <artifactId>m2x-java-client</artifactId>
	<version>0.0</version>
</dependency>
```


Example Usage
-------------

To create a client instance the only one parameter, the API Key, is required.
Your Master API Key can be found in your account settings, also a feed API key is available in your Data Source details
screen. To create a client instance just do
 
```
	import com.att.m2x.client.M2XClient;
	
	M2XClient client = new M2XClient("your api key here")
```
 
The client provides an interface to access your Blueprints, Batches,
DataSources, Feeds and Keys.


### Blueprints
 
Get an instance of `BlueprintResource` before starting to work with Blueprints:
 
```
	client.blueprints()
```
 
This is thread-safe objects, so they can be stored and reused.
 
 
#### Fetch a collection of items:
 
```
	Page<Blueprint> blueprints = client.blueprints().list()
```
 
Page object is a wrapper above every portion of available blueprints. Use `getData()` methods of `Page` object to get
list of blueprints.
 
Use params builder to change how many objects are included in the page, or move forth above pages:
 
```
	import static com.att.m2x.client.builder.QueryBuilders.*
	
	blueprints = client.blueprints().list(limit(100));
	...
	blueprints = client.blueprints().list(page(2).limit(20));
```


#### Single item retrieval
 
```
	Blueprint blueprint = client.blueprints().get("188a0afb3adc379706e780a4eafbd153");
```
 
The first parameter of get() is the Blueprint id `blueprint.getId();`


#### Create new instance
 
Library provide builder to make this process funny:
 
```
	import static com.att.m2x.client.builder.ModelBuilders.*
  
	Blueprint blueprint = client.blueprints().create(
   	     blueprint().name("Sample Blueprint").visibility(Visibility.PRIVATE)
                	.tags("Tag One", "Tag Two")
	);
```
 
Or use `Map<String, Object>` if builder is not ***TBD***
 
```
	Map<String, Object> data = new HashMap<String, Object>();
	data.put("name", "Sample Blueprint");
	data.put("visibility", Visibility.PRIVATE);
	
	Blueprint blueprint = client.blueprints().create(data);
```
 
In case of one of the required parameters are missing library will throw `UnprocessableEntityException`.


#### Update item
 
Like in case of creating, use two ways to create objects, with builder or with date in map.
 
```
	import static com.att.m2x.client.builder.ModelBuilders.*
  
    client.blueprints().update(blueprint.getId(),
        	blueprint().name("Sample Blueprint").visibility(Visibility.PRIVATE)
                	.tags("Tag One", "Tag Two")
	);
	
```


#### Removal
 
```
    client.blueprints().delete(blueprint.getId());
 
```


### DataSources
 
`DataSourceResource` has the same API like `BlueprintResource`. Get it from `M2XClient`:
 
```
	DataSourceResource dataSetResource = client.dataSources()
```

and use it in the same way. Except creating new instance and update:
 
```
	import static com.att.m2x.client.builder.ModelBuilders.*
  
	DataSource blueprint = client.dataSources().create(
        	dataSource().name("Sample Blueprint").visibility(Visibility.PUBLIC)
                	.tags("Tag One", "Tag Two")
	);
```


### Batches
 
Get `BatchResource` to start working with batches:
 
```
	client.batches();
```
 
This resource provides the same basic api to work with batches like `BlueprintResource`.


#### List DataSources from a Batch
 
To fetch all the datasources in given batch use the following snippet:
 
```
	Page<DataSource> sources = client.batches().batch("188a0afb3adc379706e780a4eafbd153").dataSources();
```
 
The first parameter of batch() is the Batch id.


#### Add new DataSource to Batch
 
```
	client.batches().batch("b1e8abbad65cb52b0d75eb2e63efa782").addSerial("ABC1234");
```


### Keys
 
All operation for Keys is placed inside the `KeyResource`:
 
```
	client.keys();
```


### Fetch the collection of keys
 
Return list of all registered keys:
 
```
	List<Key> keys = client.keys().list();
```
 
Or return list of keys for a given Feed:

```
	List<Key> keys = client.keys().list("61179472a42583cffc889478010a092a");
```


#### Create new key
 
Can be done with builder or map:
 
```
    Key key = client.keys().create(key().name("Test Key").permissions(Permission.GET));
```

In case using of map please keep in mind that permissions should be placed inside either list or set.


#### Update key
 
```
    client.keys().update("b1e8abbad65cb52b0d75eb2e63efa782", key().name("Test Key").permission(Permission.GET));
```


#### Delete key
 
```
    client.keys().delete("b1e8abbad65cb52b0d75eb2e63efa782");
```


#### Key regeneration
 
```
	Key key = client.keys().regenerate("b1e8abbad65cb52b0d75eb2e63efa782");
```


### Feeds
 
Feed is accessible by the `feeds()` method of `M2XClient` instance.
Feeds creation is done when creating a DataSource, Blueprint or Batch.
So that is why `FeedResource` can't create or update or delete Feed.


#### Fetch a list
 
Get first page of feeds:
 
```
	Page<Feed> feeds = client.feeds().list()
```
 
Get given page with non-default page size:

```
	import static com.att.m2x.client.builder.QueryBuilders.*
	
	Page<Feed> feeds = client.feeds().list(page(2).limit(25));
```
 
Search by name and filter type:

```
	import static com.att.m2x.client.builder.QueryBuilders.*
	
	Page<Feed> feeds = client.feeds().list(query("Feed Name").type(FeedType.BATCH));
```

Search by location:

```
	import static com.att.m2x.client.builder.QueryBuilders.*
	
	Page<Feed> feeds = client.feeds().list(latlong(-37.978, -57.547).within(100, DistanceUnit.KM));
```

Every search can be combined with pagination:

```
	import static com.att.m2x.client.builder.QueryBuilders.*
	
	Page<Feed> feeds = client.feeds().list(
        	latlong(-37.978, -57.547).within(100, DistanceUnit.KM),
        	limit(25)
	);
```


#### Get an item
 
```
	Feed feed = client.feeds().get("0e545075fd71aaabf5e85bfb502ea35a");
```


#### Working with streams

Get list of streams:

```
    List<Stream> streams = client.feed("0e545075fd71aaabf5e85bfb502ea35a").streams().list();
```

Or in a bit longer form:

```
    List<Stream> streams = client.feeds().feed("0e545075fd71aaabf5e85bfb502ea35a").streams().list();
```

Create or update stream:

```
    Stream stream = client.feed("").streams().create("Sample Stream", new Unit("speed", "kph"));
    
    client.feed("").streams().update("Sample Stream", new Unit("speed", "kph"));
```

Add value to stream:

```
    client.feed("Feed Id").streams().addValues("Stream Name", new Value("Value to add"),
            new Value("Value to add", new Date()));
```

It's possible to add several values in multiple streams:

```
    Map<String, List<Value>> wrapper = new HashMap<String, List<Value>>();
    List<Value> values = new ArrayList<Value>();
    values.add(new Value("Value to go"));
    wrapper.put("Stream Name"), values);

    client.feed("Feed Id").addValues(wrapper);
```

Get list of values:

```
    ValueListResponse response = client.feed("0e545075fd71aaabf5e85bfb502ea35a")
            .streams().values("Stream Id");
```

Method `values()` also support filtering values by start and the end date:

```
    import static com.att.m2x.client.builder.QueryBuilders.*
    
    ValueListResponse response = client.feed("0e545075fd71aaabf5e85bfb502ea35a")
            .streams().values("Stream Id", start(new Date).limit(15));
```


#### Working with triggers

Get the list of triggers:

```
    List<Trigger> triggers = client.feed("0e545075fd71aaabf5e85bfb502ea35a").triggers().list();
```

Get specific trigger:

```
    List<Trigger> triggers = client.feed("0e545075fd71aaabf5e85bfb502ea35a").triggers().get("1");
```
    
Create a new instance with builder:

```
    import static com.att.m2x.client.builder.ModelBuilders.*

    Trigger trigger = client.feed("0e545075fd71aaabf5e85bfb502ea35a").triggers().create(
            trigger().stream("Stream Id").name(name)
                    .condition(Condition.EQUAL).value(String.valueOf(10))
                    .callback("http://example.com")
    );
```

Delete trigger:

```
    client().feed("0e545075fd71aaabf5e85bfb502ea35a").triggers().delete("2");
```

Test trigger:

```
    client().feed("0e545075fd71aaabf5e85bfb502ea35a").triggers().test("1");
```

