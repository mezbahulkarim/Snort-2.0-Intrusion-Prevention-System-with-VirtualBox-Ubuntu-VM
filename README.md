# Snort-2.0-Intrusion-Prevention-System-with-Ubuntu-VM
This is a guide on how to set up and install Snort as an Intrusion Detection OR Intrusion Prevention System on Ubuntu machines. The installation was done on an Ubuntu Virtual Machine set up using VirtualBox.

**Setting up the Virtual Machine(s) and the basic Network Structure:**

* Install VirtualBox and install an Ubuntu Distro
* In the Oracle VM VirtualBox Manger, change the network settings for that VM. Under the adatpers that you wish to use click 'Advanced' then make sure Promiscuous Mode is set to Allow All. 
* If you want to use your actual machine that runs the VM to interact with the VM then set 'Attached to' 'Bridged adapter'. This way your VM will get an IP address directly from the router your actual machine is connected to. 
* If you want to set up Snort for Intrusion Prevention then you need to install another VM. As your VM needs to have 2 interfaces in order to prevent traffic flowing from one side to another side.
* For the second VM make sure that under network settings it has an Adapter 'Attached' to 'Host-only Adapter'. Do the same for the first VM, which should now have 2 Adapters, one set to 'Bridged adapter' mode and another set to 'Host-only Adapter' mode.
* Now under 'File' in Oracle VM VirtualBox Manager, click on 'Host Network Manager' and Create a new one. This is the network that the second VM is connected to the first VM with. Thus the two interfaces of the first VM are connected to two different subnets, one subnet with the actual machine and the other with the second VM.


**Setting up Snort on the VM:**

* Snort will only need to run on the first VM. 
* First install the package:

    ```bash 
    $ sudo apt-get install snort -y
    ```

* Set the interface to listen on to the desired interface when prompted, same for subnet, which should be in the format X.X.X.X/Y where Y is the subnet mask and X.X.X.X is the IP.
* Now you should have the Snort files in the /etc/Snort directory. 
* Now you can edit the configuration file for Snort:


    ```bash 
    sudo nano /etc/snort/snort.conf
    OR 
    sudo gedit /etc/snort/snort.conf
    ```

* Under Step 7 in the .conf file, you can see the file etc/snort/rules/local.rules mentioned, this is the file where you can input your own rules for Snort.
* If you edit the configuration file for example changing HOME_NET, EXTERNAL_NET or any other ipvar, you can test the .conf file to see if everything is working with the following command, replace enp0s3 with your own interface:
    ```bash
    sudo snort -T -i enp0s3 -c /etc/snort/snort.conf
    ```

* Run the command below to start running Snort, remember to change the interface:
  ```bash
  sudo snort -q -l /var/log/snort -i enp0s3 -A console -c /etc/snort/snort.conf
  ```

* Run the following if you need help understanding the flags or commands. 
    ```bash
    Snort --help
    ```

* Check the local.rules file in this repo to find an example of how to write commands for Snort. The command shown will alert all icmp requets being made in the network to the machine running Snort, thus you can test if Snort is working by pinging the machine.

* In order to use Snort in Intrusion Prevention Mode, some more changes need to be made, skip the rest of these instructions if you want to run Snort in Intrusion Detection Mode only. 

* Set up a default gateway for the second VM to use the interface of the first VM on the same subnet as the second VM. 

* Now we need to run Snort in 'inline' mode which can be done by:
    
    Replacing the lines
       
        # config daq: <type>
        # config daq_dir: <dir>
        # config daq_mode: <mode>
        # config daq_var: <var>


    With the lines:

        config daq: afpacket
        config daq_mode: inline

* Change the local.rules file, add a 'reject' or 'drop' rule. 

* Now run Snort again, remember to change the interface names:
    ```bash
    sudo snort -c /etc/snort/snort.conf -i enp0s3:enp0s8 -Q -A console -q
    ```
* Test if the rules are working by sending traffic from side of the VM to another for example pinging the other interface of the first VM from the second VM. 



