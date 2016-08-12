### A Pattern for Managing Clusters in the Cloud with ansible-vault

If you've ever been tasked with standing up a cluster environment in the cloud, you've probably used a tool like Ansible to make your life a little easier.  By defining the configuration in code, admins can develop their cloud infrastructure in a maintainable and repeatable way.  This allows for iteration through trial and error when developing for cloud.

One drawback of developing in this fashion is that it can be difficult to maintain access to a cluster during development.  When large teams require access to a common cloud environment, security is often the first casualty in the development process.  Maintaining an individual private key for each developer is impractical while developing a cluster for a large team.  Cycling keys is a good idea but it means re-distributing the private key to the entire team every time a key is changed.

Ansible provides some useful functionality that can alleviate some of these maintenance woes. Ansible-vault utilizes symmetric key encryption which allows us to store private keys in our repository while still providing an acceptable level of security for our cloud.  Note that you should __never__ store unencrypted private keys in source control.

By storing these private keys in a repository, developers are able to access the cluster even if the private key has changed.  We can still cycle the private key frequently without the worry of hindering development.  Should a private key be misplaced, cycling keys is as simple as encrypting a new private key and replacing the old one in source control.

##### So how is this accomplished?
First, we will generate a random key to use as our vault password file.  This key should only be shared with trusted individuals and it should never be commited to source control.  Generate the key and place it in the `keys/` directory using the following command:
`date | md5 > keys/vault-key.txt && chmod 600 keys/vault-key.txt`

There are many ways to generate random strings, I'm using `date | md5` for the sake of simplicity but any method will work.

Define your cluster for whichever environment you want to launch. Here I'm using dev: `group_vars/dev/main.yml`
```
cluster:
  - name: web
    count: 1
  - name: db
    count: 1
  - name: worker
    count: 1
```

Now, just launch the cluster by running the top-level playbook and pass the extra `env` variable to define the environment.
`ansible-playbook example_main.yml -e env=dev`

You should see your cluster launching through the AWS console.  It's also important to note that the play has saved your cluster private key in the `keys/` directory.  This key can be used to connect to any of the nodes in the cluster over ssh.

There is now an encrypted `.vault` file in the `keys/` directory.  This is your encrypted private key file.  This file is safe to be committed to version control because it cannot be decrypted without `vault-key.txt` which we generated in the first step.  Once it is in your repository, any team member that possesses the vault-key will be able to access the cluster.

Team members can run `ansible-playbook example_main.yml -e env=dev --tags=key` once they have obtained the `.vault` file (from version control) and the vault-key (from you). This will decrypt the vault file and place the cluster private key in their `keys/` directory

Should a vault-key be misplaced, it may be necessary to change your vault-key file and re-distribute it to your team.  This can be done using the `ansible-vault rekey` command.  As an additional security measure, you should also cycle your private keys at that time

This is enough to get your started but there are many ways this concept could be extended to support different use cases.

The source for this project can be found here: www.link.to.source.github.com
