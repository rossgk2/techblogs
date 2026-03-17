# GitHub auth

## Overview of ways to auth to GitHub

The GitHub docs say:

> You can access your resources in GitHub in a variety of ways: in the browser, via GitHub Desktop or another desktop application, with the API, or via the command line. Each way of accessing GitHub supports different modes of authentication.

This can be misleading, as most people going to the GitHub authentication docs are probably *specifically* trying to authenticate themselves to Git operations on GitHub servers like `git clone` or `git push`, not just "access their resources in GitHub". When we focus on this concern, the relevant points are:

* The browser and the GitHub API cannot grant authentication to Git operations on GitHub servers.
* GitHub Desktop and Git commands in the CLI of your choice can grant authentication to Git operations on GitHub servers.

There's a couple ways to get authenticated with the CLI of your choice:

- Personal access token (HTTPS) - manually generated with GitHub GUI and copy-pasted into Git commands in the CLI of your choice
- SSH - one-time setup in CLI of your choice
- GitHub CLI (`gh auth login`) (HTTPS) after installing "GitHub CLI" in your CLI of choice
- Git Credential Manager (HTTPS) - automatically invoked by Git commands in the CLI of your choice

## Rationale for SSH keys

When I was working on a C# .NET app in Visual Studio, the GitHub auth GUI didn't work out of the box. I therefore had to swap the GUI out in favor of the CLI of my choice, which happened to be bash. My options in bash were:

- Personal access token (HTTPS)
- SSH
- GitHub CLI (HTTPS)
- Git Credential Manager (HTTPS)

Using a personal access token was out of the question since it's a lot of hassle any time you need to auth. All the other options seem to only require one-time setup, and are viable. Of the viable options, SSH sounds like it's least likely to change in the future, so, I'll go with that.

## Traditional way of setting up an SSH key

The GitHub documentation specifies three steps for setting up SSH key auth:

1. In the CLI of your choice (bash, for me), generate a public key, private key pair. This causes key files to be saved to `~/.ssh`  in every OS.
2. Register the public key with GitHub by going into your account settings > SSH keys. 
   1. You may also have to "SAML authenticate" your public SSH. This is very easy. Just click on your public SSH key in the GitHub browser UI and wait to be redirected through the single-sign on process.

3. Set up a SSH agent on your local machine. Configure the SSH agent to start upon machine startup so that, if your private key file has a non-default name, it is automatically provided behind the scenes (meaning you don't have to type in any credentials) every time you do a Git operation.

The documentation didn't fully cover my case because I was working on a Windows VDI, which made following step 3 impossible:

> SSH agents are a bash feature.
>
> Since I'm on a VDI that doesn't allow nested virtualization, I cannot use WSL 2 as my bash environment.
>
> Git bash, which *is* on the VDI, also will not work. SSH agents run with Git bash need to start up require the user to input the passphrase every time Git bash starts up, which is annoying.

I thought the best alternative to step 3 was to use a SSH agent and a private key with no passphrase. (In the case when using a secure VDI, this no passphrase option is still secure ,since you still have to login to the VDI to get to the private key.)

But then I realized this overcomplicates things; an equivalent[^1] approach is to use a private key with no passphrase, and change the `~/.ssh/config` file to specify a default which private key should be used for the host github.com.

[^1]: With a SSH agent, you only have to type the passphrase, if it exists, upon SSH agent startup. With the config file approach, you have to type a passphrase, if it exists for the key corresponding to your attempted connection, every time you try to attempt a connection. Clearly you don't have to do anything in either case if there isn't a passphrase.

I also found a fourth step necessary:

4. Since my firewall blocks the default GitHub port, edit the  `~/.ssh/config` file so that SSH connections to github.com are redirected to ssh.github.com:443. (This is GitHub's designated workaround for this firewall issue.)

## My way for SSH key auth on Windows VDI

Here's a concise description of the best way I could come up with to set up SSH key auth for GitHub on a Windows VDI:

1. Use Git bash to generate a public key, private key pair using a CLI tool.

   ```
   ssh-keygen -t ed25519 -N ""
   ```

   (`-N ""` means that we are using no password to protect the private key.)

   In every OS, this causes key files to be saved to `~/.ssh` .

2. Go into your GitHub account settings > SSH keys. Create an "authentication key" backed by the public key (copy paste the contents of `~/.ssh/ed25519`), and create a "signing key" backed by the same public key (copy paste the contents of `~/.ssh/ed25519` again).

   1. You may have to "SAML authenticate" your public SSH key. This is very easy. Just click on your public SSH key in the GitHub browser UI and wait to be redirected through the single-sign on process.

3. Change the `~/.ssh/config` file to specify a default which private key should be used for the host github.com.

   ```
   Host github.com
       HostName github.com
       User git
       Port 443
       IdentityFile ~/.ssh/id_ed25519
       IdentitiesOnly yes
   ```

   The `Port 443` specification is necessary to redirect SSH connections from `ssh.github.com` to `ssh.github.com:443`, since my firewall blocks the default GitHub port. (`ssh.github.com:443` is GitHub's designated workaround for this issue.)

   `IdentitiesOnly yes` is used to ensure that only the key in the provided `IdentityFile` is tried when a SSH connection to the `Host` of `github.com` is attempted.