# create_firefox-firejail
How to create a firejail instance for firefox.

The below is confirmed to work in Debian Linux and should work in any other Linux distro. Use of this routine allows the user to set up sandboxed instances of firefox, so that websites can't leak details to other websites. The routine can also be used to create isolated profiles for work, home, etc.

Note: If you have a specific style of setup for firefox, set it up in the template before you clone it to other profiles.


# Firejail + Firefox ESR Site Isolation (Linux)

This setup creates a clean Firefox template and per-site isolated sandboxes using Firejail and separate `$HOME` directories.

Model:

- One clean template (never used for browsing)
- Cloned per-site sandboxes
- Each site runs in its own `$HOME`
- No shared cookies or storage
- No cross-site tracking between sites

---

## 1. Install Requirements

    sudo apt install firejail firefox-esr

---

## 2. Create a Clean Template

Create template directory:

    mkdir ~/firejail-template

Ensure no Firefox instance is running:

    pkill firefox

Launch Firefox inside the template home:

    HOME=$HOME/firejail-template firejail --noprofile firefox-esr

Inside Firefox:

- Install uBlock Origin
- Enable strict tracking protection
- Disable telemetry
- Block microphone, camera, and location
- Set preferred search engine
- Do NOT log into anything
- Close Firefox normally

This directory is now your clean base image.

---

## 3. Verify Template Initialisation

    ls ~/firejail-template/.mozilla/firefox

You should see:

- A profile directory (e.g. xxxx.default-esr)
- profiles.ini
- installs.ini

Optional check:

    grep -R facebook ~/firejail-template/.mozilla/firefox

This should return nothing if the template is clean.

---

## 4. Make Template Read-Only (Optional)

    chmod -R a-w ~/firejail-template

To make writable again:

    chmod -R u+w ~/firejail-template

Do not launch Firefox from the template after this.

---

## 5. Create Per-Site Sandbox

Example: Facebook

    cp -a ~/firejail-template ~/firejail-facebook
    chmod -R u+w ~/firejail-facebook

Example: YouTube

    cp -a ~/firejail-template ~/firejail-youtube
    chmod -R u+w ~/firejail-youtube

Each clone starts identical and diverges only after use.

---

## 6. Launch Isolated Instance

Facebook:

    HOME=$HOME/firejail-facebook firejail --noprofile firefox-esr https://www.facebook.com

YouTube:

    HOME=$HOME/firejail-youtube firejail --noprofile firefox-esr https://www.youtube.com

Important: Always ensure no other Firefox instance is running:

    pkill firefox

---

## 7. Create Desktop Launcher (Optional)

Create:

    nano ~/.local/share/applications/facebook-isolated.desktop

Example content:

    [Desktop Entry]
    Name=Facebook (Isolated)
    Comment=Facebook in Firejail sandbox
    Exec=env HOME=/home/USERNAME/firejail-facebook firejail --noprofile firefox-esr https://www.facebook.com
    Icon=firefox-esr
    Terminal=false
    Type=Application
    Categories=Network;WebBrowser;

Make executable:

    chmod +x ~/.local/share/applications/facebook-isolated.desktop

Repeat for additional sites.

---

## 8. Reset a Sandbox

If a sandbox becomes dirty:

    rm -rf ~/firejail-facebook
    cp -a ~/firejail-template ~/firejail-facebook
    chmod -R u+w ~/firejail-facebook

Fresh instance in seconds.

---

# Architecture Summary

- Each site runs in its own `$HOME`
- No shared cookies
- No shared storage
- No cross-site tracking
- No access to your real home directory
- Template acts as a base image
- Sandboxes are disposable

---

## Verification

Inside any sandbox, open:

    about:support

Home Directory should show:

    /home/USERNAME/firejail-<sitename>

If it does, isolation is functioning correctly.
