# osTicket Installation Troubleshooting Guide

Use this guide when the Lab 1 installation does not behave as expected.

The objective is to troubleshoot systematically rather than changing multiple settings at once.

## 1. IIS Does Not Load

Test `http://localhost`, then check:

```powershell
Get-Service W3SVC
Get-WindowsFeature Web-Server
```

## 2. osTicket Returns 404

Confirm the application exists at:

```text
C:\inetpub\wwwroot\osTicket
```

Verify that `index.php` exists and that IIS Manager shows the application under the Default Web Site.

## 3. PHP Is Not Working

```powershell
C:\PHP\php.exe -v
C:\PHP\php.exe -m
```

If this fails, verify the PHP files, architecture, and Visual C++ runtime. Restart IIS after configuration changes:

```powershell
iisreset
```

## 4. osTicket Reports Missing PHP Extensions

Compare `php -m` with the osTicket prerequisite page. Common modules include `mysqli`, `ctype`, `fileinfo`, `gd`, `gettext`, `iconv`, `imap`, `intl`, `mbstring`, `opcache`, `phar`, `xml`, and `zip`.

## 5. MySQL Service Is Not Running

```powershell
Get-Service MySQL*
```

Start the appropriate service if necessary.

## 6. osTicket Cannot Connect to MySQL

Verify the host, database, username, and password, then test manually:

```powershell
mysql -u osticket_user -p -h localhost osticket
```

## 7. Database User Does Not Have Permissions

```sql
SHOW GRANTS FOR 'osticket_user'@'localhost';
```

The user should have privileges on `osticket.*`.

## 8. osTicket Cannot Write ost-config.php

Grant only the IIS application identity the temporary permissions required to complete installation. Do not grant `Everyone` Full Control. Remove write access after installation.

## 9. Installation Completes but osTicket Shows a Security Warning

Delete:

```text
C:\inetpub\wwwroot\osTicket\setup
```

Then verify that `ost-config.php` is no longer broadly writable.

## 10. Review IIS Logs

Default IIS logs are commonly stored under:

```text
C:\inetpub\logs\LogFiles
```

Look for HTTP 404/500 errors and repeated requests to `/osTicket`.

## Troubleshooting Method

```text
1. Define the symptom
2. Reproduce the issue
3. Check service state
4. Check logs/output
5. Verify configuration
6. Change one thing
7. Retest
8. Document the result
```
