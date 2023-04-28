# Download and Upload Your Hubble Peer ID to a New Hub

Your Hubble Peer ID is your unique hub identifier. Currently, it requires whitelisting before other hubs will accept the connection. It would also be important in the case that hubs gain reputability and trust. Here's how to find and upload your Hubble Peer ID to a new hub:

## Step 1: Log in to your DigitalOcean Droplet

Open your local terminal and log in to your DigitalOcean droplet using the following command:

`ssh root@(droplet-ip)`

Make sure to replace (droplet-ip) with the IP address of your droplet.

## Step 2: Find Your Peer ID File

Navigate to the .hub directory by running the following command:

```
cd /root/hubble/apps/hubble/.hub
ls
```

This will list all the files in the .hub directory. Your peer ID file will have a name similar to "12D3KooWKXAG1Z212HHrwYaW5NnwBxoXkk3Sr7USzRxMzS9pieZi_id.protobuf". Copy the filename for use in the next step.

## Step 3: Download Your Peer ID File to Your Local Computer

To download your peer ID file to your local computer, run the following command in your local terminal:

```
scp username@(droplet-ip):/root/hubble/apps/hubble/.hub/12D3KooWKXAG1Z212HHrwYaW5NnwBxoXkk3Sr7USzRxMzS9pieZi_id.protobuf .
```

Make sure to replace "username" with your username, (droplet-ip) with the IP address of your droplet, and "12D3KooWKXAG1Z212HHrwYaW5NnwBxoXkk3Sr7USzRxMzS9pieZi_id.protobuf" with the filename you copied in step 2.

## Step 4: Upload Your Peer ID to a New Hub

To upload your peer ID to a new hub, run the following command in your local terminal:

```
scp /path/to/example_id.protobuf username@(droplet-ip):/root/hubble/apps/hubble/.hub/
```

Make sure to replace "/path/to/example_id.protobuf" with the path to your peer ID file on your local computer, "username" with your username, and (droplet-ip) with the IP address of your droplet.

That's it! You have successfully found and uploaded your Hubble Peer ID to a new hub.
