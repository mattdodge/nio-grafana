# nio and Grafana

A package containing a nio instance, Elasticsearch database, and Grafana dashboard that you can use to visualize your nio data. Check out the blog post detailing these steps in far more detail over on [the niolabs blog](https://niolabs.com/blog/visualize-your-data-with-grafana).

## Requirements
1. [docker-compose](https://docs.docker.com/compose/install/)
2. [nio binary](https://docs.n.io/installation/nio/)
3. An [OpenWeatherMap API Key](https://openweathermap.org/appid)

## Quick Run

Want to skip the docs and get to the good stuff? Copy `nio-project/variables.conf.template` to `nio-project/variables.conf`, fill out the values in that file, and run this from the root of this project
```bash
docker-compose up -d
pip install -r nio-project/requirements.txt
niod -r nio-project -s nio.conf -s variables.conf
```

## Detailed Run Steps

You will need to run three servers to make this project work.

1. Elasticsearch database
2. Grafana server
3. nio instance

### Elasticsearch and Grafana

The easiest way to run the first two servers is via docker-compose:
```bash
docker-compose up
```

This will spin up two docker containers, one for each server. In addition to running the servers, it will populate Grafana with a pre-built [data source](http://docs.grafana.org/features/datasources/) to connect to the newly created Elasticsearch host. It will also persist the relevant data for each server in `es-data/` and `grafana-data` folders, which you should now see in your project folder.

You should be able to access your ES data at http://localhost:9200 and your Grafana instance at http://localhost:3000 (admin/admin default user/pass).

### nio

Now it's time to run the nio instance that will populate the Elasticsearch database. We'll switch into the `nio-project` folder for these steps and install our Python package dependencies.
```bash
cd nio-project
pip install -r requirements.txt
```

We need to tell nio some information about what we want to look up. We'll use a file called `variables.conf` to store that information. There's a template you can use to start with.
```bash
cp variables.conf.template variables.conf
vim variables.conf
```

Grab your Pubkeeper host and token from the [System Designer](https://app.n.io) for your particular nio system. You will also need an [OpenWeatherMap API Key](https://openweathermap.org/appid) and you'll want to enter the zip code you want to retrieve weather information from too.

Once you've filled out the variables, run nio and pass it both the `nio.conf` and our `variables.conf` files as configuration files.
```bash
niod -s nio.conf -s variables.conf
```

If it all works you should see log messages such as the following:
```
NIO [INFO] [main.WebServer] Server 0.0.0.0:8181 started on 0.0.0.0:8181
```
and
```
NIO [INFO] [pubkeeper.protocol.v1] Authenticated to Pubkeeper
```

You can access your newly created local instance from the System Designer with host `localhost` and port `8181`. It has a service that populates the ES database automatically.


## Troubleshooting

### No module found/Block not found errors
The blocks for the nio project are loaded as submodules, make sure that all submodules got cloned as well by running
```bash
git submodule update --init --recursive
```

Also make sure that your Python dependencies got installed
```bash
pip install -r requirements.txt
```

### Pubkeeper Connection Issues
If you see log messages/warnings indicating Pubkeeper connection problems it may be ok. As long as you see `Authenticated to Pubkeeper` at some point you are probably good. The other warnings are caused by publishers trying to send messages before the PK connection is established. To make the Pubkeeper client behave more synchronously add `async=False` to the `[pubkeeper_client]` section of your `nio.conf` file.

If you still see connection errors, make sure your `PK_HOST` and `PK_TOKEN` variables are set correctly in `variables.conf`. Also make sure you're including both files when connecting (`niod -s nio.conf -s variables.conf` - the order matters!)

### Dashboards are Empty
Make sure data is getting in to your Elasticsearch database first. You can hit this URL from your browser to see if there are records: [http://localhost:9200/nio/_search](http://localhost:9200/nio/_search). Make sure there are at least a few data points to populate the graphs with.

If there is data there then check the Grafana dashboard or the Grafana docker container logs to see if there are any errors reported:
```bash
docker-compose logs grafana
```
