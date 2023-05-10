# Auto-GPT on Docker with Web Access

# Update:

Unfortunately, the Auto-GPT project is moving so fast, with so many changes every day, that I can't keep up with this. 

The changes being made aren't small, they are major, e.g.:
* restructuring directories
* moving file locations
* rewriting entire sections
* adding selenium to open a desktop web browser (this one is a bit nutty since not everyone would even have a desktop os running it)

Which means I would have to rewrite my Dockerfile every time.

If someone wants to take this project over, please let me know since having this super simple way of spinning up a container is way more efficient than checking out their repo, setting up pip, etc.



# Old:


This repository provides a convenient and secure solution to run Auto-GPT in a Docker container with a web-based terminal. Running Auto-GPT in a Docker container isolates it from the host system, preventing accidental damage from commands like `rm -rf` or `apt install <whatever>`. Additionally, it ensures a consistent and easy-to-maintain environment.

## Features

- Runs Auto-GPT in a Docker container for improved security and maintainability
- Provides a browser-based terminal UI using [`gotty`](https://github.com/sorenisanerd/gotty)
- Accessible via `http://127.0.0.1:8080`, or by IP address

## Installation and Running

1. Clone the repository and navigate to the project directory
2. Copy the sample AI settings file and edit to suit your needs.

    ```
    cp ai_settings.sample ai_settings.yaml
    ```

3. Copy the sample .env file and edit to suit your needs, you can read about these settings on the [auto-gpt repo](https://github.com/Significant-Gravitas/Auto-GPT).

    ```
    cp .env.template .env
    ```

4. Run the Docker container:

    ```
    docker-compose up -d
    ```


## Accessing the Terminal

To access the terminal UI via a browser, visit `http://127.0.0.1:8080` or the IP address of the system it's running on. 

## Kubernetes

The whole thing can also run in a kubernetes cluster.

First create the secrets:
Remember to base_encode the secrets before putting them in (in secret.yaml)
```commandline
echo -n 'some_secret_i_have' | base64
```

Omit the secrets from the .env and then run to have the variables loaded as a configmap and loaded as environment variables into the pods:
```commandline
kubectl create configmap app-configs --from-env-file=.env
```

Then apply these files:
```commandline
kubectl create -f app-autogpt-docker-kubernetes-default-networkpolicy.yaml,app-configmap.yaml,app-deployment.yaml,app-persistentvolumeclaim.yaml,app-service.yaml,app-secret.yaml
```

Find the internal IP of the pod
```commandline
kubectl get nodes -o=wide
```

Access the lovely thing
http://10.133.120.231:31000/ (using the internal address, you must be inside a VPC or the like to access. Else you could make it public by chaning the service to a load balancer .... I wouldn't recommend it though)


Useful commands
```commandline
docker build -t autogpt .
doctl kubernetes cluster create torben-it-kubcluster --auto-upgrade=true --region ams3 --node-pool "name=torben-it-pool-1;count=2;auto-scale=true;min-nodes=1;max-nodes=3"
kubectl expose deployment app --port=8080 --target-port=8080 --name=app --type=LoadBalancer
kubectl exec --stdin --tty shell-demo -- /bin/bash
docker tag autogpt registry.digitalocean.com/torben-it-registry/autogpt
docker push registry.digitalocean.com/<your-registry-name>/my-python-app
```

### tvup/Auto-GPT:stable has changed - how to update
```commandline
cd docker/autogpt
docker build -t autogpt . --no-cache
docker tag autogpt registry.digitalocean.com/torben-it-registry/autogpt
docker push registry.digitalocean.com/torben-it-registry/autogpt
cd ../../
kubectl apply -f app-autogpt-docker-kubernetes-default-networkpolicy.yaml,app-configmap.yaml,app-deployment.yaml,app-persistentvolumeclaim.yaml,app-service.yaml,app-secret.yaml
```

## TODO

- Check if `gotty` can pass audio from the autogpt `--speak` option
- Verify that integration with ElevenLabs is functional (should be okay, but untested)

![Obligatory Screenshot](screenshots/screenshot.png)

