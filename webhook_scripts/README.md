# Publish the BTicino C100X/C300X commands to a Webhook

The goal of this guide is to export to a webhook the commands managed by the BTicino C100X/C300X video door entry unit.

## Files explanation

### TcpDump2Webhook

This is the main script that checks every 10 minutes:
* that the script for publishing commands are active and alternatively executes them.
* if the gateway to which the video door entry unit is connected can be reached. The management of the polling to the gateway has been implemented because if the connection with the Wifi is lost, the **StartWebhostSend** script would no longer work. If the gateway is not reachable, the currently active scripts are killed and then run again when the connection is restored. In my specific case, I turn off the WiFi in the evening to reactivate it in the morning, and thus I have solved in this way the problem of blocking the scripts on disconnection.

### TcpDump2Webhook.conf

Contains the following configuration parameters for the main script operation:

* **WEBHOOK_HOST**: Address to send data to.

### StartWebhookSend

This script listens to the network traffic of the video door entry unit (commands from the app, from the internal unit, from the external unit), filters the packets and extracts the commands, sending them to the Webhook through the host defined in the WEBHOST_HOST variable of the main script TcpDump2Webhost.

### TcpDump2Webhost.sh

This script is only used to launch the main TcpDump2Webhost script in background mode. A symbolic link to this file is created in the /etc/rc5.d folder to automatically start on boot.

## Insertion of scripts in the video door entry unit

**NOTE**: In order to insert the scripts in the video door entry unit, SSH must be enabled according to the guide of [@fquinto](https://github.com/fquinto/) [https://github.com/fquinto/bticinoClasse300x](https://github.com/fquinto/bticinoClasse300x)

1. Download files described above to a folder on your PC:
	* TcpDump2Webhost.sh
	* TcpDump2Webhost.conf
	* TcpDump2Webhost
	* StartWebhostSend

2. Open the __TcpDump2Webhook.conf__ file with an editor and modify at least the parameter **WEBHOOK_HOST** with the addresse of your destination WEBHOOK.

3. Transfer the files from the PC to the video door entry unit.
  The files will be transferred to the /tmp folder of the video door entry unit as the rest of the filesystem is read-only. For the transfer you can use the scp command present in linux and windows 10 (from a "Command Prompt" window).

    **Transfer of files to the video door entry unit**

    ```sh
    scp TcpDump2Webhook root2@<intercom_ip>:/tmp/TcpDump2Webhook
    scp TcpDump2Webhook.conf root2@<intercom_ip>:/tmp/TcpDump2Webhook.conf
    scp TcpDump2Webhook.sh root2@<intercom_ip>:/tmp/TcpDump2Webhook.sh
    scp StartWebhookSend root2@<intercom_ip>:/tmp/StartWebhookSend
    ```

    **NOTE**: <intercom_ip> is the IP address of the video intercom door entry unit.

    When prompted for a password, enter the one you defined in the SSH enabling procedure.

4. Once the 6 files have been transferred, connect via the terminal to the video door entry unit and execute the following commands

    ```sh
    # Move to the /tmp folder where we transferred the files.
    cd /tmp

    # Change the permissions of the scripts to make them executable.
    chmod 755 TcpDump2Webhook TcpDump2Webhook.sh StartWebhookSend

    # Make the filesystem writable.
    mount -oremount, rw /
    
    # Create the destination directory and transfer the files there.
    mkdir /etc/tcpdump2webhook    
    cp ./TcpDump2Webhook /etc/tcpdump2webhook
    cp ./TcpDump2Webhook.conf /etc/tcpdump2webhook
    cp ./TcpDump2Webhook.sh /etc/tcpdump2webhook
    cp ./StartWebhookSend /etc/tcpdump2webhook

    # Move to the /etc/rc5.d folder.
    cd /etc/rc5.d

    # Create the symbolic link for autorun on startup.
    ln -s ../tcpdump2webhook/TcpDump2Webhook.sh S99TcpDump2Webhook

    # Modify flexisipsh service to create a file in /tmp/ folder when flexisip restart
    cp /etc/init.d/flexisipsh /etc/init.d/flexisipsh_bak
    sed -i '25i\\t/bin/touch /tmp/flexisip_wh_restarted' /etc/init.d/flexisipsh

    # Make the filesystem read-only again.
    mount -oremount, ro /

    # Restart the video door entry unit.
    reboot
    ```

## Author

[@byte4c](https://github.com/byte4c)

Based on mqtt scripts from Telegram user: "Cico ScS" with some improvements of [@fquinto](https://github.com/fquinto/) and [@gzone](https://github.com/gzone156)
