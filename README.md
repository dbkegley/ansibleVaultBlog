A pattern for sharing private keys for a cluster among a team.
This post assumes a basic understanding of Ansible and AWS-EC2 Ansible modules.
For this example we will be using ansible-vault to encrypt our private keys so that they can be shared with any SCM tool

Benefits:
- Cycle private keys without constantly having to share keys with team members
- Distribute keys with any version control system
- Easily manage multiple clusters or multiple private keys among team(s)

Caveats:
- Must still manually distribute vault-key file to trusted individuals
- Must trust ansible-vault for symmetric key encryption

Note: For simplicity, this playbook utilizes ansible tags to specify which part of the playbook to run. If no tags are passed, the playbook will launch an ec2 instance, cycle the keys, and then destroy the ec2 instance.  A single part of the play can be run using the `--tags` command line option. (launch, cycle, destroy)

This example was developed using Ansible 2.2.0

1. Generate vault-key.txt file in keys/ directory. Share this key with anyone who requires access to the cluster.  This key should never be committed to source control
`date | md5 > keys/vault-key.txt && chmod 600 keys/vault-key.txt`

2. 



Using variables, this example could be extended to support multiple keys and clusters
