### TezSign as a Service with AMI

`ami` is a utility for portable application management, providing a unified interface and portability for managed apps.

#### Device Setup

For detailed instructions on device setup, please refer to the main README under the [Setup](https://github.com/tez-capital/tezsign/blob/main/readme.md#%EF%B8%8F-setup) section.

#### Setup

> **NOTE:** You do not need to specify the `path` when running `ami` commands if your current working directory (cwd) matches the path you want to run the command against. In this tutorial, we explicitly use the path for clarity.

1.  **Download the host tool**
    Download and install the host app..
    ```bash
     wget -O tezsign https://github.com/tez-capital/tezsign/releases/download/release-202601061517/tezsign-host-linux-amd64 && sudo chmod +x tezsign
     ```
     and move to your /bin directory
    ```bash
    sudo mv ./tezsign /bin/tezsign
    ```
    Connect the board to your host machine (e.g., Radxa Zero 3, RPi Zero 2W). 
    To allow your host machine to communicate with the gadget without root privileges, you need to add a udev rule. Run the helper script (it writes /etc/udev/rules.d/99-tezsign.rules and reloads udev) to install the required rule:
    Download the script:
    ```bash
     wget -O add_udev_rules.sh https://raw.githubusercontent.com/tez-capital/tezsign/main/tools/add_udev_rules.sh && sudo chmod +x add_udev_rules.sh
    ```
    Run the script
    ```bash
    sudo add_udev_rules.sh
    ```
    After running the script, make sure your user is part of the `plugdev` group:
    ```bash
    sudo usermod -aG plugdev $USER
    ```
    You will need to log out and log back in for this group change to take effect.
    
    Verify that it the board shows in the device lists
    ```bash
    tezsign list-devices
    ```
    You will need to unlock to run the signer
    ```bash
    tezsign unlock consensus companion
    ```
When your server reboots, if the device will need to be unplugged and reconnected to reset it's connection, after it appears in list-devices you will need to unlock it to have tezsign sign operations.

2.  **Install AMI**
    `wget -q https://raw.githubusercontent.com/alis-is/ami/master/install.sh -O /tmp/install.sh && sudo sh /tmp/install.sh`

3.  **Create a directory**
    Create a directory outside of the `/home` directory tree. For example:
    `sudo mkdir -p /ami-apps/tezsign` assign the right to your user with `sudo chown -R $USER /ami-apps/tezsign`

4.  **Create the configuration file**
    Create the config filewith the command `nano /ami-apps/tezsign/app.json` and copy the following content in it:

    ```yaml
    {
        configuration: {
            BACKEND: tezsign
            // define SIGNER_ENDPOINT
            //if on separate server:
            // SIGNER_ENDPOINT: 0.0.0.0:20090
            //on same server
            // SIGNER_ENDPOINT: 127.0.0.1:20090
        },
        id: tezsign
        type: {
                id: xtz.signer
                version: latest
        },
        // NOTE: actual tezsign services use <username>_tezsign as its user
        // this is to isolate tezsign into a separate context
        user: <username>
    }
    ```
5.  **Set up the app**
    `sudo ami --path=/ami-apps/tezsign setup`

6. **Initialize and Edit TezSign Configuration**  
    Run the following command to initialize the TezSign configuration:  
    `sudo ami --path=/ami-apps/tezsign setup-tezsign --init --platform`  

    This command generates a configuration file named `tezsign.config.hjson`.  
    You can modify this file as needed. To apply any changes, rerun the `ami --path=/ami-apps/tezsign setup` command.

    > **NOTE:** Avoid overriding the `listen` property in `tezsign.config.hjson`.

7.  **Start the service**
    Start the TezSign service:
    `ami --path=/ami-apps/tezsign start`

8.  **Check status**
    Check the TezSign status:
    `ami --path=/ami-apps/tezsign info`

You can now run `tezsign`-specific commands using:
`ami --path=/ami-apps/tezsign tezsign`

---

#### Enable Automatic Unlock

To enable automatic unlock, set a password by running the following command:  
`ami --path=/ami-apps/tezsign setup-tezsign --password`  

You will be prompted to enter a password. After submitting the password, run:  
`ami --path=/ami-apps/tezsign setup`  

This will apply the changes and enable the automatic unlock feature.

> **NOTE:** To remove the password for automatic unlock, simply submit an empty password when prompted during the `ami --path=/ami-apps/tezsign setup-tezsign --password` command.

---

#### Configuration Change

If you need to modify the configuration:

1.  **Stop the app:**
    `ami --path=<your app path> stop`
2.  **Edit configuration:**
    Change `app.json` or `app.hjson` as required.
3.  **Reconfigure:**
    `ami --path=<your app path> setup --configure`
4.  **Restart the app:**
    `ami --path=<your app path> start`

---

#### Removing the App

To completely remove the application:

1.  **Stop the app:**
    `ami --path=<your app path> stop`
2.  **Remove the app:**
    `ami --path=<your app path> remove --all`

---

#### Troubleshooting

To enable trace-level logging, run `ami` with the `-ll=trace` flag.

For example:
`ami --path=<your app path> -ll=trace setup`

> **Reminder:** Always adjust `<your app path>` according to your app's actual location.