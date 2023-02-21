
## Solved: cannot kill Docker container - permission denied

````
    Maja J	
    Docker	
    Created: 04 December 2020 
````

Just recently I had the following issue while trying to stop my containers:

````
Error response from daemon: Cannot kill container: 9e81a44d4c79: 
Cannot kill container xxxx: unknown error after kill: runc did not terminate sucessfully: 
container_linux.go:392: signaling init process caused "permission denied"
````

The container was started regularly with docker-compose up, and after that, it wasn't reacting to:

````
docker-compose down

docker-compose stop xxx or

docker container stop xxx
````


It turned out that AppArmor service was messing up with Docker. AppArmor (or "Application Armor") is a Linux kernel security module that allows the system administrator to restrict programs' capabilities with per-program profiles. For this problem with containers, it helped me to remove the unknown from AppArmor using the following command:
````
sudo aa-remove-unknown
````

After that, I was able to stop and kill my containers. To kill all running Docker containers, you can use the following command:

````
docker container kill $(docker ps -q)
````

If this didn't work for you, you can remove AppArmor, and then install it afterward if it's needed:

````
sudo apt-get purge --auto-remove apparmor
sudo service docker restart
docker system prune --all --volumes
````
