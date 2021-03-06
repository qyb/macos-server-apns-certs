This repository is for getting APNs (Apple Push Notification service) certificates for, e.g., push email notifications on iOS. The certificate generation function has become unavailable on modern systems (typically requiring keeping an old VM around), so this project seeks to make it easier.

# Requirements

* macOS (tested on Catalina) and the macOS Server app 5.10 (currently the latest) downloaded from the Mac App Store

# Creating the certificate requests

By "requests" in the plural, I mean for com.apple.calendar, com.apple.mail, etc., on one server. You may only care about mail or something, but we just get them all.

1. Download macOS Server 5.10 from the Mac App Store, if you haven't already. It is not necessary to open or configure anything in it, just have it available in `/Applications`.
1. `git clone https://github.com/scintill/macos-server-apns-certs.git` in a convenient folder
1. If you've previously generated different certificates using this code:
	* **Copy the certificates and private keys somewhere else(!)**
	* run `make clean` to erase the old data and start clean for the next certificates.
1. Configure
	* `echo -n yourHostName > config/hostname` (the hostname you give here is displayed in the certificates portal webpage)
	* `echo -n yourAppleID@example.com > config/username` (you should probably use an Apple ID you've purchased macOS Server with, but I haven't tried whether they require that)
	* `openssl dgst -sha256 -binary | xxd -p -c 32 > config/passwdhash # type your Apple ID password, then Ctrl-D Ctrl-D`
1. Run `make request-body`. You should sanity-check the contents compared to `test/expected/request-body`.

# Sending your certificate request

```
curl -i --data-binary @request-body -H 'Content-Type: text/x-xml-plist' --user-agent 'Servermgrd%20Plugin/6.0 CFNetwork/811.11 Darwin/16.7.0 (x86_64)' https://identity.apple.com/pushcert/caservice/new -H 'Accept: */*' -H 'Accept-Language: en-us' | tee response
```

A success response starts something like this:

```
HTTP/1.1 100 Continue

HTTP/1.1 200 
Server: Apple
Date: Thu, 09 Jul 2020 05:42:01 GMT
Content-Type: text/x-xml-plist
Content-Length: 12992
Connection: keep-alive
Cache-Control: no-cache
Strict-Transport-Security: max-age=31536000; includeSubdomains
X-Frame-Options: SAMEORIGIN
Host: identity.apple.com
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubdomains
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Host: identity.apple.com
X-Frame-Options: SAMEORIGIN

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Response</key>
    <dict>
        <key>Status</key>
        <dict>
            <key>ErrorDescription</key>
            <string></string>
            <key>ErrorMessage</key>
            <string></string>
            <key>ErrorCode</key>
            <integer>0</integer>
        </dict>
        <key>Certificates</key>
        <array>
            <dict>
                <key>Certificate</key>
                <string>-----BEGIN CERTIFICATE-----
```

Your certificates should be in the file `response`. The private keys are in the `keys` directory.

You can view and revoke your certificates at https://identity.apple.com/pushcert/ at the bottom.

# "Makefiles?!"

Yeah, it's kind of horrible, but I think it was a decent way to iteratively build up the pieces of the request data during the development process. I only anticipate needing this once a year, so I might not make a nicer implementation...
