# Raspberry Pi 4 Media Server
### Author: Rod Vincent Maitim (rvmaitim@gmail.com / www.rvmaitim.me)
----

This project makes use of Raspberry Pi 4's latest specs (Gigabit Ethernet, USB 3.0 Bus) to create a low-powered, low maintenance, and ridiculously cheap media server.

Ever wanted to have a stack of apps that automatically searches the web for your latest TV show / Anime episode, latest HD movie release, and stream it to your media players like phones, laptops, TV Boxes, etc? Well these set of apps can crawl on the net, download your movies/episodes as soon as it is released so you can play it anywhere on your devices. No more waiting and manual downloads. It's like Netflix, but with a total control of your libraries and files.


## <b> Tools: </b>  
* Raspberry Pi 4 (preferrably 8GB RAM model).
* USB C Charger (of course)
* 32GB SD Card
* External Hard drive (to place your media files)



## <b> Contents: </b>
* Raspberry Pi Initial Configuration
* OpenMediaVault Installation and Configuration 
* Docker Installation
* Docker-Compose Installation
* Media Server Apps / Stack Deployment
    * <b> Transmission: </b> A torrent downloader, which has an awesome web UI.

    * <b> Jackett: </b> A centralized torrent indexer, which scouts the torrent sites and passes the download magnet url to the your downloader
    * <b> Plex Media Server: </b> The heart of your media center. This enables you to serve videos, music, podcasts, etc to your devices with a Plex app installed. Think of it like a personal Netflix server.
    * <b> Sonarr: </b> A TV Series manager, once you add your preferred series, it automatically searches the net through your indexer (Jackett) and sends it to the downloader (Transmission).  
    * <b> Radarr: </b> Like, Sonarr, but for movies. 
    * <b> LazyLibrarian: </b> Hosts your ebook, comics, magazines, and manga collection. It also acts like Sonarr and Radarr that can search and download your preferred books. 

* App Configurations 

## <b> Install </b>

### Raspberry Pi Initial Configuration: 

1. As with any new Pi, grab your SD card and install this OS with a Raspberry Pi Imager

    Instructions: https://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2020-05-28/2020-05-27-raspios-buster-arm64.zip): 

    OS file: 
    https://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2020-05-28/2020-05-27-raspios-buster-arm64.zip

2. Once Raspberry Pi OS is installed, create an empty file named 'ssh' on the SD card so you can connect and configure your file without using a monitor. If you're on Windows (which you probably are, you can use the windows CMD, open and direct your terminal to your SD card folder, and enter this command: (change E to your SD card drive letter)
    
    `type NUL >> E:\ssh`

    *  Optional: If you're on wifi, and cannot connect a raspberry pi directly to an ethernet port of your router, you can follow this step: 


        * Create a file in the SD card named: wpa_supplicant.conf then paste the following code (adjusting country code, network/SSID name and wifi password):

            ```
            country=PH
            ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
            update_config=1

            network={
                ssid="FreeWifiKuno"
                psk="whyTheHellWouldIEnterMyPassword"
            }
            ```

3. Now that you're set, connect the SD card to the Raspberry Pi, connect your  power source and wait until it boots up. It should be up within 5 minutes and already has an IP. Open your CMD/Terminal, and run this command to check if there's any new IP on your network: `fping -g 192.168.0.0/24`. Once you have the IP, ssh into it by `ssh pi@<ip address of your pi>`. Password should `raspberry`. 

4. Congrats! You've configured a headless RPi! Now for starters, you can follow this guide https://www.raspberrypi.org/documentation/configuration/raspi-config.md to change the very insecure password, hostname, and such. Once done, you can issue these commands to update your Pi. 
    ```
    sudo apt update
    sudo apt upgrade
    ```

### <b> OpenMediaVault Installation and Configuration. </b>

Now that you're all set with the Pi, you can now turn it into a NAS first. A NAS is a Network Attached Storage, which enables you to share your folders inside the Pi to your home network. While optional, this is very helpful so you can manage your media folders and move files inside your drives without wrangling around the Pi via SSH.

1. To install OMV, SSH into your Pi again, and enter the following command:
    `wget -O - https://raw.githubusercontent.com/OpenMediaVault-Plugin-Developers/installScript/master/install | sudo bash` 

