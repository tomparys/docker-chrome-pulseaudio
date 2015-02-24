Docker! Skype! PulseAudio!
==========================

Run Skype inside an isolated [Docker](http://www.docker.io) container on your Linux desktop! See its sights via X11 forwarding! Hear its sounds through the magic of PulseAudio and SSH tunnels!

Known Issue: While audio works flawlessly during calls and Skype is perfectly usable, the notification sounds such as call ringing do not work.

Docker index
============

The easiest method to get Skype in docker is to download the already prepared image from the official Docker image index repository: [tomparys/skype](https://index.docker.io/u/tomparys/skype/). Follow further instructions there.

Building Instructions
=====================

In case you do not want to download the prepared image, you can built the image yourself using these following instructions.

1. Install [PulseAudio Preferences](http://freedesktop.org/software/pulseaudio/paprefs/). Debian/Ubuntu users can do this:

        sudo apt-get install paprefs

2. Launch PulseAudio Preferences, go to the *"Network Server"* tab, and check the *"Enable network access to local sound devices"* and *"Don't require authentication"* checkboxes

3. Restart PulseAudio

        sudo service pulseaudio restart
   
    or
   
        pulseaudio -k
        pulseaudio --start

    On some distributions, it may be necessary to completely restart your computer. You can confirm that the settings have successfully been applied using the `pax11publish` command. You should see something like this (the important part is in bold):

    > Server: {ShortAlphanumericString}unix:/run/user/1000/pulse/native **tcp:YourHostname:4713 tcp6:YourHostname:4713**
    
    > Cookie: ReallyLongAlphanumericString

4. [Install Docker](http://docs.docker.io/en/latest/installation/) if you haven't already

5. Clone this repository and get right in there

        git clone https://github.com/tomparys/docker-skype-pulseaudio.git && cd docker-skype-pulseaudio

6. Build the container

        sudo docker build -t skype .

7. Create an entry in your .ssh/config file for easy access. It should look like this:
        
        Host docker-skype
          User      docker
          Port      55555
          HostName  127.0.0.1
          RemoteForward 64713 localhost:4713
          ForwardX11 yes
          
    (Optional) I recommend creating an SSH key without a password to be used to connect to this container.
    So in case you used a non-standard filename for your SSH key, add this:
   
          IdentityFile /path/to/your/ssh/key

8. Run the container and forward the appropriate port

        sudo docker run -d -p 55555:22 skype

9. (Optional) Copy an SSH public key

    If you plan to use an SSH key, copy the public key to the docker container using the following command. The password is `docker`.

        ssh-copy-id -i /path/to/your/public/key.pub docker-skype

10. Connect via SSH and launch Skype using the provided PulseAudio wrapper script

        ssh docker-skype skype-pulseaudio
     
    In case you didn't copy the SSH public key, the password is `docker`.

11. Go use Skype in a safe container!


Frequently Asked Questions
==========================

Why would I want to do this?
----------------------------
There are a couple of reasons you might want to restrict Skype's access to your computer:

* It is proprietary Microsoft software
* The skype binary is disguised against decompiling, so nobody is (still) able to reproduce what it really does.
* It produces encrypted traffic even when you are not actively using Skype.

How make Skype settings persistent?
-----------------------------------
In order to make Skype retain settings, docker container needs to have a volume attached.

* Create a directory which will be attached to a container, eg. ```mkdir ~/.docker-skype```.
* Run docker container with extra parameter for attaching volume ```sudo docker run -d -v ~/.docker-skype:/home/docker -p 55555:22 tomparys/skype```
* You now have persistent Skype docker container.
* If you want your HOME dir to be attached to container then you dont need creating anything just link your whole HOME dir to a container. Benefit is that you will be able to send all files in your HOME.

How to view received files?
---------------------------
Access the dir which is created in persistence answer and all received files are stored there.

How to use webcam?
------------------
There is pull request for this but original maintainer does not respond for quite some time. PR is issued by https://github.com/giavjeko/docker-skype-pulseaudio .

* Video device needs to be mounted on docker container, ```/dev/video0``` is owned by root ```chmod 777``` (not so smart but fast solution for testing) the file will allow docker user to use webcam.

* Just start docker with sudo docker run -d --privileged -v ~/.docker-skype:/home/docker -v /dev/video0:/dev/video0 -p 55555:22 tomparys/skype
