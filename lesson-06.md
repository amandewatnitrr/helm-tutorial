# Chart Security

![](./imgs/Isometric_manuscript_feature_image_v1.1_gyxvrs.png)

- When we talk about chart security there are 2 parties involved, the Chart Provider, and the user of the chart who will take the chart and do the installations, and upgrades using the helm command line.

- When it comes to security there are 2 important things that you should note of, as the user of the chart, we want to ensure the chart is really from the provider we are expecting it from.

  Secondly, the integrity of the chart, once the provider puts the chart in the repository, it should not be modified or tweaked by any third unknown person.

- To avoid these, helm has in-built support for provenance and integrity. It uses `PGP`, which stands for `Pretty Good Privacy`.

- As a provider of the chart we will start by generating a keypair, private key and a public key. And, when we use the `helm package` command, we'll ask the helm package command to use the private key and generate a `.prov` message or file. 

- So, along with the pakcaged chart, we'll also have a `.prov` file once we start using PGP with the helm pakcage command. Both, the packaged chart and the `.prov` file will be pushed into the repository. This `.prov` file will have the information about the chart, and it will also have a calculated signature.

  The chart data whatever we have in the package chart, will be used to calculate this signature.

- Now, when the user starts using the chart, he will be having the public key,the provider of the chart will share the public key with the end user using the public key.

- tghe user can verify the chart using the `helm install --verify` command. These command will use this public key calculate signature on chart and both the signatures, and they should match.

### PGP Key Installation

- To sign oue charts, we need a key pair, private key and a public key. The easiest way to generate these is to use the open source tool called `GNUPG`(`GNU Privacy Guard`).

- If you are on mac use the command `brew install gnupg`.

- To check the `gnupg` version use the command:

  ```shell
  $ gpg --version
  gpg (GnuPG) 2.4.7
  libgcrypt 1.11.0-unknown
  Copyright (C) 2024 g10 Code GmbH
  License GNU GPL-3.0-or-later <https://gnu.org/licenses/gpl.html>
  This is free software: you are free to change and redistribute it.
  There is NO WARRANTY, to the extent permitted by law.
  
  Home: /Users/akd/.gnupg
  Supported algorithms:
  Pubkey: RSA, ELG, DSA, ECDH, ECDSA, EDDSA
  Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
  CAMELLIA128, CAMELLIA192, CAMELLIA256
  Hash: SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
  Compression: Uncompressed, ZIP, ZLIB, BZIP2
  ```

### Generating PGP Keys

- Use the following command to generate the keys:

  ```shell
  $ gpg --full-generate-key
  gpg (GnuPG) 2.4.7; Copyright (C) 2024 g10 Code GmbH
  This is free software: you are free to change and redistribute it.
  There is NO WARRANTY, to the extent permitted by law.
  
  Please select what kind of key you want:
     (1) RSA and RSA
     (2) DSA and Elgamal                                                                                (3) DSA (sign only)
     (4) RSA (sign only)                                                                                (9) ECC (sign and encrypt) *default*
    (10) ECC (sign only)
    (14) Existing key from card                                                                      Your selection? 1
  RSA keys may be between 1024 and 4096 bits long.                                                   What keysize do you want? (3072)
  Requested keysize is 3072 bits
  Please specify how long the key should be valid.
           0 = key does not expire
        <n>  = key expires in n days
        <n>w = key expires in n weeks
        <n>m = key expires in n months
        <n>y = key expires in n years
  Key is valid for? (0) 0
  Key does not expire at all
  Is this correct? (y/N) y
  
  GnuPG needs to construct a user ID to identify your key.
                                                                                                     Real name: test-chart-aman
  Email address: amandewatnitrr@gmail.com
  Comment: For tutorial
  You selected this USER-ID:
      "test-chart-aman (For tutorial) <amandewatnitrr@gmail.com>"
  
  Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
  We need to generate a lot of random bytes. It is a good idea to perform
  some other action (type on the keyboard, move the mouse, utilize the
  disks) during the prime generation; this gives the random number                                   generator a better chance to gain enough entropy.
  We need to generate a lot of random bytes. It is a good idea to perform
  some other action (type on the keyboard, move the mouse, utilize the
  disks) during the prime generation; this gives the random number                                   generator a better chance to gain enough entropy.
  gpg: directory '/Users/akd/.gnupg/openpgp-revocs.d' created
  gpg: revocation certificate stored as '/Users/akd/.gnupg/openpgp-revocs.d/CXXXXXXXXXXXXXXXXXXXXXXX.rev'
  public and secret key created and signed.
  
  pub   rsa3072 2025-06-08 [SC]
        XXXXXXXXXXXXXXXXXXXXXXXX
  uid                      test-chart-aman (For tutorial) <something@gmail.com>
  sub   rsa3072 2025-06-08 [E]
  ```
  
- Run the command, to export the keys to a specified file name of your choice.

  ```shell
  $ gpg --export-secret-keys > ~/.gnupg//akd-secret.gpg
  ```
  
- Now, if you check you will see our file in the `.gnupg`.

  ```shell
  $ ls -a                                                        ✔  at 22:50:12 
  .                   openpgp-revocs.d    S.gpg-agent.browser S.scdaemon
  ..                  private-keys-v1.d   S.gpg-agent.extra   trustdb.gpg
  akd-secret.gpg      public-keys.d       S.gpg-agent.ssh
  common.conf         S.gpg-agent         S.keyboxd
  ```
  
### Signing the charts

- To sign the chart with the `gpg` keys use the command:

  ```shell
  $ helm package --sign --key amandewatnitrr@gmail.com --keyring ~/.gnupg/akd-secret.gpg test-chart -d .
  Successfully packaged chart and saved it to: /Users/akd/Github/helm-tutorial/experiments/test-chart/test-chart-0.1.0.tgz
  ```
  
- And, we can verify the chart using the command:

  ```shell
  $ helm verify test-chart-0.1.0.tgz --keyring ~/.gnupg/akd-secret.gpg
  Signed by: test-chart-aman (For tutorial) <amandewatnitrr@gmail.com>
  Using Key With Fingerprint: XXXXXXXXXXXXXXXXXXXXXXXB4BFD18B575857
  Chart Hash Verified: sha256:2de5XXXXXXXXXXXXXXXXX42481XXXXXXXXXX
  ```


