## Setup Python and Code

```bash
sudo apt-get install python3-venv
pip install wheel
git clone https://github.com/pulumi/examples.git
cd examples/gcp-py-network-component/
python3 -m venv venv
source venv/bin/activate
pip3 install -r requirements.txt
```

## Create a Pulumi stack

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

## Cleanup

```bash
pulumi destroy
```

```bash
pulumi stack rm
```
