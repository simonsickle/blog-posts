## 1Password Meets Git

*👋 Hey there! I'm Simon, a senior Android engineer at [Block](https://block.xyz). I have been hacking around on Android roughly since its creation: from kernel mods to app creation, I have done a little bit of it all. I also dabble in other projects, like home automation and helpful utilities to make my life a little bit easier.*

## Security and you

With the ever evolving complexity of the internet, you may not even realize the amount of attack surface available to a bad actor. We all know the best practices now that have been hammered into our heads: use a random password different for every website, use two-factor auth, use a password manager to encrypt your digital life, etc. Most engineers will abide by these well established rules, but how many take it a step further and apply it to the software we create? This is where [1Password 8](https://1password.com/) comes in.

### Key to the kingdom

Without going too far in depth, SSH keys can be thought of the same way as your home's front door key - you use something you have to prove that access should be granted. For years this has been a standard and secure practice to access hosts over the internet by generating and storing a key on your local disk. What happens if your laptop takes a bath or you decide that shiny M1 processor is worth an upgrade, though - you now need to go through a long process of restoring your backup (you have one, right?) or creating a new key and adding it to all of your services... This is *not fun*.


What if I told you there is an easier way, using a tool that should already be in your back pocket?

### 1Password saves the day (key)

1Password has a wonderful SSH key generation functionality built into the mobile and desktop versions. Simply hit "New Item" and pick "SSH Key". 

![Screen Shot 2022-09-15 at 6.21.19 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663280502282/GHG1CzO8I.png align="left")

Give it a good name, hit "add private key" and click "generate". By default [Ed25519](https://developer.1password.com/docs/ssh/manage-keys/#ed25519) is used to generate the key but you may select RSA instead if you need that compatibility. *Most* servers now days should work fine with Ed25519, the most modern and fastest standard, however.


![Screen Shot 2022-09-15 at 6.24.47 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663280700174/KSvIoT-_B.png align="left")

Save your new key to your private vault and you are ready to go (well, almost... stick with me). Now you will click on the row in 1Password that says "public key" and copy it to your clipboard. Go [add that to GitHub](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account#adding-a-new-ssh-key-to-your-account) now.

### Ok, but how do I use it

Ok you caught me, you have a fancy new key store but no way to actually use it to SSH into your servers or to clone a repository. Worry not, 1Password thought of this and provides a SshAgent to do our bidding (err, our authenticating at least). 

To enable this functionality, we need to tell 1Password that we are a developer which also enables fancy CLI options! Hit `CMD + ,` or open the app settings in the way your OS supports and click "developer". Enable the SSH Agent and optionally biometrics if you wish.  Now, we must add a snippet of code to our SSH configuration file to tell the SSH command we wish to delegate key management to 1Password. You can follow the directions and copy the snippet like in the screen below.

![Screen Shot 2022-09-15 at 6.32.28 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663281159509/5BXDDrfPV.png align="left")

Once you have done this successfully,  you will see a screen that looks like this telling you how awesome you are.

![Screen Shot 2022-09-15 at 6.31.20 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663281107611/8P5Bal3eV.png align="left")

#### Note: what if I only want to use it for git and not my servers?

No worries then, I have you covered! At work, we have some complex configurations and security rules that wouldn't allow me to store my SSH key in this manor... I do, however, work on some open source software from time to time which requires that I have an SSH key to authenticate with GitHub. To accomplish this, I modified the pasted snippet from 1Password to only use the SSH agent for github connections using the git user. Run the following command (on macOS - for Linux and Windows you need to change the SSH Agent location)

```bash
echo 'Host github.com
    User git
    Hostname github.com
    IdentityAgent "~/Library/Group Containers/2BUA8C4S2C.com.1password/t/agent.sock"' >> ~/.ssh/config
```

And to confirm that everything is working, you can run the following command to see if 1Password prompts you for authentication and if GitHub recognizes your key!

```bash
ssh git@github.com
```

### What about signing my code?

Luckily, GitHub now supports code signed by an SSH key! This means you can use 1Password's SSH agent functionality to sign your commits instead of relying on GPG (which has been troublesome in the past on macOS).

To configure this, make sure that you have updated to the latest version of 1Password 8 for your OS and then you should see a "configure for commit signing" pop-up at the top of your key which can auto-configure git to use 1Password for signing. You can learn more about this feature from [1Password's support site](https://developer.1password.com/docs/ssh/git-commit-signing/). Be sure to add your SSH key to as a "signing" type key this time (it may be the same as your auth key, but you need to add it in both auth and signing locations).

Last thing to do is to sit back and enjoy your now biometrics secured workflow


![Screen Shot 2022-09-15 at 7.25.38 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663284393270/KgPNMpQWy.png align="left")

## Summary

Security is important and everyone must play a part in a secure ecosystem. Humans are the weakest link in the chain and are [often the target by malicious actors](https://www.theverge.com/2020/7/15/21326656/twitter-hack-explanation-bitcoin-accounts-employee-tools). By migrating to your SSH keys (the very tool used to identify yourself and provide access) to 1Password, you are insuring a smaller attack surface and saving yourself from having to type a (likely weak) password every time you need to sign a git commit or connect over SSH by using biometrics, a security-key (like a YubiKey), or your secure vault password that won't be shared on any other sites. Don't just take my word for it though, go inspect [their security model](https://support.1password.com/1password-security/) yourself to decide what is right for you. 
