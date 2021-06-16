# Mounting an S3 Bucket to an EC2 Instance

## Steps to follow:

### Initial Mounting
1. Launch an EC2 instance (within the Ireland region -> 'eu-west-1')

2. Attach the appropriate IAM role to the instance:
    -  Select the Instance -> 'Actions' -> 'Instance Settings' -> 'Attach/Replace IAM Role'
    - Select and apply 'S3-mount-unsupervised-data' from the drop-down menu under IAM role.

3. Login to your EC2 instance which has been granted IAM role permissions

4. Install the `s3fs` client on your instance by entering the following commands line-by-line:

    **Note:** Upon initial start-up your instance may install some applications in the background, and as such you may need to wait 5 - 10 minutes (or reboot your instance) before proceeding.   

    ```bash
    sudo apt-get install automake autotools-dev fuse g++ git libcurl4-gnutls-dev libfuse-dev libssl-dev libxml2-dev make pkg-config -y

    git clone https://github.com/s3fs-fuse/s3fs-fuse.git

    cd s3fs-fuse/

    ./autogen.sh

    ./configure --prefix=/usr --with-openssl

    make

    sudo make install
    ```
    Verify that the installation was successful by running:
    ```bash
    which s3fs # <--- Should return /usr/bin/s3fs
    ```


5. Navigate to your `home` folder and create a new directory as the mount point for the S3 bucket:

    ```bash
    cd ~/
    mkdir unsupervised_data
    ```

6. Use the s3fs client to mount the bucket to our mount point:

    ```bash
    s3fs -o iam_role="S3-mount-unsupervised-data" -o url="https://s3-eu-west-1.amazonaws.com" -o endpoint=eu-west-1 -o dbglevel=info -o curldbg edsa-2020-unsupervised-predict unsupervised_data
    ```

    Verfy that the mount process is successful by running:

    ```bash
    ls -Rla unsupervised_data/
    ```

    If mounted correctly, you should the contents of a new subfolder, `unsupervised_movie_data`, along with their read/write/execute permissions.

    **Note:** Students will only have read access to this mounted directory, thus they should not try to work within it.  

### Automating the Mount Process

These steps are given for convenience so that the bucket will mount each time the EC2 instance is started.

7. Create a startup script containing our mount command from Step 6:

    ```bash
    echo 's3fs -o iam_role="S3-mount-unsupervised-data" -o url="https://s3-eu-west-1.amazonaws.com" -o endpoint=eu-west-1 -o dbglevel=info -o curldbg edsa-2020-unsupervised-predict unsupervised_data' > mount_unsupervised.sh
    ```

8. Make the script executable:

    ```bash
    sudo chmod +x mount_unsupervised.sh
    ```

9. Configure `cron` to run the script whenever the instance starts up.

    - Run `crontab -e`. Select a text editor you are comfortable with (default 1 is `nano`).
    - Insert the following line at the end of the file:
    ```bash
    @reboot ~/mount_unsupervised.sh
    ```
    - Save the changes (in `nano` type `ctrl + s`), and exit (`ctrl + x`).

10. Reboot your instance to ensure that the S3 bucket has been mounted on startup. If successful, you're done!
