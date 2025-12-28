# Proxmox VE: Centralized SMTP Relay with Brevo (Hardened)

## This guide explains how to configure Proxmox VE to send system notifications using Brevo as an SMTP Relay.
Key Features

- Centralized Management: Monitor multiple nodes and clients with a single professional email.

- Zero-Trust for Clients: No need to request sensitive email credentials from your clients.

- Hardened Security: Uses chattr +i immutability to lock down sensitive configuration files.

- Professional Branding: Uses Postfix generic mapping to "beautify" the sender's name.

### 1. Get your SMTP Credentials from Brevo

Log in to Brevo.

Click on your Profile Name (top right) and select "Configuration".

Inside the menu, find and select "SMTP & API".

Go to the "SMTP" tab and click "Generate a new SMTP key".

Copy the key immediately (it won't be shown again).

![brevod](https://github.com/user-attachments/assets/3d149138-df96-4029-9d68-06d0928f1668)


### 2. Proxmox Terminal Configuration

A. Create the Credentials File

```Bash:
nano /etc/postfix/sasl_passwd
```
Paste: [smtp-relay.brevo.com]:587 YOUR_BREVO_EMAIL:YOUR_GENERATED_SMTP_KEY


B. Security: Hashing and Immutable Lock

This ensures your credentials are encrypted and locked against any modification.

Generate Hashed Database: 
```Bash:
postmap /etc/postfix/sasl_passwd
```
Restrict Access and Set Immutable Flag: 
```Bash:
chmod 600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
chattr +i /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
```

C. Optional: Beautify the Sender (Generic Mapping)

Create the generic map file: 
```Bash
nano /etc/postfix/generic
```
 Add the mapping: `root "Proxmox Service" <tu-email-verificado@dominio.com>`

Map and lock it: 
```Bash
postmap /etc/postfix/generic
chmod 600 /etc/postfix/generic /etc/postfix/generic.db
chattr +i /etc/postfix/generic /etc/postfix/generic.db
```

D. Configure Postfix main.cf

Edit the file: 
```Bash
nano /etc/postfix/main.cf
```

Add these lines at the end:
```
relayhost = [smtp-relay.brevo.com]:587 
smtp_sasl_auth_enable = yes 
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd 
smtp_sasl_security_options = noanonymous 
smtp_use_tls = yes 
smtp_generic_maps = hash:/etc/postfix/generic (ONLY if you completed Step C)
```
E. Apply and Restart
```Bash:
systemctl restart postfix
```
### 3. Testing

Verify by sending a test email: 
```Bash:
echo "Test desde el nodo" | mail -s "Check Postfix OK" your-destination@email.com
```
![Mail](https://github.com/user-attachments/assets/e1fc63cc-0450-470f-a541-649aad04a09f)
