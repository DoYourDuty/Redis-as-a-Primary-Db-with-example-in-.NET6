# Redis as a primary database?

> ▶️ [Youtube](https://www.youtube.com/watch?v=GgyizgXwXAg) 🌐 [Snippets](https://dotnetplaybook.com/redis-as-a-primary-database/)


 Redis is renowned for its speed and use as a cache, but can we use Redis as our primary application database? In this tutorial we find out if we can build a .NET 6 API using Redis as the primary db…

----------

> # What You’ll Learn
### In this step by step tutorial you will learn:
 - What caching is (we’ll cover theory but will not be implementing)
 - What Redis is, along with its main data types
 - How to set up Redis in a Docker container and run the Redis CLI
 - Approaches for connecting and working with Redis in .NET
 - Using Redis as our .NET application primary database

----------

> ## **What is Redis?** 
> Taken from https://redis.io

**Redis is an open source, in-memory data structure store, used as a database, cache and message broker.**

 A great summary, (and why I didn’t bother attempting my own!), but if you’re anything like me you’re probably not that familiar with Redis working as a database or message broker, (as opposed to a cache which is what it’s know for).

 Today we’re going to focus on **using Redis as Database**, but before we jump to that, lets look at why Redis is such a popular choice for caching…

----------

> ## **What is caching?**


 The use-case for caching is simple: caching is employed to make the serving of data faster. That’s it. But how is this achieved? Let’s first look at an example where caching is not used:


![alt](https://i1.wp.com/dotnetplaybook.com/wp-content/uploads/2021/11/2021-11-23_20-32-15.png?w=679&ssl=1)

 In the above example we see a very simple solution architecture comprising a Client App, an API and a Database. The sequence depicted above is as follows:

 1. The Client app makes a call to the API to get some data
 2. The API in turn reaches out to the database (where data is  stored on disk)
 3. A query is run to fetch the data (let’s say this is a  relational / SQL-based DB)
 4. Data is returned to the API
 5. Data is returned to the Client
 6. Resulting in a total execution time


 Now you may not see an issue with this, and to be honest in many cases, there is no issue, especially when:

 - Solution components are local to each other (non distributed  apps)
 - Datasets are “small”
 - Queries are well designed and optimized
 - Indexes are in place

 However if any or all of the above characteristics are not evident, then we’ll probably see request times reach unacceptable levels especially when viewed in the context of interactive, customer-centirc apps, where near real-time responses can be expected…
> **Adding a Cache**

 This is where caching comes in… Let’s take a look at the same solution and scenario, except this time a cache has been introduced:

![Cache No Hit Sequence](https://i0.wp.com/dotnetplaybook.com/wp-content/uploads/2021/11/2021-11-23_20-56-15.png?w=869&ssl=1)

 Now your first reaction to this is that we have made things worse! As we have:

 - Added more (unnecessary?) technical complexity to our solution
 - Increased the total request time! (well not really, see note below)

The scenario executed as follows:

1. The Client app makes a call to the API to get some data
2. The API attempts to get this data from an In-memory cache (e.g. Redis)
3. The data required is not in the cache…
4. The API attempts to get the data from the original database
5. We have the same or similar database fetch time
6. Data is returned to the API
7. The API Caches this data
8. The API returns the data to the client
9. Total Request time has “increased”

>> Note: The sequence diagram above depicts quite a dramatically increased total response time – the reality is quite different… The times taken to both query the cache (steps 2 & 3) and then update the cache (step 7), are going to be so small that in-practice they would not be noticed by a human user. The overall total request time, while probably greater than the 1st scenario, is only going to be so by an order of milliseconds. This point in-fact is a clue as to the power of caches, and in particular – Redis.


-------

> **Hitting the Cache**

For our final example of using a cache we’ll depict a scenario where this time we hit the cache:

![Cache Hit](https://i2.wp.com/dotnetplaybook.com/wp-content/uploads/2021/11/2021-11-24_18-22-25.png?w=822&ssl=1)

The scenario runs as follows:

1. The Client app makes a call to the API to get some data
2. The API attempts to get this data from an In-memory cache (e.g. Redis)
3. Data is held in the cache and fetched
4. Data is returned to the API
5. Data is returned to the Client
6. Resulting in a total execution time

 This is effectively the exact same scenario as the first, all that’s changed is the source of the data, in this case an in-memory cache (Redis) which serves the data much faster than its (disk-based) database equivalent.

> **Data expiry**

 There is one further consideration with caching. The fact still remains that the cache is not the primary data source, it’s a temporary source of cached data, so it’s conceivable, (and indeed likely), that the data in the primary data source will be updated at some point, rendering the cached data outdated.

This scenario highlights two things:

1. Caching is probably not that suited to data that changes very frequently
2. When you do use caching, you should employ some form of expiry / refresh of the cache

 By employing an expiry strategy you help ensure that data served form the cache does not persist past it’s useful “use-by date”. As luck would have it, Redis has the native ability to set an expiry on the data it stores.

 We’ll leave caching there for now, as it’s not the real point of this article, but I did want to highlight how Redis has been use as a cache and why – basically it’s incredibly fast, (when compared to other databases).

----------
> **Redis as a Primary Database**

 Redis has been incredibly successful as a cache, to such an extent it has become synonymous with caching, and for the most part is the go-to solution when a “distributed” cache is required.

>> We term Redis a “distributed” cache is it’s external to the application it serves. This would be in contrast to an internal in-memory cache resident as part of the application, e.g. a cache built with IMemoryCache in .NET

 This success has come at a cost, specifically that most people, (myself included!), didn’t realize that Redis offers much more beyond “just” caching.

 Looking back at our discussion on caching, you’ll observe that for all intents and purposes, scenario 1 and scenario 3 are essentially doing the exact same thing. The only difference is platform we are using. So it begs question: why not just do away with the “database” and replace it completely with Redis as a database?

---------
> **Did he just say that?!**

 Probably the single biggest reason most people would put forward as to why we should not use Redis as our primary data source is that Redis is an “in-memory” solution, rendering it useless for persistent data storage. In short, if Redis crashes you’d loose all your data.

 This is completely incorrect.

 Redis offers an number of approaches to persistence, rivaling that of more established players like PostgreSQL. I’m not going to delve into this too much, as the team over at Redis have provided some excellent docs on the subject so you can read all about it yourself. So much for that blocker…

---------

> **No(t)SQL(Server)**

 For me though the biggest blocker in adopting Redis as my primary database has nothing to do with persistence, (obviously), but everything to do with the fact that its not a SQL-based database, or probably more correctly it’s not a Relational Database Management System (RDBMS).

 I appreciate that this speaks more to my preference for RDBMS’s, and in particular my **love affair with SQL Server** than anything else, but for me it’s still a blocker… In short Redis doesn’t work the way that I like or am familiar with, but hey, life’s all about trying new things so why don’t we give it a go…

-----------
> **Setting Up Our Project**

   We’re going to be using .NET 6.0 for our REST API, so of course you’ll need the .NET 6.0 SDK installed which you can get here. Once you have installed that, you should be able to issue the following command:

```
dotnet --version
```

You should see something similar to this:

![dotnet --version](https://i0.wp.com/dotnetplaybook.com/wp-content/uploads/2021/11/2021-11-24_20-49-01.png?w=396&ssl=1)

 Next we’re going to create our template project for our API, so again at a command prompt, move into your “working directory” and issue the following command to create an webapi template project:

```
dotnet new webapi -n CacheService
```

 Performing a directory listing you should see a folder has been created with the name of our project:

![Created Project](https://i0.wp.com/dotnetplaybook.com/wp-content/uploads/2021/11/2021-11-25_19-09-55.png?w=615&ssl=1)

 For the rest of this tutorial I’m going to work with VS Code, (you can get a copy here), so if you’re doing the same, at the command prompt type the following to open our project:

```
code CacheService
```

 This will open the CacheService project in VS Code.

> **Running Redis**

 Now before we start writing any code I want to get an instance of Redis up and running. There are many ways to achieve this, from using managed cloud offerings, to a local installation, but I’m going to use Docker.

 If you want to follow along with this approach then you’ll need to have Docker installed locally, for Windows and Mac it’s as simple as installing Docker Desktop, for Linux users, you’re so intelligent you don’t need me to tell you how to install Docker.

 With Docker installed, open a command line and type the following to ensure it’s up and running correctly:

```
docker  --version 
```

   If it’s been successfully installed you should see something similar to this:
 
![docker --version](https://i2.wp.com/dotnetplaybook.com/wp-content/uploads/2021/11/2021-11-24_20-35-57.png?w=473&ssl=1)

 We’re going to use Docker Compose to get our instance up and running as we can easily include the docker-compose file with our CacheService project. So back over in VS Code (or whatever text editor / IDE your using), create a docker-compose.yaml file in the root of the project as shown below:

![Docker Compose file in our project](https://i1.wp.com/dotnetplaybook.com/wp-content/uploads/2021/11/2021-11-25_19-23-51.png?w=398&ssl=1)

 Open the docker-compose file and add the following mark-up that will allow us to spin up a Redis instance running in a Docker container:

```
version: '3.8'
services:
  redis:
 image: redis
 container_name: redis_cache
 ports:
   - "6379:6379"
```

This simply:

- Creates a new “service” (think of that as a container)
- Requires the Redis image (pulled from Docker Hub)
- Gives our container an name: redis_cache
- Specifies an external to internal port mapping

Ensure you save the file then at a command prompt (ensuring you’re in the CacheService project folder) type the following:

```
docker-compose up -d
```

>> TIP: If you’re using VS Code you can hold down CTRL and hit the backtick key – the key with the ` character), to open the in-line terminal in VS Code.

 This will start a Docker Container running Redis. If you don’t have the Redis Docker image available locally that will also be pulled down for you:

![Running Redis in Docker](https://i1.wp.com/dotnetplaybook.com/wp-content/uploads/2021/11/2021-11-25_19-37-23.png?w=714&ssl=1)

 With Redis up an running, let’s spend a little bit of time with the Redis CLI to familiarize ourselves with some simple operations.

---------

> **Redis Basics**

 In this section we’ll connect into our Docker container and used the redis-cli tool to issue some commands. Now there is already a great introduction to Redis data types on the Redis site, so I’m not going to go into too much detail in this section as there’s no point duplicating what someone else has already written. We will cover some of the basics here though, and when it comes to coding our solution we’ll introduce some of the more complex data types.

-------

> **Connecting in**

 As we’re running our Redis instance in Docker, I’m simply going to attach to that instance and use the redis-cli from inside the container, this avoids having to install any further software on our machine. In order to do this we need to get the container id of our running Redis container, to do so type the following at a command prompt:

```
docker ps
```

This lists our running containers, so you should see something like this:

![Obtaining our container id](https://i1.wp.com/dotnetplaybook.com/wp-content/uploads/2021/11/2021-11-25_19-59-30.png?w=1008&ssl=1)

 Copy the container id then issue the following command to attach to our running container:

```
docker exec -it <YOUR CONTAINER ID> /bin/bash
```
 Hit enter and you should see something like this:

Attached terminal

Here you can see that we have attached to the running Redis container and have direct access to the command line. Here we can issue the redis-cli command to start the Redis command line tool:

Redis Command Line
Strings

Redis is essentially a key / value store – that’s it! You give a piece of data a Key, e.g. “platform:10001” and then assign it a value: “Docker”. Redis can work with a number of different data types that essentially follow this same pattern, but to begin with, we’ll take a look at the simplest of those data types: strings.

At the redis-cli type the following:

set platform:10001 Docker 

This will result in the following:

set a key

What we have done here “set” a key / value pair, with:

 key = platform:10001
 Value = Docker

We can retrieve the value by passing the key to a get command:

get platform:10001

With the following result:

get value

While the other data types in Redis are more complex than this simple string type, they are all basically variations on this theme:

 Set keys and values
 Get values (using the key)

It is this last point that took me a while to get, (no pun intended). What I mean by that is that as SQL user I found it difficult to get my head around the fact that there isn’t really a concept of “querying” in NoSQL databases such as Redis. Retrieval of data is essentially based on the provision of keys (and some other operations such as popping off of lists etc.)

But I soon realized I’d missed the point! It is this method of data retrieval that gives Redis it’s speed advantage. It doesn’t need to worry about complex table joins, unions, query execution optimization etc. It’s just basically works off of retrieving data from a key.

Now I have probably over-simplified things a bit here, as there are other ways to to get data back out of Redis, but we’ll cover those when we start coding…

For now let’s quit out of the redis-cli tool and and begin coding. To do so first issue the following at the redis-cli:

quit

Then exit the attached command-line session in the Docker container:

exit

This should return you back to your local command prompt:

Exiting attached session
Application Build

The following is a high-level representation of our application architecture:

Application Architecture

We’re now finally at the point where we can start to write some code! The steps we are going to follow are:

 Add the required package reference(s)
 Remove redundant / unnecessary template code
 Register Redis Connection
 Create Our Model
 Create Our Repository
 Create Our Controller
 Start experimenting with controller actions

We have a lot to do, so let’s get cracking.
Code on GitHub
If you don't fancy typing in code as you go (although I recommend that for learning purposes), you can download the code from GitHub here: https://github.com/binarythistle/Redis-as-a-Primary-Db-with-example-in-.NET6
Add Package References

There is a bit of potential confusion around which package reference to add to your project to enable you to use Redis from within .NET:

 Microsoft.Extensions.Caching.Redis
 Microsoft.Extensions.Caching.StackExchangeRedis
 StackExchange.Redis

To cut to the chase #1 is essentially deprecated so don’t use that, so you really have a choice of 2 or 3. Indeed Microsoft.Extensions.Caching.StackExchangeRedis depends on StackExchange.Redis, so what’s the difference?

It boils down to how you want to access and work with Redis, and there you have 2 choices:

 Use IDistibutedCache (a simpler approach, with a less rich feature set)
 Use IConnectionMultiplexer (a more complex approach, with complete feature set)

Microsoft.Extensions.Caching.StackExchangeRedis supports both approaches, while at the time of writing StackExchange.Redis only supports IConnectionMultiplexer.

IDistributedCache is slightly simpler to use, and offers a greater degree of abstraction from the core Redis commands, but I think this simplification comes at a cost, depriving you of being able to use some of the more interesting Redis commands and Data Structures.

We are therefore going to use IConnectionMultiplexer, which means we can use either package! Looking at the number of downloads for each on Nuget you’ll see the StackExchange.Redis is more popular, however I have a thing for using the “official” Microsoft packages where possible so I’m going to use Microsoft.Extensions.Caching.StackExchangeRedis, (again you can use either).

With that, at a command prompt “inside” the CacheService project directory, type the following:

dotnet add package Microsoft.Extensions.Caching.StackExchangeRedis

This should add a package reference to your CacheService.csproj file as follows:

Package Reference
Removing Redundant Template Code

When we created our webapi template project, we got with that some example code that we’re not going to use, so we want to remove the following files:

 WeatherForecast.cs
 Controllers/WeatherForecastController.cs

So please go ahead and remove those by which ever method makes most sense to you.

 TIP: You can right click files and folders from within the VS Code directory explorer and add, rename & delete files and folders as required.

You should end up with a project structure like this, (noting that the Controllers folder is empty):

After deletion
Connecting to Redis

Not unlike any other database, we need to configure a connection string that contains the location of our Redis server, along with any other detail like user names and passwords, (which in this case we don’t need). I’m going to place this connection string in the appsettings.Development.json file in project, so open that and add the following connection string:

 "ConnectionStrings":
  {
 "DockerRedisConnection": "127.0.0.1:6379"
  }

In the context of the wider file it should look like this, noting the required comma just before the ConnectionStrings attribute:

Redis connection string

The connection string is pretty simple: our local IP address followed by a colon and then the default Redis port which should look familiar to you by now. Again, there is no need for user names and passwords when using our Redis instance running in Docker.
Registering IConnectionMultiplexer

Next move into the Program.cs file:
WARNING: .NET 6 Change
If you're not using .NET 6.0 or above, you'll need to place the following code in the Startup class. With .NET 6.0 Microsoft have attempted to simplify application start up by removing the Startup class from some project types, hence why we are not using it here. It is still supported, but I'm going with the newer streamlined set up.

And add the following code just after the “Add services to container” comment:

builder.Services.AddSingleton<IConnectionMultiplexer>(opt => 
 ConnectionMultiplexer.Connect(builder.Configuration.GetConnectionString("DockerRedisConnection")));

Your Program.cs file should look like this:

Program.cs File

A few points to note:

 We needed to bring in StackExchange.Redis
 We need to reference Services (note the capital ‘S’) through our instance of WebApplicationBuilder called “builder“.
 We need to reference Configuration via our builder instance too.
 We read in our DockerRedisConnection string configured previously

Points 2 & 3 are really more call-outs for those .NET developers used to earlier versions of the framework, and in particular the use of the Startup class.
Creating our Model

We are going to work with a single, super simple Model representing a list of “Platforms”, e.g.: Docker, Redis, Kubernetes, .Net etc. To be honest it’s really not at all relevant to the article…

In our working project folder, do the following:

 Create a Models folder in the root of our project
 Into that folder create a file called: Platform.cs

When complete your project structure should look like this:

Add Models

Into the Platform.cs file place the following code:

using System.ComponentModel.DataAnnotations;

namespace CacheService.Models
{
 public class Platform
 {
  [Required]
  public string Id { get; set; } = $"platform:{Guid.NewGuid().ToString()}";
  
  [Required]
  public string Name { get; set; } = String.Empty;
 }
}

.NET 6 defaults to warning you about variables that could return a null exception, hence the use of default values for both our properties. You can turn off this behavior by editing your .csproj file and removing the following line of config:

<Nullable>enable</Nullable>

It’s probably best practice to keep it in though, (but I’ll admit I find it annoying!)
Redis Design Decision #1
With an RDBMS I'd usually let the database specify the Id of the entity and not define it myself as I've done here. As we are required to provide a key (aka Id) value upfront when adding to data to Redis I feel it's a necessary design decision.

Make sure you save the Platform.cs file before moving on.
Creating our Repository

I’m going to define the following methods for our repository, which will closely align to the API endpoints we are going to offer up:

 Create a new Platform
 Return an individual Platform (based on the Id)
 Return all Platforms

I feel these need no more explaining, so back over in our project create the following:

 A folder in the root of our project called Data
 A file named IPlatrformRepo.cs (inside the newly created Data folder)
 A file named RedisPlatformRepo.cs (inside the newly created Data folder)

When complete your project file structure should mirror the following:

Data folder
IPlatformRepo

Open the IPlatformRepo.cs file and add the following code:

using CacheService.Models;

namespace CacheService.Data
{
 public interface IPlatformRepo
 {
  void CreatePlatform(Platform plat);
  Platform? GetPlatformById(string id);
  IEnumerable<Platform?>? GetAllPlatforms();
 }
}

Here we are defining an Interface that we’ll implement in the next section, it simply defines what methods someone using this interface can expect to be implemented.
RedisPlatformRepo

This is really the most novel part of our app as this is where we’ll be developing all our “Redis code”. We’ll be creating a controller in the next section, but we’ll just be injecting an instance of IPlatformRepo into that, so it’s oblivious to what technology it’s using. Indeed this is the whole point of the repository pattern, we can swap out implementation classes at will without impacting consumers of the interface (in this case our Controller).

To begin we’re only going to fully implement the following methods:

 CreatePlatform
 GetPlatformById

We’ll just have GetAllPlatforms throw an exception for now, but we’ll come back to it later…

So in the RedisPlatformRepo.cs file add the following code:

using System.Text.Json;
using CacheService.Models;
using StackExchange.Redis;

namespace CacheService.Data
{
 public class RedisPlatformRepo : IPlatformRepo
 {
  private readonly IConnectionMultiplexer _redis;

  public RedisPlatformRepo(IConnectionMultiplexer redis)
  {
   _redis = redis;
  }

  public void CreatePlatform(Platform plat)
  {
   if (plat == null)
   {
    throw new ArgumentOutOfRangeException(nameof(plat));
   }

   var db = _redis.GetDatabase();

   var serialPlat = JsonSerializer.Serialize(plat);

   db.StringSet(plat.Id, serialPlat);
  }

  public Platform? GetPlatformById(string id)
  {
   var db = _redis.GetDatabase();

   var plat = db.StringGet(id);

   if (!string.IsNullOrEmpty(plat))
   {
    return JsonSerializer.Deserialize<Platform>(plat);
   }

   return null;
  }

  public IEnumerable<Platform?>? GetAllPlatforms()
  {
   throw new NotImplementedException();
  }
 }
}

Let’s take a look at the interesting code sections below:

Repo 1st Cut

 Here we state that our class intends to implement the methods defined in the IPlatformRepo interface.
 We use constructor dependency injection to inject an instance of our IConnectionMultiplexer into our class (remember we registered this in Program.cs). We also define a readonly field _redis that we assign our injected instance to, we will use this in the rest of our class.
 In our CreatePlatform method we get an instance of our Redis DB
 We serialize our platform object (we know it’s not null as we have done this check)
 Finally we call the StringSet method which equates to the Redis set command we used via the redis-cli at the start of the article. For this we use the Id of the Platform as the Key and the serialized object as the Value.

I’m not going to detail the workings of GetPlatformById as its essentially just CreatePlatform in reverse – I’m sure you can work it out! (Also don’t forget to save your code)
Registering IPlatformRepo

Before we move onto our Controller so we can test this code, lets not forget to register our interface and concrete implementation class in our dependency injection container, we do that back over on the Program.cs file as shown below:

builder.Services.AddScoped<IPlatformRepo, RedisPlatformRepo>();

These changes are shown in context below:

Registered Repo

Don’t forget to save your changes…
Creating our Controller

Moving on to creating our controller now, in the existing Controllers folder create a file called PlatformsController.cs, your folder hierarchy should look as follows:

Controller 1st cut

Add the following code to that file, this will give us a controller with 2 endpoints:

using CacheService.Data;
using CacheService.Models;
using Microsoft.AspNetCore.Mvc;

namespace CacheService.Controllers
{
 [Route("api/[controller]")]
 [ApiController]
 public class PlatformsController : ControllerBase
 {
  private readonly IPlatformRepo _repository;

  public PlatformsController(IPlatformRepo repository)
  {
   _repository = repository;
  }

  [HttpGet("{id}", Name="GetPlatformById")]
  public ActionResult<IEnumerable<Platform>> GetPlatformById(string id)
  {
   var platform = _repository.GetPlatformById(id);
   
   if (platform != null)
   {
    return Ok(platform);
   }

   return NotFound();
  }

  [HttpPost]
  public ActionResult <Platform> CreatePlatform(Platform platform)
  {
   _repository.CreatePlatform(platform);

   return CreatedAtRoute(nameof(GetPlatformById), new {Id = platform.Id}, platform);
  }
 }
}

I’ve highlighted the interesting parts of the code below:

Controller Code pt1

 We define the base route for the controller, this equates to: /api/platforms
 We use constructor dependency injection to obtain an instance of our repository
 In addition to specifying the the GetPlatformById endpoint responds to a GET request with an additional id parameter in the route, we also name this endpoint as it this is referenced by our CreatePlatform method when it returns the route of the newly created platform resource
 We are using our platform model as both input and return types for our endpoints. While this will work fine for our example, this is not best practice – see the “Taking a shortcut” box below for more detail
 As per standard REST practice, when we create a resource we return back a HTTP 201 with the resource in the response, and also as a navigate-able route. If this doesn’t make sense to you I cover this more in the testing section next.

Taking a shortcut
Using 'internal representations' of our data structures (aka models) as inputs and return values to 'external endpoints' is not best practice as you are tying an internal representation to an external contract. This makes changing the internals of your code harder, it may also expose too much of our internals, externally. In this case I would usually use a Data Transfer Object to represent our Platform model externally. You can learn more about this in my full .NET API tutorial on YouTube: https://www.youtube.com/watch?v=fmvcAzHpsk8

Save the file before we move onto testing our work.
Testing Part 1

We’re going to do a little bit of manual testing to see how our service operates, (and indeed if it works!), first we must run it up…

At a command prompt “in” the project folder type the following to run up our service:

dotnet run

You should see something similar to this:

Runing Server

The service should be running on both HTTP and HTTPS with a different port for each, I’m going to test with the HTTPS endpoint only (as highlighted above).
Creating a Resource

Open your favorite API client, examples of some free tools are Postman and Insomnia, I’m going to use Insomnia due to its clean, uncluttered interface…. First we’ll call our CreatePlatform endpoint by configuring a request as follows:

POST Request

Points to note are:

 Make sure you configure this request as a HTTP POST
 The route to the endpoint is made up of:
  The Hostname & Port: https://localhost:7284
  The controller route: api/platforms
 The body is of type JSON
 The body contains a simple JSON object specifying the name of our 1st Platform

Execute your request and you should get a 201 Created response along with a JSON representation of our newly created resource:

Resource Created

Looking at the response headers you should also find the route to the newly created resource as is standard practice with REST endpoints:

Route to resource

In the above example you should not that the colon between “platform” and the GUID has been encoded as %3A to make it URL safe.
Returning a Resource

To really prove out that we have successfully placed something in the Redis Database, and that we can retrieve it, we now need to test the GetPlatformById endpoint. Copy the full id from the previous example, in my case that would be:

platform:5607e91c-9883-4d0e-aa1c-2ac878d334b6

Now construct a 2nd , (GET), request, which should look as follows:

Get a resource

 Ensure the GET verb is selected
 Append the full key to the end of the route, this will ensure we hit the GetPlatformById endpoint

Execute your request and you should get the resource returned:

Resource Returned

All in all that looks quite successful!

Before we move on to the 3rd endpoint which should return all our resources, lets’ just digress slightly and take a quick “visual look” at how our data is stored in Redis.
Under the Covers

While the redis-cli tool we used earlier on is perfectly decent, I do like “visual” management tools when it comes to working with databases, as they help me contextualize how data is organized, especially if I’m new to a particular platform. With that in mind I made a small purchase and bought Redis Desktop Manager (RDM) which allows me to do just that, (I believe you can get a free version too – just Google Redis Desktop Manager a look for the GitHub repo link).
Other tools are available
Please note that there are other tools available that allow you to work with Redis in a more visual way, I went with RDM as it seemed to be mentioned a lot and was reasonably priced. I am also not affiliated with the people who produce RDM!

Connecting into our Redis instance and looking at the data via RDM we see the following:

RDM View 1

 Our key is represented here under “db0”
 This is a String data type
 The Value is the complete platform object serialized as JSON (I felt this was the simplest & fastest approach)
 You can see a list of other “databases” a Redis instance such as ours has 16 of these numbered 0-15 with the “0” being the default.

Let’s add another Platform to the Database by running another call to our API, (I won’t repeat or show the steps for that though). Having done that, and refreshing the view in RDM, we see the following:

RDM 2

You can see that RDM has taken the prefix of ‘platform’ and created folder with that name, placing all keys with that prefix in that folder. Important: this is just an RDM feature and not related to any particular functionality in Redis. However as per the official Redis docs, keys often benefit form more meaningful names (refer to the section on Redis Keys).

We’ll return to RDM when we start to look at options for creating our 3rd endpoint….
Application Build Part 2

So how are we going to pull back all our platforms from Redis and complete our final endpoint? In an RDBMS system it would literally be the simplest SQL query, something like:

SELECT * FROM PLATFORMS;

However that assumes we have a table called “platforms”, which in the case of a RDBMS is perfectly reasonable, in the case of Redis, both tables and indeed SQL don’t exist – so how do we do it?
Returning all (String) Keys

One approach I looked at was to use the Redis SCAN command, which is documented here. Use of SCAN allows you to bring back a collection of String-type keys, you can even specify a “match” parameter, e.g. “platform” that would only bring back a collection of Keys that started with platform. This was the closest analogy to the SQL example above I could find, the main issue with this approach was that it was “slow” for large data sets, (at least the way I implemented it!), which negated the use of an in-memory cache like Redis in the first place.
If you're interested...
There is a more information on how to use SCAN form StackExchange here: https://stackexchange.github.io/StackExchange.Redis/KeysScan.html

As far as I could tell when using the String data type this was the only option avaialble to me which seemed sub-optimal.
Other things I tried,

One other approach I looked at was to update the the CreatePlatform method in the RedisPlatformRepo class to additoinally add our Platform to a Redis Set, (you can read about Redis Sets here). This approach meant:

 The existing GetPlatformById method (and endpoint) would continue to work as before using the string datatype
 I could now implement the GetAllPlatforms method by returning the Set of all platform objects – btw when I tried this it was super-fast as you’d expect.

If you’re intersted this was the code I added to the CreatePlatfom method (note this is not the final solution I landed on so feel free not to try it out!)

db.SetAdd($"setplatforms", serialPlat);

In context of the wider method this is where it was placed:

Add Set

If we save and execute another platform create request, (for “Kubernetes”), this would be the result in RDM:

Use of Set

 We now have 3 String Keys (and GetPlatformById will work as before)
 We have 1 new “Set” Key (called “setplatforms”)
 The set contains 1 item

And just to be clear, lets do 1 more platform create request, this is what you would get:

add another item to the set

 We have 4 string keys
 We still only have 1 “Set” key, but it contains 2 items now

The code I implemented for GetAllPlatforms in the RedisPlatformRepo class was, (again I don’t expect you to implement this):

public IEnumerable<Platform?>? GetAllPlatforms()
{
 var db = _redis.GetDatabase();

 var completeSet = db.SetMembers("setplatforms");

 if (completeSet.Length > 0)
 {
   var obj = Array.ConvertAll(completeSet, val => JsonSerializer.Deserialize<Platform>(val)).ToList();
   return obj;
 }

 return null;
}

If you do want to have a go at implementing this, you’ll also need to implement the Controller code which we’ll be doing in the next section. As mentioned this was super fast.
Why didn’t I go with this approach

This kind of worked, but there are obvious flaws… The Sting-based platforms could potentially be out of sync with the Set-based platforms. In this example we had 4 “String” platforms, and only 2 “Set” platforms. Yes you could reset and start again, and chances are both approaches would probably remain in sync, however I still did not like this approach.

Note: I didn’t spend any time in looking at how I could roll the 2 methods of inserting platforms in to a “transaction”, meaning that either both were guaranteed to succeed, or both were guaranteed to fail, thus keeping the 2 approaches in sync.

Additionally, while maybe less of a consideration given the abundance of storage these days, this approach also doubled up on storing the same data…
Use of Lists
I also implemented this exact same approach using Redis Lists, it worked in a similar way, but had the same down slides....
Introducing the Hash

So to clarify the acceptance criteria for my required solution (if I were to adopt Redis as my primary db):

 Returning data needs to be fast
 I only want to store data using 1 data-type

With these criteria defined, I realized that a re-write of the RedisPlatformRepo class was required and that I needed to use a different datatype… Enter the Hash.

Again, Redis have done a great job of documenting their data-types here, so I really recommend you take a quick read of that. To summarize what a hash is though:

 Based on the storage on Field / Value pairs (along with the Key for the Hash it’s self)
 Suitable for storing “objects”
 You can get individual Values (based on the Key/Field combination)
 You can get “all” items in the hash.

I think at this point we move back to the code, and take a look at the re-written RedisPlatformRepo class, (I have commented out the previous string data-type implementation for reference and comparison):

using System.Text.Json;
using CacheService.Models;
using StackExchange.Redis;

namespace CacheService.Data
{
 public class RedisPlatformRepo : IPlatformRepo
 {
  private readonly IConnectionMultiplexer _redis;

  public RedisPlatformRepo(IConnectionMultiplexer redis)
  {
   _redis = redis;
  }

  public void CreatePlatform(Platform plat)
  {
   if (plat == null)
   {
    throw new ArgumentOutOfRangeException(nameof(plat));
   }

   var db = _redis.GetDatabase();

   var serialPlat = JsonSerializer.Serialize(plat);

   //db.StringSet(plat.Id, serialPlat);
   db.HashSet($"hashplatform", new HashEntry[] {new HashEntry(plat.Id, serialPlat)});
  }

  public Platform? GetPlatformById(string id)
  {
   var db = _redis.GetDatabase();

   //var plat = db.StringGet(id);

   var plat = db.HashGet("hashplatform", id);

   if (!string.IsNullOrEmpty(plat))
   {
    return JsonSerializer.Deserialize<Platform>(plat);
   }
   return null;
  }

  public IEnumerable<Platform?>? GetAllPlatforms()
  {
   var db = _redis.GetDatabase();

   var completeSet = db.HashGetAll("hashplatform");
   
   if (completeSet.Length > 0)
   {
    var obj = Array.ConvertAll(completeSet, val => JsonSerializer.Deserialize<Platform>(val.Value)).ToList();
    return obj;   
   }
   
   return null;
  }
 }
}

I’ve highlighted the relevant code below:

New Repo

 We create a new Hash with a Key of “hashplatform” and add a new entry with a Field equal to the platform Id, and a Value equal to the entire platform object
 When retrieving an individual platform, we use HashGet and supply the key of the Hash (“hashplatform”) and the value of the Field we want (in this case the platform id).
 The new GetAllPlatforms method makes use of HashGetAll to get all the Values stored in the hash. You can see we just deserialze the Value.

Save the file, before we move on to completing our controller
Finalizing the Controller

The great thing about using the repository pattern is that we don’t need to change the existing implementation of our controller even though we have change the underlying data-types being used. We do however have to create the GetPlatforms Controller Action, see the code for this below:

[HttpGet]
 public ActionResult<IEnumerable<Platform>> GetPlatforms()
 {
   return Ok(_repository.GetAllPlatforms());
}

Arguably the simplest of our controller actions, so much so that I’m not going to explain the code. In context though this is our completed Controller class:

Completed Controller

Save the file, before we move onto some testing.
Testing Part 2

I’m going to “reset” my Redis instance to remove all the other Keys before we try the final solution. You can start deleting keys using the redis-cli tool, but by far the quickest way to achieve this is to destroy our container and restart it.

Back at a comment prompt issue the following command:
Stop Vs Down
Issuing 'docker-compose down' will completely destroy your container and all the data that it holds, so use it wisely! If you simply want to stop your container from running, issue the 'docker-compose stop' command. This just stops the container, so you can restart it at a later date with all your data intact.

`
docker-compose down
`

Then issue the following command again, (this time you need to be in the same folder as the docker-compose.yaml file):

`
docker-compose start -d
`
This will restart a clean instance of Redis.

Back in our project make sure you have saved all your work and if necessary restart the service.
Adding a Platform

Let’s execute a call to our CreatePlatform endpoint as we have done before, this should succeed as before and if we take a look at RDM we see:

Added hash platform

 We have a Hash-type with a Key of “hashplatform”
 It’s of type HASH
 We have 1 entry for our platform
 The Key (or really Field – think this is incorrectly named in RDM) is the Id of the platform
 The Value is the entire JSON object

Before we move on let’s add 2 more, this is what it would look like in RDM:

3 Platforms in our hash

 We now have 3 platforms store in our Hash

Returning a Single Platform

Execute this endpoint exactly as you have done before making sure that you use a valid key value:

Get a single platform

No surprises, this works as before except we are using a different implementation. One other thing just to point out as shown in the screen shot above, is that the response time is insanely fast! (Yes this is running with a tiny data set on a local machine, but I still find it impressive).
Returning all Platforms

Now for the moment of truth, which is to exerecise our newest endpoint. Configure your API client and follows:

Get all Platforms

Then execute the call:

Get all platforms

And there we go, we have completed our API using Redis as our Primary DB!
Final Thoughts

I suppose the main question is would I use Redis as my primary DB in a real work project? The answer (as always!), would be: that depends!

I am still very much vested in the world of the RDBMS, I simply like them, and in many cases that’s how I’d choose to implement my projects. If however I was dealing with very large quantities of data where speed of retrieval was paramount and intricate searching and filtering wasn’t a consideration then I absolutely would consider using Redis as my primary db.

The real take away point here though is that I would now always consider Redis as an option for my primary database, which is not something I would have done before…




🌐 Links 🌐

💾 GitHub Repository: https://github.com/binarythistle/Redi

✏️ Blog: Redis as a Primary DB: https://dotnetplaybook.com/redis-as-a

🤩 Patreon Site (Exclusive Member Benefits!): https://www.patreon.com/binarythistle

🎓 My other courses: https://dotnetplaybook.learnworlds.com

📕 My Book: https://link.springer.com/book/10.100

🌲 Linktree: https://linktr.ee/binarythistle

🔗 Redis Persistence: https://redis.io/topics/persistence

🔗 Redis data types: https://redis.io/topics/data-types-intro

🔗 Nullable Reference Types: https://docs.microsoft.com/en-us/dotn

🔗 Using SCAN: https://redis.io/commands/scan

🔗 Stack Exchange Key Scan: https://stackexchange.github.io/Stack
