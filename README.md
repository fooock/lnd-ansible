# Ansible LND role

![GitHub Workflow Status (with branch)](https://img.shields.io/github/actions/workflow/status/fooock/lnd-ansible/ansible.yml?branch=main&label=Ansible%20Tests&logo=github&style=for-the-badge)

Ansible role to install the Lightning Network Daemon ([LND](https://github.com/lightningnetwork/lnd)) as a `systemd`
service. By default, it uses sane defaults and some hardening measures for the Systemd service.

By default, all binaries are installed inside `/usr/local/lnd-<version>/` directory. So for example, if you are
installing the version `v0.16.0-beta`, if you want to invoke the `lncli` binary, you will need to use `/usr/local/lnd-v0.16.0-beta/lncli`.

### Requirements

This role requires a user with `sudo` permissions to work properly.

List of officially supported operating systems:

| ID           | Name         | Status             |
|--------------|--------------|--------------------|
| `ubuntu2004` | Ubuntu 20.04 | :white_check_mark: |
| `ubuntu2204` | Ubuntu 22.04 | :white_check_mark: |

### How to run this?

Create a playbook like this one:

```yaml
- hosts: lnd
  roles:
    - role: fooock.lnd
      become: yes
```

Note that you can use `become` at a global level instead at the role level. Now you can execute
your playbook executing:

```shell
ansible-playbook -i $VM_TARGET_IP, -u $VM_TARGET_USER --ask-become-pass playbook.yml
```

### Testing

You can execute tests using `molecule`. Install the [`requirements.txt`](molecule/default/requirements.txt) to execute
tests using the Docker driver. Inside this directory execute:

```bash
$ molecule test
```

### Variables

You can change some variables to install this role to fit your needs. The default values to install the
Lightning daemon node are the following ones:

| Name              	 | Value              	        |
|---------------------|-----------------------------|
| `lnd_user`    	     | `lnd`          	            |
| `lnd_group`   	     | `lnd`          	            |
| `lnd_version` 	     | `0.16.0-beta`             	 |
| `lnd_arch`    	     | `linux-amd64` 	             |

> If you want to install LND into a Raspberry you need to change the architecture to `linux-aarch64`.

By default, this installer uses `gpg` to verify the integrity and signature of the downloaded artifacts. This
behaviour is controlled by the `lnd_pgp_keys` field. The content of this structure is a list of names
available in the [official Github repository](https://github.com/lightningnetwork/lnd/tree/master/scripts/keys).

If you only want to verify with one user, you should use something like this:

```yaml
lnd_pgp_keys:
  - roasbeef
```

> Internally this method downloads the `user.asc` file and does all the job verifying the release.
> If the release can't be trusted the role will fail the installation.

To configure the LND node, you can use the following variables:

| Name                   	     | Value           	    | Note                                             	                               |
|------------------------------|----------------------|----------------------------------------------------------------------------------|
| `lnd_data_dir`     	         | `/data/lnd` 	        | 	                                                                                |
| `lnd_network`      	         | `mainnet`          	 | Valid values are: `testnet` and `simnet` 	                                       |
| `lnd_alias`     	            | `fooock`       	     | 	                                                                                |
| `lnd_log_level` 	            | `info`       	       | 	                                                                                |
| `lnd_listen`     	           | `127.0.0.1`     	    | 	                                                                                |
| `lnd_tls_extra_domain`     	 | `$HOSTNAME`     	    | Default VM hostname	                                                             |

Since by default the expected behavior is that LND node will run in the same instance where the Bitcoin
node is running, you can use the following fields (you need to change them if required):

| Name                   	           | Value           	                      |
|------------------------------------|----------------------------------------|
| `lnd_bitcoind_dir`     	           | `/data/bitcoin` 	                      |
| `lnd_bitcoind_config`      	       | `/etc/bitcoin/bitcoin.conf`          	 |
| `lnd_bitcoind_rpc_host`     	      | `127.0.0.1` 	                          |
| `lnd_bitcoind_rpc_user`      	     | `bitcoin`          	                   |
| `lnd_bitcoind_rpc_password`      	 | `bitcoin`          	                   |
