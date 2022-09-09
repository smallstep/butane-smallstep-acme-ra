# Butane smallstep ACME RA

This is a repo for launching an step-ca setup as an ACME RA with a Jinja2 template. You can render it easily with [Bupy](https://github.com/quickvm/bupy). Just customize the `butanevars.yaml` file and run Bupy to process it.

```
bupy template smallstep-acme-ra.bu.j2 butanevars.yaml --show
```

You can use this with a free smallstep Certificates Manager account (signup [here](https://smallstep.com/signup?product=cm)!) or with https://github.com/smallstep/certificates.

## Setup

## smallstep Certificate Manager RA

The smallstep [Certificate Manager](https://smallstep.com/signup?product=cm) product offers an automated setup to configure an ACME RA. You can create a new CA to be used with an ACME RA which will walk you through the setup. Just goto `https://smallstep/app/<smallstep team name>/cm/authorities/add?step=0` to start this process. Once you have your CA setup to be used with an ACME RA you will see a RA Token and Password. Copy those as you will need them when you configure the butanevars.yaml file to setup your RA.

### Customize butanevars.yaml

Replace the example SSH key with your SSH key on the `core` user and replace `yourusername` with your POSIX username and replace the example SSH key with your SSH key:

```
passwd:
  users:
  - name: core
    ssh_authorized_keys:
      - ssh-ed25519 AAAAGnNrLXNzaC1lZDI1NTE5QG9wZW5zc2guY29tAAAAIaaaaaaaaaaaaaaaaaaaaaaaaaaa you@example.com
  - name: yourusername
    ssh_authorized_keys:
      - ssh-ed25519 AAAAGnNrLXNzaC1lZDI1NTE5QG9wZW5zc2guY29tAAAAIaaaaaaaaaaaaaaaaaaaaaaaaaaa you@example.com
    groups:
      - wheel
      - sudo
```

Set a `hostname` and `domainname` for your RA. By default Fedora CoreOS will be configured to auto update at 11AM CT on Monday every week. Fedora CoreOS provides a rolling release and is ver stable on upgrades. In most cases, if your deployment can handle a few minutes of downtime on your RA, you can leave auto updates turned on. You can disable auto updates if you like by setting `auto: false`. If you turn off automatic updates please see the [FCOS documentation](https://docs.fedoraproject.org/en-US/fedora-coreos/auto-updates/) on how to manage your updates manually.

```
system:
  hostname: smallstep-acme-ra
  domainname: example.com
  updates:
    auto: true
    time_zone: America/Chicago
    rollout_wariness: 1
    days: '[ "Mon" ]'
    start_time: "11:00"
    length_minutes: "60"
```

The last thing you will need to is add the smallstep CA token and provisioner password to `smallstep_ca_token` and `smallstep_provisioner_password` respectively.

```
stepca:
  smallstep_ca_token: step-acme-ra-token-from-smallstep.co
  smallstep_provisioner_password: mysupersecretpassword
```

**Render the smallstep ACME RA template**

You can view the rendered template by using [Bupy](https://github.com/quickvm/bupy).

```
$ bupy template smallstep-acme-ra.bu.j2 butanevars.yaml --show
    1 variant: fcos
    2 version: 1.4.0
    3 passwd:
    4   users:
    5   - name: core
    6     ssh_authorized_keys:
    7       - sk-ssh-ed25519@openssh.com AAAAGnNrLXNzaC1lZDI1NTE5QG9wZW5zc2guY29tAAAAIaaaaaaaaaaaaaaaaaaaaaaaaaaa example@example.com
    8   - name: example
    9     groups:
   10       - wheel
   11       - sudo
   12     ssh_authorized_keys:
   13       - sk-ssh-ed25519@openssh.com AAAAGnNrLXNzaC1lZDI1NTE5QG9wZW5zc2guY29tAAAAIaaaaaaaaaaaaaaaaaaaaaaaaaaa example@example.com
   14   - name: step
   15     system: true
   16
   17 storage:
   18   directories:
   19   - path: /etc/step
   20     mode: 0700
   21   - path: /etc/step-ca
   22     mode: 0700
   ...snip...
```

and save it to a file:

```
bupy template smallstep-acme-ra.bu.j2 butanevars.yaml --write smallstep-acme-ra.ign
```

### Launch smallstep ACME RA

You can now take the Ignition file you have just created from the Butane template (smallstep-acme-ra.ign in this example) and use that to launch your smallstep ACME RA. Fedora CoreOS supports vast number of cloud providers and even bare metal deployments (even [Raspberry Pi 4](https://docs.fedoraproject.org/en-US/fedora-coreos/provisioning-raspberry-pi4/)!). See the [FCOS documentation](https://docs.fedoraproject.org/en-US/fedora-coreos/) for full details. Here are a few examples:

**Locally on Linux**

```
bupy vm smallstep-acme-ra.bu.j2 --template butanevars.yaml -p 443
```

**AWS**

```
NAME='acme-ra.example.com'
REGION='us-east-2'  # the target region
AMI=$(curl -sSL https://builds.coreos.fedoraproject.org/streams/stable.json | jq --arg region ${REGION} -r '.architectures.x86_64.images.aws.regions[$region].image')
SSHKEY='my-key'     # the name of your SSH key: `aws ec2 describe-key-pairs`
DISK='20'           # the size of the hard disk
TYPE='m5.large'     # the instance type
SUBNET='subnet-xxx' # the subnet: `aws ec2 describe-subnets`
SECURITY_GROUPS='sg-xx' # the security group `aws ec2 describe-security-groups`
USERDATA='smallstep-acme-ra.ign' # path to your Ignition config
aws ec2 run-instances                     \
    --region $REGION                      \
    --image-id $AMI                       \
    --instance-type $TYPE                 \
    --key-name $SSHKEY                    \
    --subnet-id $SUBNET                   \
    --security-group-ids $SECURITY_GROUPS \
    --user-data "file://${USERDATA}"      \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=${NAME}}]" \
    --block-device-mappings "VirtualName=/dev/xvda,DeviceName=/dev/xvda,Ebs={VolumeSize=${DISK}}"
```

**Google Compute**

```
STREAM='stable'
VM_NAME='acme-ra.example.com'
CONFIG='smallstep-acme-ra.ign'
gcloud compute instances create --metadata-from-file "user-data=${CONFIG}" --image-project "fedora-coreos-cloud" --image-family "fedora-coreos-${STREAM}" "${VM_NAME}"
```

## Opensource

You can use this automation without a smallstep [Certificate Manager](https://smallstep.com/signup?product=cm) account and use it with https://github.com/smallstep/certificates. The `butanevars.yaml` file has some commented out variables that you can use to configure the Butane template to not use a token from smallstep Certificate Manager. Just comment out `smallstep_ca_token` and `smallstep_provisioner_password` and uncomment and configure the other variables. See

```
stepca:
  # smallstep_ca_token: step-acme-ra-token-from-smallstep.co
  # smallstep_provisioner_password: mysupersecretpassword
  # Manually configure a step-ca ACME RA
  smallstep_ca_fingerprint: my_smallstep_ca_fingerprint
  smallstep_ca_url: https://myacme.myteam.ca.mystepca.biz
  smallstep_provisioner: acme-ra
  smallstep_ra_address: :443
  smallstep_ra_dns_names: acme.example.com
```

## Getting a certificate with Certbot

```
step ca root --ca-url mycool-ca.example.com --fingerprint my-ca-fingerprint > /tmp/mycool-ca.example.com-root.pem
sudo REQUESTS_CA_BUNDLE=/tmp/mycool-ca.example.com-root.pem ACME_RA_PROVISIONER=my-acme-provisoner ACME_RA_URL=https://mycool-ca.example.com:443 certbot certonly --standalone --preferred-challenges http-01 --work-dir .certbot --logs-dir .certbot --config-dir .certbot --agree-tos --server ${ACME_RA_URL}/acme/{ACME_RA_PROVISIONER}/directory --email you@example.com --domains my-domain.example.com
```

## Getting a certificate with step CLI

```
step ca root --ca-url mycool-ca.example.com --fingerprint my-ca-fingerprint > /tmp/mycool-ca.example.com-root.pem
sudo REQUESTS_CA_BUNDLE=/tmp/mycool-ca.example.com-root.pem ACME_RA_PROVISIONER=my-acme-provisoner ACME_RA_URL=https://mycool-ca.example.com:443 step ca certificate --standalone --acme=${ACME_RA_URL}/acme/{ACME_RA_PROVISIONER}/directory my-domain.example.com example.localhost.crt example.localhost.key --root=${REQUESTS_CA_BUNDLE}
```

## Security

It is important that you do not run your smallstep ACME RA on the public Internet. Any ACME client that can access your RA and go through the ACME workflow using the HTTP challenge will be issued a certificate from your CA. When you launch your ACME RA, please ensure that you use best practices with security groups or firewalls to prevent unwanted access. You can also configure configure your CA with some additional controls for certificate issuance. Plese see the smallstep [documentation](https://smallstep.com/docs/certificate-manager/acme/how-to-use-acme/#certificate-issuance-policies) for more information.

## License

MIT License

Copyright (c) 2022 Smallstep Labs, Inc

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

