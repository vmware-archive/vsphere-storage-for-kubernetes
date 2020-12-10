---
  title: SAML token authentication
  summary: "This documentation is not applicable to vSphere CSI Driver. Please visit https://vsphere-csi-driver.sigs.k8s.io/ for information about vSphere CSI Driver."
---

As of Kubernetes [release 1.11](https://github.com/kubernetes/kubernetes/releases/tag/v1.11.0), the vSphere Cloud Provider includes support for SAML token authentication using the vCenter [SSO API](https://code.vmware.com/apis/34/vcenter-sso).
Currently, VCP only issues Holder-of-Key tokens which require a vCenter Solution User and key pair for signing SAML requests.
The preexisting VCP configuration user and password options can be set to the Solution User's public and private keys to enable this feature. A vCenter Solution User can only use public key authentication via SAML, a Solution User does not have a plain text password.

We expect to enhance the SSO related configuration options in future versions of vSphere Cloud Provider. This includes the ability for a Solution User to act as an existing Person User, but a Solution User is required in either case.


## Generate a self-signed certificate

```
% openssl req -newkey rsa:2048 -x509 -days 365 -nodes -keyout k8s-vcp.key -out k8s-vcp.crt -subj "/C=US/ST=CA/L=SF/O=VMware/OU=CNA/CN=www.vmware.com"
Generating a 2048 bit RSA private key
...............................+++
...............................................................................+++
writing new private key to 'k8s-vcp.key'
-----
```

## Create a solution user
Using [govc v0.18](https://github.com/vmware/govmomi/releases/tag/v0.18.0) or higher:

```
% govc sso.user.create -A -R Administrator -C "$(cat k8s-vcp.crt)" k8s-vcp
```

Or, using [dir-cli](https://www.virtuallyghetto.com/2015/05/vcenter-server-6-0-tidbits-part-9-creating-managing-sso-users-using-dir-cli.html):

```
% /usr/lib/vmware-vmafd/bin/dir-cli service create --wstrustrole --ssoadminrole Administrator --cert k8s-vcp.crt --name k8s-vcp
Service [k8s-vcp] created successfully
```

The example openssl command can be run anywhere, but if you use dir-cli to create the solution user, the public key will need to be local to the vCenter machine. The public key file can be removed from the vCenter machine after the user is created.

The Administrator role used in the examples above can be replaced with any role name that contains the minimal [privileges required](https://vmware.github.io/vsphere-storage-for-kubernetes/documentation/vcp-roles.html#minimal-set-of-vsphere-rolesprivileges-required-for-dynamic-persistent-volume-provisioning-with-storage-policy-based-volume-placement).

A users role can be changed using the update command, for example:

```
% govc sso.user.update -R k8s-role-name k8s-vcp
```
When a solution user is created, it will automatically be added to the SolutionUsers group. There are no other roles or permissions required for solution users. A user's role and group membership can be viewed using the ```govc sso.user.id``` command:

```
% govc sso.user.id k8s-vcp
solution=k8s-vcp@vsphere.local groups=LicenseService.Administrators,ActAsUsers,Administrators,Everyone,SystemConfiguration.Administrators,SolutionUsers
```

## Cloud config options
vSphere Cloud Provider will use SAML token authentication if the user config option is set a PEM encoded public key and password option to the private key.
**Note** that newlines in ```gcfg``` values must be escaped, which can be scripted for example:

```
% cat <<EOF
[VirtualCenter "$(govc env -x GOVC_URL_HOST)"]
        user = "$(awk '{printf "%s\\n", $0}' k8s-vcp.crt)"
        password = "$(awk '{printf "%s\\n", $0}' k8s-vcp.key)"
EOF
```
Resulting in:

```
VirtualCenter "example-vcenter.eng.vmware.com"]
    user = "-----BEGIN CERTIFICATE-----\nMIIDkTCCAnmgAwIBAgIJALVKv3+BwTORMA0GCSqGSIb3DQEBCwUAMF8xCzAJBgNV\nBAYTAlVTMQswCQYDVQQIDAJDQTELMAkGA1UEBwwCU0YxDzANBgNVBAoMBlZNd2Fy\nZTEMMAoGA1UECwwDQ05BMRcwFQYDVQQDDA53d3cudm13YXJlLmNvbTAeFw0xODA2\nMTEyMTAzMTdaFw0xOTA2MTEyMTAzMTdaMF8xCzAJBgNVBAYTAlVTMQswCQYDVQQI\nDAJDQTELMAkGA1UEBwwCU0YxDzANBgNVBAoMBlZNd2FyZTEMMAoGA1UECwwDQ05B\nMRcwFQYDVQQDDA53d3cudm13YXJlLmNvbTCCASIwDQYJKoZIhvcNAQEBBQADggEP\nADCCAQoCggEBALChi3frVLyKG2KC9SQyidW5Ji2iOaDMxRZvQiDw/3oNvpFa4oPa\nROFkoi/0uFPLcBhJsduGDnb2gRafNnc+CjvwrqaEESKBgUT6fbtq+ECgV+YJvVs2\nNYdG3ScmLkvr8d5yHDdaVYF5ccq/Z4s6+alc8wHMUyayoqtXTYXf3ksoTgz/z+gD\nQoy5JWXUzfkwvQ5eJs8SVgioLkeNoZ6RMHJCzt9ZUf1pXiuH0fUR9XSz5k/2clRV\nHRnXCPbqBtuBOn15eyr5Ssy4lHb+DYHE0k5KiQNc6lDlPG42hFby+FhOQ0H7RNmV\ncsPKqVsQl918GsKrneM4i4WLF4Wgl1n1f1sCAwEAAaNQME4wHQYDVR0OBBYEFPz1\nmwLeEs3KWF94VdYWxISKwpBCMB8GA1UdIwQYMBaAFPz1mwLeEs3KWF94VdYWxISK\nwpBCMAwGA1UdEwQFMAMBAf8wDQYJKoZIhvcNAQELBQADggEBAIdgpgpFQjSUUiRS\n33PliI9440Ul53/UgWs/0Q0Lmrd1Y07QJ97IgL39zbJBiU8Ndwhup6SEpG/N/F5C\ne3BEZSlM4l6HPpeQ7N8GqMvQt333IvYazKSvMmKisJe6Su7w8NjHbn+yKPDpWc+X\n8dSxqDNbAtTEipHICTUbpuDTM7SF8ZnwdI7viUcMBZOX7cU3uCFC6BqguejLmEH/\neoJtQAwrTrNPakDG77yQyU4EI1Px8CcaxL4pY2DieAkSU8Ors6hZewxC0m9Q0Oth\nsaJY5XigXVGRM7yI23PrZcCBAy7wA1KZNtthSMs1m6zO7NctXm1c/PmYl9PaMWhL\ndUfBxkA=\n-----END CERTIFICATE-----\n"
    password = "-----BEGIN PRIVATE KEY-----\nMIIEvAIBADANBgkqhkiG9w0BAQEFAASCBKYwggSiAgEAAoIBAQCwoYt361S8ihti\ngvUkMonVuSYtojmgzMUWb0Ig8P96Db6RWuKD2kThZKIv9LhTy3AYSbHbhg529oEW\nnzZ3Pgo78K6mhBEigYFE+n27avhAoFfmCb1bNjWHRt0nJi5L6/Hechw3WlWBeXHK\nv2eLOvmpXPMBzFMmsqKrV02F395LKE4M/8/oA0KMuSVl1M35ML0OXibPElYIqC5H\njaGekTByQs7fWVH9aV4rh9H1EfV0s+ZP9nJUVR0Z1wj26gbbgTp9eXsq+UrMuJR2\n/g2BxNJOSokDXOpQ5TxuNoRW8vhYTkNB+0TZlXLDyqlbEJfdfBrCq53jOIuFixeF\noJdZ9X9bAgMBAAECggEAQUrWJXQmlLNwwA+s0r6j2Q9iH4hSSTCowkxKY6byqYmf\nIlg4V4k94Ru0IIoUAVW4kCHdz0pU2oDw4w3jslyKp/GmfgNf2iOJR5hZFgjK0Aj1\ntSFwj+EQFHuLkMc6YfJMLHB+IbAQ35WnDM2IVx1r4MFtSwLe0fVC0JerHovMvncA\neVFg2QEcE1cEZKHexBWLmxIvjWb1yK0HET8R2GTcR+NHHRtBoxnXHHBLX+OxP55s\n6lsc8+jW6W9to5iMZOSzJH4WEyQ3ltg8HLbofrC6t6yzXSM4hvPD8fTpJsgogrrX\nZODltvBiqoZ8u36GHgrgeV5RK1pb+2QV/vvSJulhQQKBgQDXgy0OrTcMHhrk/3RP\nAp2A8LmPYJFVl1xSu6hWR4q0z/Jbjhf2nXBYDi/r2jMsQkowtcJx7qacyoaKvzij\ndmpNJQVygmi/p+vdi4Hsf3VRy0VbY4qobkvpRP1TirPAe2nkQSdjodc0MSNAJnyj\n1hBxxDg9zy3RSlUoYmihN7qCEQKBgQDR0GnXoMVwL/xSv4ozSBuo/kgkuAvB008H\nzIB42QlD+r6yluj8ipsvbeR8fq0miInplaNH8nsNNZyLSfN0SX9qMckdVgLJthQu\nQV0s7tqiroVTZJ4vmiW7VcP3zUTItUWmrDPoZ7eMjINHdD4L3M2iJNJ+xNB3mPDP\nbL8wHli+qwKBgBuBFkMFQD0/qlcHcySSRN+r2UK/JE00IAg/AuDgCIfC8j9VByHm\nPew/A0aqdlVzsFw/Fi3MM19XSYxzkxrphe+KhgNzOUMcfzGrGE3ChoqF0rgzIAMW\n8IE42MvMq9wo4/7JgelpQjna+5C4WLfgHgEm9baNtl87iVq6FHhe0GLBAoGALrMf\ny9HKAFV96QEfBpkHJw8qCZo5a7PXxFmdQsi0CkB2T5PNWeCT9/OSxq7/ZTNA1w/q\nXuo2v1LufAZCvOBbDsz0AaaSSklPppf/4C9t1IXZwR0FJH0/5rmJO8+hfrbyQM3V\nY+Yp8YuY8L+Ly+IilvNxMqwl5mjROKnwyAoJIK8CgYBUUtjoTYneBeOSIiHYaHgB\noPSMATyVvx1wSnPQU0Z7CXLWfLcdWd48+YxxY+CLT+O7LeD1HJBuL+clUkk3LLSU\nq7Gwz7JttCax67VJ+Rvca/Ye99Z08San+oyleOc4vaWn+m3elfLYY35I6q2mjEGH\nK6YwNTQeE+GxGIJJncNJXQ==\n-----END PRIVATE KEY-----\n"
```
