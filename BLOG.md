#### A pattern for managing clusters in the cloud with ansible vault

If you've ever been tasked with standing up a cluster environment in the cloud, you've probably used a tool like Ansible to make your life a little easier.  By defining the configuration in code, admins can develop their cloud infrastructure in a maintainable and repeatable way.  This allows for iteration through trial and error when developing for cloud.

One drawback of developing in this fashion is that it can be difficult to maintain access to a cluster during development.  When large teams require access to a common cloud environment, security is often the first casualty in the development process.  Maintaining an individual private key for each developer is impractical while developing a cluster for a large team.  Cycling keys is a good idea but it means re-distributing the private key to the entire team every time a key is changed.

Ansible provides some useful functionality that can alleviate some of these maintenance woes. Ansible-vault utilizes symmetric key encryption which allows us to store private keys in our repository while still providing an acceptable level of security for our cloud.  Note that you should __never__ store unencrypted private keys in source control.

By storing these private keys in a repository, developers are able to access the cluster even if the private key has changed.  We can still cycle the private key frequently without the worry of hindering development.  Should a private key be misplaced, cycling keys is as simple as encrypting a new private key and replacing the old one in source control.

So how is this accomplished?
...
