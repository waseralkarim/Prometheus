### **Docker**

Use Docker volumes when you want the Docker Engine to manage the storage volume.

To use Docker volumes for persistent storage, complete the following steps:

1. Create a Docker volume to be used by the Grafana container, giving it a descriptive name (e.g. `grafana-storage`). Run the following command:
    
    ```jsx
    # create a persistent volume for your data
    docker volume create grafana-storage
    
    # verify that the volume was created correctly
    # you should see some JSON output
    docker volume inspect grafana-storage
    ```
    
2. Start the Grafana container by running the following command:
    
    ```jsx
    # start grafana
    docker run -d -p 3000:3000 --name=grafana \
      --volume grafana-storage:/var/lib/grafana \
      grafana/grafana-enterprise
    ```
    

### **Use bind mounts**

If you plan to use directories on your host for the database or configuration when running Grafana in Docker, you must start the container with a user with permission to access and write to the directory you map.

To use bind mounts, run the following command:

```jsx
# create a directory for your data
mkdir data

# start grafana with your user id and using the data directory
docker run -d -p 3000:3000 --name=grafana \
  --user "$(id -u)" \
  --volume "$PWD/data:/var/lib/grafana" \
  grafana/grafana-enterprise
```