2. Once the script finishes, reboot your Pi via `sudo reboot`
3. After rebooting, access the GUI on your browser with the address `http://<ip of your pi>` (eg. http://192.168.1.5). Once done, you should be able to set the admin user/password. If prompted with a default login, the default admin/pass credentials should be: `admin / openmediavault`

4. Now's a pretty good time to connect your external HDD on the USB 3.0 Ports of the Raspberry Pi. And once you're inside the interface, click on Disks and see if your HDD is detected. You can proceed to wiping it out, and clicking into `File Systems` to create a EXT4 file system on your drive. Afterwards, click `Mount`. You can skip out on wiping it, though as best practice, you should really start with a blank device.

5. Once mounted, share your whole drive (or a specific directory) by clicking `Shared Folders` and setting these parameters. Don't forget to save!:
    * `Name: <Your Preferered Name>`
    * `Device: <Your Disk>`
    * `Path: <Path of the folder on your disk>`
    * `Permission: <Your Preferered Permission, though it's usually 'Admin R/W; User R/W, Others R'>`

6. Now configure SMB by clicking into `SMB`, and on the `Settings` tab, clicking Enable. After enabling, click on the `Shares` Tab > `Add` and follow the configurations I made here: https://imgur.com/a/wL8Xola

7. Click on `Users` > `Add` and set the following: https://imgur.com/aZcdj0j



### Docker and Docker Compose Installation:

Once OMV is now configured, head back to the Pi via SSH, and paste the following command:

`curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh`


`sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && sudo chmod +x /usr/local/bin/docker-compose`


### Running our Docker-Compose file to install the Apps:


1. Create a directory `mediaserver` on your home folder via `mkdir ~/mediaserver`, then download the docker-compose file here via curl:

    `curl https://raw.githubusercontent.com/rvincemt/rpi-mediaserver/master/docker-compose.yml -o ~/mediaserver/docker-compose.yml`

2. Check your /srv/ folder to know your shared drive directory via `ls /srv/`, then we can go ahead and modify the docker-compose file. Now's also the best time to register to Plex, login, and get your claim code at `https://www.plex.tv/claim/`.
Here's a quick script to replace all of that references on the docker-compose file:
    ```
    folderName=$(ls /srv/  | grep dev-disk-by-label)
    transmissionPass="change this to your preferred password"
    claimCode="Replace Your Plex Claim Code Here"

    sed -i "s/<your folder name here>/$folderName/" ~/mediaserver/docker-compose.yml
    sed -i "s/<downloaderpassword>/$transmissionPass ~/mediaserver/docker-compose.yml
    sed -i "s/<claim code>/$claimCode ~/mediaserver/docker-compose.yml
    ```
    Double-check the file via nano/vi on ~/mediaserver/docker-compose.yml before proceeding to the next step.

3. Once done, we can provision it up! Enter the command below:

```
sudo docker-compose -f ~/mediaserver/docker-compose.yml up -d 
```

It will download all the images and create containers, and once done, you can check out the statuses via `sudo docker ps`.

4. Check all UIs if it's up by checking the following addresses in your browser:

    <b>Jackett:</b>
    `http://<IP of your Pi>:9117`

    <b>Sonarr</b>`
    http://<IP of your Pi>:8989`

    <b>Radarr</b>`
    http://<IP of your Pi>:7878`

    <b>LazyLibrarian</b>`
    http://<IP of your Pi>:5299`

    <b>Transmission</b>`
    http://<IP of your Pi>:9091`

    <b>Plex</b>`
    http://<IP of your Pi>:32400`

5. Once they're all up, proceed to configuring the following in this order: Jackett > Sonarr > Radarr > Plex.


## App Configurations:

### Jackett:

* On Jackett UI, start by clicking `Add Indexer` and select your source. Here's mine: (https://imgur.com/yVqH2Xs). Once done, take note of your API Key which is at the top-right side of the site. 

### Sonarr:

* On Sonarr UI, click `Indexer` > click the Plus Icon > `Torznab` > `Custom` and configure: https://imgur.com/yVqH2Xs

* For the Downloader setting, click `Settings` > `Download Client` > click Plus Icon and click `Transmission.` Refer to these settings: https://imgur.com/zMrPd6A
(your password should be the one you configured earlier on the docker-compose file)

### Radarr:

* Radarr setup should be the same with Sonarr as it shares almost the same UI. Setup the Indexer and Download Client settings.

### Plex

Now to the Plex UI (http://IP-Of-Your-Pi:32400), login (you should be registered at this point). 

#### Adding Libraries:
* Click on Settings (top Right), Libraries (left side, scrolldown), then Add Library. 

    * Movies: Library Type: `Movie`, Click `Add Folder` and `Browse`, select `movies` on the left side` (https://imgur.com/mqKtg5F).
    * TV Series: Library Type: `TV`, Click `Add Folder` and `Browse`, select `tv` on the left side` (https://imgur.com/aMgz4Qr).

* After adding these folders, click on `Scan Library Files`. You can also enable auto-scanning of library via `Settings` > check on `Scan my Library Automatically` and check `Scan my library periodically`.




## Adding Media.

* On Sonarr (for TV), click `Series` > `Add Series` and start typing the preferred Series on the searchbar. Once you've searched, refer to these parameters and click the Magnifying Glass Icon: https://imgur.com/lH1EAas

* On Radarr (for Movies), click `Add Movies` > Search for your preferred movie. Refer to these parameters, and click the Magnifying Glass to Add and Search for that movie. https://imgur.com/ST2Dqms


## Verify the Download:

On both Sonarr and Radarr, when it searches and found a matching movie, you can check the `Activity` button on the top-center, and check the queue. If you're not getting any, refer back to your indexer and downloader settings. 

Once the download finishes, it'll be available on Plex.



## Plex on your Devices:

* Download Plex on your devices (Laptop, PC, Tablets, etc and signup)
* Make sure that the libraries are on your Plex app by clicking `More` on your left side of the screen. Refer to the example: https://imgur.com/16ojP5T. 
* If you notice that the download's completed but not showing on your folders, click on the respective folder's option (three dots), and `Scan Library Files`. If it's still not there, you probably have an issue with the folder references.














