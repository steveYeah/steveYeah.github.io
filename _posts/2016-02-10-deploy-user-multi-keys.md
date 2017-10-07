### Single user, multiple keys
GitHub has a really handy feature that allows you to add an SSH key to a repository that isn't tied to a 
personal user account, but to the repository itself. This key can then be linked to a deploy user on your system
that dosen't need a GitHub account, which you can use for your deployments. This then allows anyone with access to the deploy user to 
deploy your project. At this point you are probably thinking "Great!" and you should, it's a good idea. The 
problems start when you have more than one repository that is deployed by your deploy user. You can only use
an SSH key as a deploy key for one repository. If you try to use it for more than one, GitHub will not allow 
you to add the key.    

After a lot of reading, and a lot of trial and error, I found there are a few ways around this. One of these is
to have multiple deploy users, one for each project. This is probably OK if you only have a few projects,
but seems a little messy to me, but there is another way...   

Here I'll go through how to setup a single deploy user with multiple GitHub deploy keys.   
 
### Before we start
So far I have only tried this on a Debian 7.8 (wheezy). I have a deploy user named `deploy` that is not a
sudoer (for obvious reasons). In the examples we will look at setting up keys for `project_a` and `project_b`,
but replace these names with that of your projects. I am assuming that you have already cloned your repositories
using SSH and you are comfortable with the command line.

### 1. Generate the keys
First we have to create the keys. We need two keys, one for each project. I'm assuming you have done this
before but if not take a look at GitHub's [Generating a new SSH key](https://help.github.com/articles/generating-a-new-ssh-key/)

First `project_a`:
 
    $ ssh-keygen -t rsa -f ~/.ssh/id_rsa.project_a

Then `project_b`:

    $ ssh-keygen -t rsa -f ~/.ssh/id_rsa.project_b


The naming here is not essential, but I think it's quite logical. Feel free to use a different format that 
suits you if you like.

### 2. Add your deploy keys to your repository
This is nice and straight forward. Add `id_rsa.project_a.pub` to `project_a` and `id_rsa.project_b.pub` to `project_b`
by following the [deploy keys guide on GitHub](https://developer.github.com/guides/managing-deploy-keys/#deploy-keys)

### 3. Set up your SSH hosts
Now you have your keys and they are linked to the correct repository, you need a way to make sure you can use 
them. This is done by creating an SSH host. To do this you need to edit `~/.ssh/config` for your deploy user
and add the following:

    Host project_a.github.com
        Hostname ssh.github.com
        Port 443
        IdentityFile ~/.ssh/id_rsa.project_a

    Host project_b.github.com
        Hostname ssh.github.com
        Port 443
        IdentityFile ~/.ssh/id_rsa.project_b

If you are not familiar with SSH hosts then I'll try and explain a little. Here we have two hosts, one for 
each project. With this saved, if you now try `$ ssh project_a.github.com` it will attempt an SSH connection
to `ssh.github.com` on port `443` using the SSH key `~/.ssh/id_rsa.project_a`. **Note:** This will fail if you 
try it, and is only listed here in a an attempt to explain what it does, so don't panic :D.

### 4. Use your SSH hosts in you repositories
The final step is to start using your SSH hosts. Lets start with `project_a`. `cd` into the `project_a` directory
and list the existing origins. You should see something like this:

    $ git remote -v
    origin  git@github.com:username/project_a.git (fetch)
    origin  git@github.com:username/project_a.git (push)

Now remove that origin:

    $ git remote rm origin

Now we can add our new origin using our SSH host for `project_a`

    $ git remote add origin git@project_a.github.com:username/project_b.git
    $ git remote -v
    origin  git@project_a.github.com:username/project_b.git (fetch)
    origin  git@project_a.github.com:username/project_b.git (push)

To test things are OK try a remote git operation

    $ git remote update

and, assuming you followed this correctly, everything should work as expected. Nice! (If you get any problems, 
leave a comment and I'll try to help where I can)

Now you just have to follow step `4` again, but this time for `project_b`. After completing the steps for 
`project_b` just make sure both repositories are still OK.   
  
All Done! From now on you will be able to deploy either project from a single user. Hope it helps!
