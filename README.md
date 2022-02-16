# Building a Docker environment with vagrant + mutagen
 
## This repository provides a way to use Docker engine on Mac about x2 faster than Docker For Mac.

# Preface
Docker For Mac has been known for its slow performance.

This problem is due to the overhead of the File I/O API provided by Apple for bi-directional file sharing between different file systems in Mac(APFS) and in Docker engine(basically, ext4)

[For more information on this issue, a Docker staff explains it in their forum. ](https://forums.docker.com/t/file-access-in-mounted-volumes-extremely-slow-cpu-bound/8076/158)

This problem is so deep-rooted that it has not been solved even several years after the release of Docker For Mac, and there is still no solution in sight.

Under the circumstances, [Docker Inc. was changed Docker For Mac as paid subscription for large businesses.]( https://www.docker.com/blog/updating-product-subscriptions/ )

Personally, Docker For Mac is not a product that I would dare to pay for and continue to use, nor is it essential to use Docker For Mac to use Docker engine.


For this reason, I decided to use this Git repository to provide a way to use Docker on a Linux VM by combining Vagrant + Mutagen without going through Docker For Mac during product development.

In this case, the speed bottleneck described above will no longer occur.

Therefore, for projects that require a large amount of file I/O, such as Rails and Laravel, the server can run much faster.

# Installation

1. Open terminal on the Mac host and execute the following command.
```shell
    
brew install virtualbox --cask

brew install vagrant --cask
     
brew install mutagen-io/mutagen/mutagen 

vagrant plugin install vagrant-disksize vagrant-hostsupdater vagrant-mutagen vagrant-docker-compose
```

2. On the Mac terminal, run `sudo vim /etc/hosts` and add the following description to the hosts file.
```shell
 192.168.56.10  host.app.local
```

3. Go to System Preferences/Security and Privacy/General on your Mac and give virtualbox permission to run.
   (Some Mac will ask you to reboot the machine here.)

4. Create a vagrant installation directory and run `vagrant init`.

5. Copy the files under the `vagrant` directory of this repository into the directory where you ran `vagrant init`, and rewrite the BOTH contents of `vagrantfile` and `mutagen.yml` according to the comments in the files. 


6. Open Terminal in that directory and run `vagrant up`. 

If vagrant doesn't start, run the following command in Terminal on your Mac, give oracle permission to run in System Preferences/Security and Privacy, and reboot your machine.
```shell
 sudo kmutil load -b org.virtualbox.kext.VBoxDrv
```

7. After vagrant is started, enter the VM using `vagrant ssh` and execute the following command. 
```shell

 sudo apt update && sudo apt upgrade

```

8. For the project to be migrated to Vagrant development, replace the host descriptions `host.docker.internal` and `localhost` in .env and MySQLWorkbench with the description `host.app.local`. Replace the host description with `host.app.local`.


9. Go to each project directory where the Dockerfile and docker-compose.yml are located in vagrant, and execute the command to start the container.
```shell 
 ex.
 > cd app/laravel-project
 > docker-compose up
```

10. From a browser or MySQLWorkbench on the Mac side, access `http://host.app.local:[port you set]`, and if you can access the service of the Docker container you started, the configuration is complete.

# Notes.

## About Mutagen

 In the acceleration method provided in this repository, the file I/O part, which has been a bottleneck in Docker For Mac, is covered by bi-directional collaboration using mutagen.

 (This solution was previously considered in the preview version of Docker For Mac, but there were many issues with permissions, etc., and it was soon discontinued.)

 Changes on the host side are detected by mutagen and reflected in the VM, resulting in almost the same operability as docker for mac.

 If mutagen is stopped, changes on the host side will not be automatically reflected in the VM.

 You can check the status of mutagen by running `mutagen sync list` on the host side, so if you suspect that your changes are not reflected, please run it accordingly.

 If it is not working, there is a problem with mutagen configuration or permissions.
```bash
 # If mutagen is running
 $ mutagen sync list

 --------------------------------------------------------------------------------
 Name: app
 Identifier: sync_HcRP8vDxZrHyDT25ptpKscrwmizxGOWvy5u0d1ivI0e
 Labels:
 	io.mutagen.project: proj_iT61MqCFGsZb7lhr06qhCT72kDvNTI7XH4jpyJbnwqy
 Alpha: 
	URL: /Users/user/dev/laravel-project
	Connection state: Connected
 Beta:
	URL: host.app.local:/home/vagrant/app
	Connection state: Connected
 Status: Watching for changes
 --------------------------------------------------------------------------------
```

```bash
 # If mutagen is not running
 $ mutagen sync list

 --------------------------------------------------------------------------------
 No sessions found
 --------------------------------------------------------------------------------
```

## About compatibility of Apple Silicone Architecture

 This project is not avairable in Apple Silicone Architecture like Apple M1.

 This is because almost all of the major VMs do not support Apple Silicone.

 You can make them work if you try hard enough, but you will have to choose between using paid software or accepting performance degradation.

 Since all the options have some problems, I do not recommend using this project in  Apple Silicone at this time.

 At this point, the best solution for Apple silicon is to use Docker For Mac as-is.