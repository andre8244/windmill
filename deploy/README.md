# WindMill

## Requirements
- packer: [installation](https://www.packer.io/docs/install/index.html)
- ansible: [installation]()

## Configuration
Before launch packer to provision the machine, edit the following files:
- `roles/firewall/defaults/main.yml`
    ```yaml
    ---
    tcp_accessbyip_v4:
        "0.0.0.0/0":
            - "22"
    ```
    change `0.0.0.0/0` and `22` with your company/home/office public IPs and SSH port to restric access only from certain ips.

- `roles/windmill/defaults/main.yml`
    ```yaml
    ---
    machine_hostname:
        "example.com"
    admin_email:
        "admin@example.com"
    mariadb_root_password:
        "YourMariaDBPassWordHere"
    keyholder_passphrase:
        "YourKeyHolderPassPhrase"
    ca_name:
        "EasyWindmill"
    ca_country:
        "SP"
    ca_province:
        "MA"
    ca_city:
        "Mancha"
    ca_org:
        "WindmillCompany"
    ca_ou:
        "Windmill Office"
    ```
    change `example.com` with the bastion host's hostname

    change `admin@example.com` with the a valid email address (used both for let's encrypt and CA authority)

    change `YourMariaDBPassWordHere` with your MariaDB root password

    change `YourKeyHolderPassPhrase` with your secure passphrase to encrypt public ssh key of support ssh-agent

    change `EasyWindmill` with CA authority name

    change `SP` with CA authority country

    change `MA` with CA authority province

    change `Mancha` with CA authority city

    change `WindmillCompany` with CA authority organization

    change `Windmill Office` with CA authority OU

## Build
Launch packer to provision the machine. There are two different building options:
- DigitalOcean

    ```bash
    DIGITALOCEAN_API_TOKEN=<your_do_api_token> packer build -only=do-centos packer.json
    ```

- Generic Machine
    ```bash
    SSH_HOST=<hostname_of_machine> \
    SSH_USERNAME=<username_to_access_machine> \
    SSH_PASSWORD=<password_of_user> \
    packer build -only=null-machine packer.json
    ```

:warning: Provision scripts currently support only CentOS 7 OS

## Check configuration
After provisioning, check if services is correctly configured
- `systemctl status mariadb` (for mariadb database)
- `systemctl status windmill@windmill` (for the openvpn server default conf)
- `systemctl status windmill@windmill-https` (for the openvpn server with https conf)
- `systemctl status caddy-windmill` (for the caddy web server)
- `systemctl status ronzinante` (for the ronzinante REST API server)
- `systemctl status keyholder-agent` (for the keyholder ssh agent)
- `systemctl status keyholder-proxy` (for the keyholder ssh proxy)

## Add operators
To create new operators and grant access to windmill bastion host, use the command below:

- `windmill-add-operator <user> <path_of_public_ssh_key_of_user>` (work only for user `root`)

*Example*: Add user `edoardo` as operator and grant login to bastion host with his public ssh key
```bash
# copy edoardo's ssh key
echo "ssh-rsa AAAA...XxYyZz edoardo@edoardo-PC" > edoardo-id_rsa.pub

# create operator with public ssh key
windmill-add-operator edoardo edoardo-id_rsa.pub
```
