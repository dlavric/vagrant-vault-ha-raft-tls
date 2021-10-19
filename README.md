# vagrant-vault-ha-raft

 Repository that builds High Availability of 3xVault nodes with Raft as the backend storage.

**This repository is just a laboratory environment and it shouldn't be used in production.
The purpose of this repo is learning how to setup TLS for a 3x Vault node with Raft storage.**

## Pre-requisites

1. Install [Vagrant](https://www.vagrantup.com/docs/installation)

2. Install [VirtualBox](https://www.virtualbox.org/)

## Clone the Repository

```shell
git clone git@github.com:dlavric/vagrant-vault-ha-raft-tls.git
cd vagrant-vault-ha-raft-tls
```

## How to use this repository

1. Start Vagrant:

```shell
vagrant up
```

2. Connect to the `vault-1` virtual machine:

```shell
vagrant ssh vault-1
```

3. Export the `VAULT_ADDR` of Vault and check its status:

```shell
export VAULT_ADDR="http://127.0.0.1:8200"
vault status
```

You will observe that Vault has not been initialized or unsealed:

```
Key                Value
---                -----
Seal Type          shamir
Initialized        false
Sealed             true
Total Shares       0
Threshold          0
Unseal Progress    0/0
Unseal Nonce       n/a
Version            1.7.0
Storage Type       raft
HA Enabled         true
```

4. Initialize and unseal Vault

```shell
vault operator init \
    -key-shares=1 \
    -key-threshold=1 > unseal.key

vault operator unseal <unseal_key_1>
```

The output of `vault status` for `vault-3` should look like this:

```
Key                     Value
---                     -----
Seal Type               shamir
Initialized             true
Sealed                  false
Total Shares            1
Threshold               1
Version                 1.7.0
Storage Type            raft
Cluster Name            vault-cluster-67eafb06
Cluster ID              58eba43b-6b68-ffaf-360e-cc7392660cf0
HA Enabled              true
HA Cluster              https://192.168.56.153:8201
HA Mode                 active
Active Since            2021-10-07T11:00:45.01605804Z
Raft Committed Index    29
Raft Applied Index      29
```

5. Logout and then log into the other Vault node:

```shell
logout
vagrant ssh vault-2
```

6. Initialize and unseal Vault node `vault-2`:

```shell
export VAULT_ADDR="http://127.0.0.1:8200"

vault operator unseal <unseal_key_1> 

vault operator raft join <IP-from-HA-Cluster>

vault login <root_token>

vault operator raft list-peers
```

The output of `vault status` for `vault-1` should look like this:

```
Key                     Value
---                     -----
Seal Type               shamir
Initialized             true
Sealed                  false
Total Shares            1
Threshold               1
Version                 1.7.0
Storage Type            raft
Cluster Name            vault-cluster-67eafb06
Cluster ID              58eba43b-6b68-ffaf-360e-cc7392660cf0
HA Enabled              true
HA Cluster              https://192.168.56.153:8201
HA Mode                 standby
Active Node Address     http://192.168.56.153:8200
Raft Committed Index    458
Raft Applied Index      458
```

The output of `vault operator raft list-peers` should look like this:

```
Node     Address                State       Voter
----     -------                -----       -----
raft3    192.168.56.153:8201    leader      true
raft2    192.168.56.152:8201    follower    true
```

7. Repeat step `5` and `6` for `vault-1`

**NOTE: The unseal key is the same for all 3 nodes of the cluster**

The output of `vault status` for `vault-1` should look like this:

```
Key                     Value
---                     -----
Seal Type               shamir
Initialized             true
Sealed                  false
Total Shares            1
Threshold               1
Version                 1.7.0
Storage Type            raft
Cluster Name            vault-cluster-67eafb06
Cluster ID              58eba43b-6b68-ffaf-360e-cc7392660cf0
HA Enabled              true
HA Cluster              https://192.168.56.153:8201
HA Mode                 standby
Active Node Address     http://192.168.56.153:8200
Raft Committed Index    213
Raft Applied Index      213
```

The output of `vault operator raft list-peers` looks like this:

```
Node     Address                State       Voter
----     -------                -----       -----
raft3    192.168.56.153:8201    leader      true
raft2    192.168.56.152:8201    follower    true
raft1    192.168.56.151:8201    follower    true
```

8. Now, log into the other nodes, like `vault-3`, `vault-2` and check the status if you want to:

```shell
logout
vagrant ssh vault-1
```

```shell
export VAULT_ADDR="http://127.0.0.1:8200"
vault status
```

### TLS Certificates

For setting up the certificates I have used OpenSSL and followed this [guide](https://jamielinux.com/docs/openssl-certificate-authority/introduction.html).

I have created the certificates locally on my Mac and copied the key and the certificate to Vagrant.

## Steps on how I setup the certificate in Vagrant and used it

**NOTE: Certificate is defined in the Vault configuration file**

1. Copy the cert to Vagrant's home directory `~` and then to `/vagrant`, give full permissions and change the ownership to `vault` user:

```
scp intermediate/certs/www.example.com.cert.pem vagrant@192.168.56.151:~

sudo chmod 777 www.example.com.cert.pem

sudo chown vault:vault www.example.com.cert.pem 

sudo cp www.example.com.cert.pem /vagrant
```

2. Copy the key to Vagrant's home directory `~` and then to `/vagrant`:

```
scp intermediate/private/www.example.com.key.pem vagrant@192.168.56.151:~

chmod 777 www.example.com.key.pem

sudo chown vault:vault www.example.com.key.pem 

sudo cp www.example.com.key.pem /vagrant
```
