# Self-Host an Overleaf Instance on Your Laptop

## Backstory

I wanted to write a report for an exam with my friends, but we had the misfortune of being a team of 3, which is the limit for the free plan on Overleaf. We are students, so we could have paid €8 for the monthly student subscription or used Papeeria, but we are computer science students. So, we decided to take the path of pain: self-hosting an Overleaf instance without a physical server.

This was done in ArchLinux

---

## Setup

### What You Need

To host the Overleaf Community Edition, only the server/unfortunate laptop owner needs to complete this setup.

### Step 1 — Install Docker and Docker Compose

1. Run:
    ```bash
    sudo pacman -S docker docker-compose
    ```
2. Enable and start Docker:
    ```bash
    sudo systemctl enable --now docker
    ```
3. Add yourself to the Docker group to run Docker without `sudo`:
    ```bash
    sudo usermod -aG docker $USER
    ```
    Log out and log in again for this change to take effect.

### Step 2 — Download and Configure Overleaf CE Toolkit

1. Create a directory and clone the toolkit:
    ```bash
    mkdir ~/overleaf
    cd ~/overleaf
    git clone https://github.com/overleaf/toolkit.git
    cd overleaf-toolkit
    ```
2. Initialize the configuration:
    ```bash
    bin/init
    ```
3. Check the config files:
    ```bash
    ls config
    # You should see: overleaf.rc, variables.env, version
    ```
4. Edit the main configuration file:
    ```bash
    nano config/overleaf.rc
    ```
    Find the line `OVERLEAF_LISTEN_IP=127.0.0.1` and change it to:
    ```
    OVERLEAF_LISTEN_IP=0.0.0.0
    # This tells the Overleaf server to listen on all network interfaces and not only to your localhost
    ```
    Save and exit the editor.

5. Start the services:
    ```bash
    bin/up
    ```
6. Verify the server is listening on port 80:
    ```bash
    ss -tulpn | grep 80
    # You should see: 0.0.0.0:80 LISTEN
    ```

### Step 3 — Install and Set Up Tailscale

1. On your machine (the server), install Tailscale:
    ```bash
    sudo pacman -S tailscale
    sudo tailscale up
    ```
2. On your friends' devices, they need to:
    - Install Tailscale (available for Windows, macOS, Linux, Android, and iOS).
    - Accept your Tailscale invitation.
    - Ensure their Tailscale status is "Connected."

### Step 4 — Manage Users and Finalize Access

1. Go to the Tailscale admin panel: [https://login.tailscale.com/admin/machines](https://login.tailscale.com/admin/machines).
2. Go to the "Users" section and click "Invite external users." Send the invitation to your friends.
3. Once they accept, you will see them under "Users." You must approve them.
4. Find your server's Tailscale IP address:
    ```bash
    tailscale ip
    ```
5. Access your Overleaf instance by going to `http://<TAILSCALE_IP>/login` in a browser.
6. Create your admin account. You will arrive at the landing page.
7. Click on "Admin" and then "Manage Users." Add your friends' email addresses.
8. Copy the password reset link for each user. **Important:** Change the `localhost` part of the link to your `<TAILSCALE_IP>` before sending it to them.
9. Your friends must use this link to reset their password.
10. Once their password is set, they can log in at `http://<TAILSCALE_IP>/login`.

### Step 5 — Collaborate on Projects

Now all you have to do is create a new project and add your friends as collaborators using the same method you would normally use on the standard Overleaf platform.

1. Create a new project from the dashboard
2. Click on the project to open it
3. Click the "Share" button in the top right corner
4. Add your friends' email addresses as collaborators
5. They will receive access to the project immediately

**Voilà!** You now have your own Overleaf instance with as many collaborators as you want.
