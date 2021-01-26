# Pulumi in GCP Tutorial

### Prerequisites

 -  You have a Google Cloud Platform account and a Google Project (note the Google Project Id).

## Install Pulumi

```bash
curl -fsSL https://get.pulumi.com | sh
```

## Create a Pulumi Stack
```bash
mkdir webserver && cd webserver
pulumi new gcp-javascript
```

When creating the new pulumi stack follow the link to generate a token and paste it in the console.

```js
const gcp = require("@pulumi/gcp");

// Create a network
const network = new gcp.compute.Network("network");
const computeFirewall = new gcp.compute.Firewall("firewall", {
    network: network.id,
    allows: [{
        protocol: "tcp",
        ports: [ "22" ],
    }],
});

// Create a Virtual Machine Instance
const computeInstance = new gcp.compute.Instance("instance", {
    machineType: "f1-micro",
    zone: "us-central1-a",
    bootDisk: { initializeParams: { image: "debian-cloud/debian-9" } },
    networkInterfaces: [{
        network: network.id,
        // accessConfigus must includ a single empty config to request an ephemeral IP
        accessConfigs: [{}],
    }],
});

// Export the name and IP address of the Instance
exports.instanceName = computeInstance.name;
exports.instanceIP = computeInstance.networkInterfaces.apply(ni => ni[0].accessConfigs[0].natIp);
```

```bash
pulumi up
```

```bash
pulumi stack
```

## Update

```js
const gcp = require("@pulumi/gcp");

// Create a network
const network = new gcp.compute.Network("network");
const computeFirewall = new gcp.compute.Firewall("firewall", {
    network: network.id,
    allows: [{
        protocol: "tcp",
        ports: [ "22", "80" ], // <-- ADD "80" HERE
    }],
});

// v-- ADD THIS DEFINITION
const startupScript = `#!/bin/bash
echo "Hello, World!" > index.html
nohup python -m SimpleHTTPServer 80 &`;

// Create a Virtual Machine Instance
const computeInstance = new gcp.compute.Instance("instance", {
    machineType: "f1-micro",
    zone: "us-central1-a",
    metadataStartupScript: startupScript, // <-- ADD THIS LINE
    bootDisk: { initializeParams: { image: "debian-cloud/debian-9" } },
    networkInterfaces: [{
        network: network.id,
        // accessConfigus must includ a single empty config to request an ephemeral IP
        accessConfigs: [{}],
    }],
});

// Export the name and IP address of the Instance
exports.instanceName = computeInstance.name;
exports.instanceIP = computeInstance.networkInterfaces.apply(ni => ni[0].accessConfigs[0].natIp);
```

```bash
pulumi up
```

```bash
curl $(pulumi stack output instanceIP)
```

## Cleanup

```bash
pulumi destroy
```

```bash
pulumi stack rm
```

## Install Python dependencies

```bash
sudo apt-get install python3-venv
pip3 install wheel
```

```bash
git clone https://github.com/pulumi/examples.git
cd examples/gcp-py-network-component/
python3 -m venv venv
source venv/bin/activate
```

```bash
pip3 install -r requirements.txt
```

## Create a Pulumi stack using Python

```bash
pulumi stack init
pulumi config set gcp:project codelab-eitany-prober
pulumi config set gcp:region us-central1
pulumi config set gcp:zone us-central1-b
pulumi config set gcp-py-network-component:subnet_cidr_blocks '{ "172.31.0.0/20":0, "172.32.0.0/20":0 }'
```

```bash
pulumi up
```

```bash
curl $(pulumi stack output nginx_public_ip)
```

### Cleanup

```bash
pulumi destroy
```

```bash
pulumi stack rm
```
