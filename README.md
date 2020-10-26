# app-http-logger
Provide out-of-the-box automatic logging of your running docker containers, and make the data available on ElasticSearch + Kibana for further analysis and visualization. 

**Note**: A more modern version of app-http-logger is available on https://github.com/redpencilio/app-http-logger

## Usage 
The app-http-logger consists of multilple components which you can combine at your leisure. To make things a bit easier, we've provided 3 docker-compose files that you can combine as required.
* The main `docker-compose.yml` provides the basic setup to capture network traffic and convert it to har files. By default it will log traffic for any container with the label "logging"
* `docker-compose.visualize.yml` provides the setup for loading, displaying and exploring the traffic in kibana
* `docker-compose.encrypt.yml` provides the setup to automatically encrypt logs after a configured timespan. *NOTE*: make sure to include a public gpg key 

A `.env` file is included in this repository that configures docker-compose to load the basic setup and visualize docker-compose files when you run `docker-compose`. 


### logging traffic and visualizing it
Clone this repository and then run the following commands to start the stack

```
docker volume create --name=pcaps
docker-compose up -d
```

After a few minutes (if traffic has been logged), visit *http://localhost:5601* to start visualizing your data. For a basic setup, click on 'discover' and add the index `hars*`

Be sure to set the `logging` label to any container of which you want to log the traffic.

### logging traffic and encrypting it
Clone this repository and then run the following commands to start the stack. 

```
docker volume create --name=pcaps
docker-compose -f docker-compose.yml -f docker-compose.encrypt.yml up -d
```

Make sure you have a public GPG key available in the `./keys` folder and configure the correct recipient in the `docker-compose.encrypt.yml` file. You can create a gpg key with the following command:

```
gpg --gen-key
```

### visualizing encrypted logs
To visualize encrypted logs from an application, first decrypt the encrypted har files from the `processed_hars` Docker volume.
```
for file in $source_dir/*/*.trans.har.gpg ; do
    mkdir -p "$target_dir/`dirname $file`"
    gpg --decrypt --recipient $recipient --trust-model always --passphrase-file $gpg_key_pwd -o "$target_dir/`dirname $file`/`basename $file ".gpg"`" $file
done
```

Next clone this repository and start the visualization stack.
```
docker-compose -f docker-compose.visualize.yml up -d
```

Copy the decrypted files to the `hars` Docker volume. Visit *http://localhost:5601* to start visualizing your data. For a basic setup, click on 'discover' and add the index `hars*`.

## Components

* [docker-monitor-service](https://github.com/lblod/docker-monitor-service/): keeps track of running containers in the database

* [docker-network-capture-service](https://github.com/lblod/docker-network-capture-service/): captures network traffic of docker containers as listed in the database. Optionally filters on a docker label.

* [mu-har-transformation-service](https://github.com/lblod/mu-har-transformation-service/): converts the captured pcap files to har (a json structure)

* [encryption-service](https://github.com/lblod/encryption-service/): encrypts files

* [har-to-elastic-service](https://github.com/lblod/har-to-elastic-service/): loads har files into elasticsearch 

* [elasticsearch](https://www.docker.elastic.co/): searchengine/database

* [kibana](https://www.docker.elastic.co/): dashboard 

## Known issues
 * tcpdump containers are not removed when logger is stopped, a quick way to remove them is running `docker ps | grep crccheck | awk '{ print $1 }' | xargs docker rm -f`
 * PATCH requests are not logged
 * crashed tcpdump containers are not restarted
 
 
