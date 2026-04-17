# Runbook: Capsule Certificate Renewal & Generation

> **Auto-renewal is DISABLED on all capsule certificates. Manual action is required before expiration.**

---

## Prerequisites

Before starting, confirm you have:

- [ ] SSH access to the Satellite (SAT) server with root privileges
- [ ] An admin account with access to the Certificate Manager portal
- [ ] The FQDN and IP of the target server
- [ ] The incident/ticket number for this renewal

---

## Step 1 — Prepare Your Server List

Identify all servers that need the renewed certificate installed. Verify the target FQDN and IP match the certificate's SAN fields in the Certificate Manager before proceeding.

- [ ] Note the target server FQDN
- [ ] Note the associated IP address
- [ ] Confirm the certificate entry exists in the Certificate Manager under the correct policy path

---

## Step 2 — Generate the Certificate Signing Request (CSR)

> Run all commands from the **SAT root home directory**.

### 2.1 — Run CertGen.sh

```sh
./CertGen.sh
```

### 2.2 — Navigate to the Certs Directory

```sh
cd capsule_certs/
ll
# You should see a list of directories named after each server
```

### 2.3 — Copy the CSR Content

```sh
cd capsule_certs/<servername>/
cat <servername>.csr
```

> Copy everything from `-----BEGIN CERTIFICATE REQUEST-----` to `-----END CERTIFICATE REQUEST-----` (inclusive).
> You will paste this into the Certificate Manager in Step 3.

---

## Step 3 — Renew the Certificate via Certificate Manager

### 3.1 — Log In

Log into the Certificate Manager portal with your admin account.

### 3.2 — Locate the Certificate

1. Navigate to: **Inventory → Certificates**
2. Filter by the server FQDN: `<servername>`
3. Click the certificate **hyperlink** in the results

### 3.3 — Initiate Renewal

1. Click **Actions** (upper right) → **Renew Now**
2. Click **Edit and Review** → verify all settings are correct
3. Click **Next**

### 3.4 — Paste the CSR

1. In the CSR field, paste the full CSR content copied in Step 2.3
2. Click **Submit**, then **Close** the wizard

### 3.5 — Confirm Renewal & Download

1. **Refresh** the page until the expiration date shows the newly issued date (~1 year out)
2. Click **Actions** (upper right) → **Download**
   - Format: **PEM (OpenSSL)**
   - Check **"Extract PEM..."**
   - Download and unzip the archive if needed
3. Open the downloaded `.crt` file in a text editor and **copy its full contents**

> A new expiration date in the Certificate Manager confirms the certificate was successfully issued.

---

## Step 4 — Install the Renewed Certificate on Satellite

### 4.1 — Paste the Renewed Certificate

```sh
vi capsule_certs/<servername>/<servername>.crt
# Paste the full certificate contents copied from the text editor
```

### 4.2 — Validate and Apply

```sh
./cert_check.sh <servername>
```

If the check passes with no errors:

```sh
./capsule-cert-update.sh
```

> No errors from `cert_check.sh` means the certificate is valid and ready to publish.

---

## Step 5 — Publish Result Files

### 5.1 — Create Installer File & Move the Tarball

**On the primary web satellite:**

```sh
vi /var/www/html/pub/certs/<servername>-installer.txt
mv <servername>.tar /var/www/html/pub/certs/
```

**On the secondary web satellite:**

```sh
vi /var/www/html/pub/capsules/<servername>-installer.txt
mv <servername>.tar /var/www/html/pub/capsules/
```

### 5.2 — SELinux Issue (if tarball does not appear in web directory)

> If the tarball doesn't show up after moving it, an SELinux context issue may be blocking access.

Check:

```sh
fixfiles check /var/www/html/pub/certs
```

Fix:

```sh
restorecon -Rv /var/www/html/pub/certs/
```

---

## Step 6 — Install the Certificate on the Capsule Server

> Run from your **Capsule Server**.

### 6.1 — Delete Old Files

```sh
rm <servername>.tar <servername>-installer.txt
```

### 6.2 — Download the New Installer Tarball

Verify the tarball is available on the web satellite, then download it:

```sh
wget https://<web-satellite-host>/pub/certs/<servername>.tar
```

### 6.3 — Set Permissions & Run Installer

```sh
chmod a+x <servername>-installer.txt
./<servername>-installer.txt
```

> No errors from the installer means the certificate is fully deployed on the Capsule server.

---

## Step 7 — Verify & Close Out

### 7.1 — Verify via OpenSSL

From any system with network access to the server:

```sh
openssl s_client -connect <servername>:443 -showcerts </dev/null 2>/dev/null \
  | openssl x509 -noout -dates

# Expected: notAfter should show ~1 year from today
```

### 7.2 — Close the Ticket

- [ ] Confirm the new expiration date in the Certificate Manager (Inventory → Certificates → filter by FQDN)
- [ ] Update and resolve the incident ticket with renewal completion notes
- [ ] Notify the appropriate team and contacts
