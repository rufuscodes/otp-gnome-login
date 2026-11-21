# otp-gnome-login

Add two-factor authentication (TOTP) to the **GNOME (GDM) login screen** on
Ubuntu / Debian.

Require a 6-digit code from an authenticator app (Google Authenticator, Authy,
1Password, Aegis, etc.) **in addition to** your normal password. Shares one
secret with SSH OTP, so a single app entry unlocks both.

Tested on **Ubuntu 26.04 LTS / GDM3 / GNOME Shell (Wayland)**.

## What's included

| File | Purpose |
|------|---------|
| [`setup-gnome-otp.sh`](setup-gnome-otp.sh) | Install the PAM module, generate (or reuse) your TOTP secret, configure GDM |
| [`gnome-otp.md`](gnome-otp.md) | Full step-by-step tutorial, manual commands, and troubleshooting |

## Requirements

- Ubuntu / Debian with `sudo`
- An authenticator app on your phone
- (Recommended) SSH OTP already set up — see
  [`rufuscodes/otp-ssh-login`](https://github.com/rufuscodes/otp-ssh-login)

## Quick start

```bash
chmod +x setup-gnome-otp.sh

./setup-gnome-otp.sh            # nullok (DEFAULT): only users WITH a secret are prompted
./setup-gnome-otp.sh required   # strict: every user must have a secret
```

> If `~/.google_authenticator` already exists (e.g. from SSH OTP), the script
> **reuses it** — it will not overwrite the secret SSH depends on.

Then log out of GNOME and confirm the login screen asks for your password,
followed by the 6-digit code.

## Modes

`setup-gnome-otp.sh` accepts one argument:

| Mode | GNOME asks for | Who is affected |
|------|----------------|-----------------|
| `nullok` (default) | password, then code | only users with a `~/.google_authenticator` file |
| `required` | password, then 6-digit code | every user |

The default (`nullok`) is the safe choice: users who haven't set up OTP keep
logging in with just their password, so nothing breaks for existing accounts.
Users who **do** have a `~/.google_authenticator` file are required to enter
their code.

## What the script does

1. Installs `libpam-google-authenticator`
2. Generates your TOTP secret **only if** `~/.google_authenticator` is missing
   (otherwise reuses the existing one — shared with SSH)
3. Backs up `/etc/pam.d/gdm-password`
4. Inserts `auth required pam_google_authenticator.so nullok` (or plain
   `required` in strict mode) after the password check (`@include common-auth`)
5. Prints the resulting auth stack

No GDM restart is required — the change is read on the next login.

## Verify

Test the PAM module in isolation with `pamtester` (see
[`gnome-otp.md`](gnome-otp.md) for the full snippet):

```bash
echo "$CODE" | pamtester ga-test you authenticate     # valid code  -> success
echo "000000" | pamtester ga-test you authenticate    # wrong code   -> failure
```

## Rollback

```bash
sudo sed -i '/pam_google_authenticator\.so/d' /etc/pam.d/gdm-password
```

or restore the timestamped backup the script created.

## Full walkthrough

See [`gnome-otp.md`](gnome-otp.md) for the manual step-by-step, every directive
explained, copy-paste one-command versions, and a troubleshooting table
(lockout recovery, `nullok`, clock drift, lost phone).
