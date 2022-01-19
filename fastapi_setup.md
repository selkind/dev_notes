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

## Configuring RichHandler for logging in Docker-compose hosted python app
So I found [this tutorial](https://blog.hay-kot.dev/fastapi-and-rich-tracebacks-in-development/) to configure RichHandler for colors in my log messages (and potentiall tracebacks too)
I essentially copied and pasted the code (using a pydantic class instead of a dataclass) and adjusting the configuration settings to match my preferences.
However, the stdout stream of docker-compose did not reflect any colorings even though the log format changed and the rich handler was definitely the logging handler.

Turns out the parameter "tty" needs to be set to true in the **docker-compose.yml** file. The backend service (or whatever your service is named that runs fastapi) needs to have a line with *"tty: true"*. This worked immediately.
Additionally, I learned that adding COLUMNS=120 as an environment variable in the .env file is something that the RichHandler will pick up on to increase the width of the logging message.

There is still an issue where the docker compose log format leaves a bunch of space between the container name and the beginning of the log (where time is sometimes inserted). I will see if I can get rid of this.
