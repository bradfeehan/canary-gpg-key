# Canary Mail rejects passphrase when key missing main secret subkey

```shell
# Access the keyring in this repo
export GNUPGHOME="${PWD}/gnupg"
```

This key has two keys with Sign capability, but I've deleted the first signing
subkey (I don't want Canary to have THIS signing subkey, but I do give it A
signing subkey).

I send `secret.canary.asc` via iTunes to Canary, import the key successfully.
I can associate it with my personal e-mail address, e.g. brad.feehan@gmail.com.
Then when I send an e-mail from that address to itself, it tries to encrypt and
sign using the key. It asks for the passphrase, and I enter it (`test`) but it
says "Invalid Passphrase".

I first ran into this issue when trying to decrypt an incoming mail, and there
is only one Encrypt subkey in the key, and it is included in what's sent to
Canary Mail. So I'd expect that to work.

GPG will let me sign with the second one when the first is missing like this.
It will also let me decrypt as long as the Encrypt secret subkey is there.
But Canary iOS gives this Invalid Passphrase error in both those situations.

This becomes an issue when using a dedicated offline machine for the master
signing key, with dedicated signing sub-keys for each machine.

My setup is loosely based on https://github.com/drduh/YubiKey-Guide for
reference.

Reproduction
------------

Here's how I created the contents of this repo:

```
$ export GNUPGHOME="${PWD}/gnupg"
$ gpg --full-gen-key --expert

# - type: 8 (RSA -- set your own capabilities)
#    Sign Certify only (not Encrypt)
#    enter "E" then "Q"
# - length: 4096
# - expiry: none
# - name: Test User
# - email: test@example.com
# - passphrase: test



$ gpg --expert --edit-key test@example.com

# addkey
# - type: 8 (RSA -- set your own capabilities)
#    Encrypt only
#    enter "S", "C", then "Q"
# - length: 4096
# - expiry: none
#
# addkey
# - type: 4 (RSA -- sign only)
# - length: 4096
# - expiry: none
#
# save



$ gpg -K --with-keygrip
------------------------------------------------------------------
sec   rsa4096/0xBBDEFDEAC7D38B3F 2023-01-30 [SC]
      548CD624685322428EABA8C2BBDEFDEAC7D38B3F
      Keygrip = 9812E5AE97A549FEBD22E307C426017A50331B94
uid                   [ultimate] Test User <test@example.com>
ssb   rsa4096/0xA259EEC72A506291 2023-01-30 [E]
      8F14AD32E791A4A2E88C8952A259EEC72A506291
      Keygrip = C228B573480DF9F0D9B08F3C1B165EFBE7785E5B
ssb   rsa4096/0x180B88BFFA91B781 2023-01-30 [S]
      A5AF127A2D1DE812EE77365D180B88BFFA91B781
      Keygrip = BD8EF89E4E740B0D824151C89FFA8E57A2857DC7



$ rm gnupg/private-keys-v1.d/9812E5AE97A549FEBD22E307C426017A50331B94.key



$ gpg -K
[...]
sec#  rsa4096/0xBBDEFDEAC7D38B3F 2023-01-30 [SC]
[...]



$ gpg --export-secret-subkeys --armor > secret.canary.asc
```