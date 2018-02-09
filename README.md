### A Pattern for Managing Clusters in the Cloud with Ansible Vault

This post assumes a basic understanding of Ansible and AWS-EC2 Ansible modules as well as a working AWS account with boto configured.
http://boto.cloudhackers.com/en/latest/boto_config_tut.html

For this example we will be using ansible-vault to encrypt our private keys so that they can be shared with any version control tool

Benefits:
- Cycle private keys without constantly having to share keys with team members
- Distribute keys with any version control system
- Easily manage multiple clusters or multiple private keys among team(s)

Caveats:
- Must still manually distribute vault-key file to trusted individuals
- Must trust ansible-vault for symmetric key encryption

This playbook was created with Ansible 2.2.0

One thing to note is that I am writing to a static inventory file to track the hosts for the cluster. The recommended approach for this is to use ansible dynamic inventory:
http://docs.ansible.com/ansible/intro_dynamic_inventory.html

#### Getting Started
1. Generate vault-key.txt file in keys/ directory. Share this key with anyone who requires access to the cluster.  __This key should
never be committed to source control__
`date | md5 > keys/vault-key.txt && chmod 600 keys/vault-key.txt`

> Update: This isn't a very good way to generate a password, use `cat /dev/urandom | env LC_CTYPE=C tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1` instead to generate a random string. `date | md5` is deterministic, not random.

2. Define your cluster in `group_vars`

3. To run the playbook use:
`ansible-playbook example_main.yml -e env=dev`
This will create a private key, launch a cluster (using the cluster configuration in group_vars/), and add the hosts to an inventory
file. This inventory file and the encrypted `.vault` file can then be committed to source control for use by other team members.

4. Terminating the cluster is as simple as setting `count: 0` for each node in the cluster using the variable
definitions in `group_vars/`

#### Next steps
This example can be extended to support multiple keys and clusters.
Now, cycling ssh keys regularly is only a matter of creating an ansible role which will:
1. create a new private key
2. encrypt the new key with ansible-vault
3. replace the old key on each host
4. upload the new encrypted `.vault` file to source control where it will be available to anyone with access to your repository
