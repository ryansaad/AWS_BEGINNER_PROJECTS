# Project 3: EC2 Instance Backup and Restore

This project simulates a core disaster recovery task by creating a backup (an Amazon Machine Image or AMI) of a running server and then launching a new, identical server from that backup.

---

### ## Step 1: Launch the 'Original' Server

First, we need a server with some unique data on it that we want to back up.

1.  Navigate to the **EC2 Dashboard** in the AWS Console and click **"Launch instances"**.
2.  **Name:** Give it a clear name, like `Original-Server`.
3.  **AMI:** Select the `Amazon Linux 2023 AMI`.
4.  **Instance Type:** Choose `t2.micro` (this is Free Tier eligible).
5.  **Key Pair:** Select a key pair that you have the `.pem` file for, as you'll need it to connect.
6.  **Network Settings:** Click "Edit" and ensure "Create security group" is selected.
    * **Security group name:** `Server-SG`.
    * **Inbound rule:** The default `SSH` rule is fine. For the "Source type," you can select `My IP` for better security.
7.  Click **"Launch instance"** and wait for its "Instance state" to become "Running".

---

### ## Step 2: Create a Test File on the Server

This file represents the important data on your server that you need to preserve in the backup.

1.  Select the `Original-Server` in your EC2 list and copy its **Public IPv4 address**.
2.  Connect to the instance using your SSH client (like MobaXterm or Terminal) and your key pair.

    ```bash
    ssh -i "your-key-file.pem" ec2-user@your-public-ip
    ```

3.  Once you are logged in, run the following command to create a simple text file:

    ```bash
    echo "This is my critical data for the backup test." > backup-file.txt
    ```

4.  Verify that the file was created and contains the correct data:

    ```bash
    # List files to see it exists
    ls
    
    # Print the file's contents to the screen
    cat backup-file.txt
    ```

5.  You can now disconnect from the SSH session.

---

### ## Step 3: Create the Backup (Amazon Machine Image)

Now, we'll take a "snapshot" of the server's entire disk in its current state.

1.  Go back to the **EC2 console** in your web browser.
2.  Select the running `Original-Server` instance from your instance list.
3.  Click the **"Actions"** button at the top right.
4.  In the dropdown menu, select **"Image and templates"**, then click **"Create image"**.
5.  On the "Create image" page:
    * **Image name:** Give it a descriptive name, like `My-Web-Server-Backup-July25`.
    * **Image description:** (Optional) Add a note like "Backup of server containing backup-file.txt".
    * **No reboot:** Leave this option **unchecked**. In a real-world scenario, allowing AWS to reboot the instance ensures the file system is consistent and the data is not corrupted.
6.  Click the **"Create image"** button.
7.  AWS will now create the AMI. You can monitor its progress by clicking on **"AMIs"** in the left-hand navigation menu. The status will change from "pending" to **"available"** in a few minutes. You must wait for it to become "available" before proceeding.

---

### ## Step 4: Launch a New 'Restored' Server from the Backup

1.  Once your AMI's status is "available", select it from the AMIs list.
2.  Click the **"Launch instance from AMI"** button.
3.  The launch wizard will start, with your custom AMI already selected. Configure the new instance:
    * **Name:** `Restored-Server`.
    * **Instance Type:** `t2.micro`.
    * **Key Pair:** Select the same key pair you used for the original server.
    * **Network settings:** In the security group section, choose **"Select existing security group"** and select the `Server-SG` you created earlier.
4.  Click **"Launch instance"**.

---

### ## Step 5: Verify the Restore Was Successful ✅

This is the final test to confirm the backup worked.

1.  Wait for the new `Restored-Server` to enter the "Running" state.
2.  Select the `Restored-Server` and copy its new, unique **Public IPv4 address**.
3.  Connect to this new server via SSH, just as you did before, but using the new IP address.
4.  Once you are logged in, check for your file:

    ```bash
    ls
    ```

    You should see `backup-file.txt` listed in the output.
5.  Check the file's contents:

    ```bash
    cat backup-file.txt
    ```

    The output should be "This is my critical data for the backup test."

If the file and its content are there, you have successfully backed up and restored a server.

---

### ## Step 6: Clean Up Your Resources

To avoid any unexpected AWS charges, you must delete all the resources you created for this project.

1.  **Terminate EC2 Instances:** Go to the **Instances** page, select both `Original-Server` and `Restored-Server`, click **"Instance state"**, and choose **"Terminate instance"**.
2.  **Deregister the AMI:** Go to the **AMIs** page, select your `My-Web-Server-Backup-July25` AMI, click **"Actions"**, and choose **"Deregister AMI"**. Confirm the deregistration.
3.  **Delete the Snapshot:** When you create an AMI, AWS creates an EBS Snapshot in the background. You must delete this separately to stop storage costs.
    * In the left menu, under "Elastic Block Store," click on **"Snapshots"**.
    * Find the snapshot that was created by your AMI (it will have the AMI ID in its description).
    * Select the snapshot, click **"Actions"**, and choose **"Delete snapshot"**.
