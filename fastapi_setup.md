## Docker secrets vs Env vars
Docker secrets seem like something valuable to know about but they may be more than what I need [pro-secrets SO post](https://stackoverflow.com/questions/48141233/why-are-docker-secrets-considered-safe)
[This post](https://stackoverflow.com/questions/42729723/should-i-use-user-secrets-or-environment-variables-with-docker) Seems to say that env variables are better, but their 
argument doesn't seem to make a whole lot of sense. They simply link to [this source](https://12factor.net/config) which is great, but doesn't actually support any argument between
secrets and env vars. It simply advocates for env vars.

It seems to me that both secrets and env vars allow for defining sensitive keys, passwords, config vars, etc. in a file that can be loaded into an application
but that helps avoid it being stored in version control. On top of that, Docker secrets (when used with a service or within Docker Swarm) allows for controls on
which processes see which secrets. They also are encrypted on the container whereas env variables could be easily accessed if an attacker gained root access on a container.
Docker secrets in my mind seem like the better option, but also overkill for my purposes especially when getting my dev environment set up is blocking me from motivation
and progress on the actual API...
Note to self when you come back to switch to Docker secrets: you've been running into issues with getting the db_uri to load as a docker secret into the pydantic BaseSettings
subclass (app/core/config.py)

## Configuring testing with FastAPI
FastAPI boasts support for asynchronous handling of requests. This is great because web servers are all about
exchanging IO between the client and server and the server and persistence layer.

Currently, Postgres is my persistence layer. I'm using sqlalchemy to model my data at the db level and pydantic
to model my data within the webserver. Unfortunately, that means there's a bit of redundancy between the db
model classes and the pydantic classes. But I want to figure this all out before I use something like 
SQLModel.

The sqlalchemy ORM does not support truly asynchronous db queries. Thus I'm defining ORM objects, but only
referencing the tables to do core-style queries and not designing any object relationshipts beyond
including foreign keys. 

All of this means that I want my automated tests that exercise my database query logic to be asynchronous.
This is great. However, FastAPI-users uses the ORM and thus the tests (included in the cookie-cutter project)
are synchronous.

I had a difficult time getting my sync and async tests to run (even though they were in different modules)
After installing pytest_asyncio and marking my async tests appropriately, all my tests ran except for the
last test, which happened to be sync. Strangely, it did run when executed in isolation. It only failed 
when I ran the entire test suite.

The error was happening in the teardown of the test. Essentially saying that it was trying to shutdown
an event loop that did not exist. 

After poking around fruitlessly for SO posts, I changed the pytest fixture that provided a FastAPI 
test client in conftest.py to have a 'module' scope instead of a 'session' scope. This fixed the problem.
It's weird that it did, because the final test did not even use that fixture.

My hunch is that my async test was essentially hijacking the event loop of the existing test client (even though
the async test didn't use the client fixture either) and when the async test was over, it closed the event loop
so when the session teardown happened, it also tried to stop the event loop and there was nothing to stop.

I haven't tried it yet, but I'm curious to see what would happen if I just switched it to be an Async Test Client.

At the end of the day, it works now, so I'm not going to fiddle with it any more until I need to.
I have a sneaking suspicion that I'm going to have to restructure all the testing fixtures that came
with the cookie cutter anyway.
