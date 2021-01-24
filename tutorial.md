# Tutorial on Pulumi in GCP

### Prerequisites

 -  You have a Google Cloud Platform account and a Google Project (note the Google Project Id).

## Install Pulumi

```bash
curl -fsSL https://get.pulumi.com | sh
```

```bash
PATH=$HOME/.pulumi/bin:$PATH
```

## Create a Pulumi Stack

```bash
mkdir webserver && cd webserver
pulumi new gcp-javascript
```

```bash
cat > index.js <<<EOF
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
EOF
```

```bash
pulumi up
```

```bash
pulumi stack
```

## Update

```bash
cat > kk.js <<<EOF

const computeFirewall = new gcp.compute.Firewall("firewall", {
    network: computeNetwork.id,
    allows: [{
        protocol: "tcp",
        ports: [ "22", "80" ], // <-- ADD "80" HERE
    }],
});

// v-- ADD THIS DEFINITION
const startupScript = `#!/bin/bash
echo "Hello, World!" > index.html
nohup python -m SimpleHTTPServer 80 &`;

const computeInstance = new gcp.compute.Instance("instance", {
    machineType: "f1-micro",
    zone: "us-central1-a",
    metadataStartupScript: startupScript, // <-- ADD THIS LINE
    bootDisk: { initializeParams: { image: "debian-cloud/debian-9" } },
    networkInterfaces: [{
        network: network.id,
        // accessConfigus must include a single empty config to request an ephemeral IP
        accessConfigs: [{}],
    }],
});

EOF
```

```bash
pulumi up
```

```bash
curl $(pulumi stack output instanceIP)
```

## Cleanup


