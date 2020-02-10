# SSM

This tool allows you to connect to AWS instances using the `awscli` tools and Session Manager Plugin without having to enable SSH.
This makes it easy to establish a session with your instance from anywhere (think VPN) using only your AWS credentials,
without having to specify Security Group Inbound Rules for specific IP ranges.

## Setup

If you wish to disable having an external IP altogether and still be able to access your instance using SSM, there are specific VPC endpoints you can set up in order to achieve this.
However, that is out of scope here, so just [read the SSM documentation](https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html), and we will assume
you are accessing an instance that has a public IP.

- [Install AWS Session Manager Plugin](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html#install-plugin-macos)
- Create a custom IAM Role, and attach the "AmazonEC2RoleforSSM" AWS managed policy to the role.
- Associate the new IAM Role with your EC2 instance.
- Enable SSH access in your instance's Security Group
- SSH into the instance, and upgrade the packages so that you are running the latest version of the SSM agent, which comes pre-installed on the most popular EC2 AMIs.
- You can either restart the ssm agent service, or for the sake of cross-compatibility, my instruction here is to reboot the instance.
- Disable SSH access by deleting the Inbound Rule in your instance's Security Group.
- Move the ssm script somewhere in your `$PATH`.

`ssm [instance id or public IP]`

`sudo su - [user] -c bash`

Bingo bango.

## Caveats

SSM won't launch a Bash shell initially and defaults to `/bin/sh` at the time of writing this.

That is why you have to run `sudo bash` to get a normal root shell or `sudo su - [ubuntu | ec2user] -c bash` in order to get a user-level prompt.

I have noticed that when launching a user-level prompt, it complains about job control.  You could possibly get around this by making sure python is installed and running the following:

`python -c 'import pty; pty.spawn("/bin/bash")`

(I haven't fooled around with it much, so YMMV.)

I have also noticed that if there is not regular output to `stdout`, your SSM session times out after a certain period of time, like an hour or so.  I haven't researched this, so there may be a way to
eliminate the timeouts or increase them programatically, but honestly, it was just simpler for me to use `screen` or `tmux` in order to reconnect to my previous session.  

