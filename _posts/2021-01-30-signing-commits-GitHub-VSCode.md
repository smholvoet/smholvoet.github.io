---
published: false
title: "Signing commits in GitHub & VS Code"
excerpt: "Signing commits in GitHub & VS Code"
date: 2021-01-30T00:00:00-04:00
show_date: true
tags:
  - Git
  - commit
  - signed
  - security
  - GitHub
  - VS Code
---

## Why *should* you be signing commits? 🤔

If you've landed on this blog post looking *how* to use signed commits... you probably already know the motivation behind signing commits and/or [tags](https://git-scm.com/book/en/v2/Git-Basics-Tagging).
The 2 main advantages which I've distilled from the official [Git documentation](https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work):

- Verify that commits are from a trusted source
- Signing your own work, preventing others from pretending to be you

> Git is cryptographically secure, but it’s not foolproof. If you’re taking work from others on the internet and want to verify that commits are actually from a trusted source, Git has a few ways to sign and verify work using GPG.

As you can see, we'll need *GnuPG*, commonly referred to as [**GPG**](https://gnupg.org/). I'll provide 2 different methods to get things working:

- [Method 1](#Method 1: GPG (CLI)): via CLI, using the [GPG command line tools](https://www.gnupg.org/download/)
- [Method 2](# Method 1: GPG (CLI)):via a GUI frontend, called [Gpg4win](https://www.gpg4win.org/download.html)

## Method 1: GPG (CLI)

### Installation

Download & install [GnuPG](https://www.gnupg.org/download/)

Verify succesful installation by running

```terminal
gpg --version
```

Example output:

```terminal
gpg (GnuPG) 2.2.27
libgcrypt 1.8.7
Copyright (C) 2021 g10 Code GmbH
License GNU GPL-3.0-or-later <https://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: C:/Users/Sander/AppData/Roaming/gnupg
Supported algorithms:
Pubkey: RSA, ELG, DSA, ECDH, ECDSA, EDDSA
Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
        CAMELLIA128, CAMELLIA192, CAMELLIA256
Hash: SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
Compression: Uncompressed, ZIP, ZLIB, BZIP2
```

### Create a key

Generate a new key:

```terminal
gpg --full-generate-key
```

The following prompt will show up:

```terminal
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
  (14) Existing key from card
```

👉 You'll want `RSA and RSA` here, so accept the default value by pressing `Enter`.

Next up, the key size:

```terminal
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072)
```

👉 As you can see, the default key size is `3072` bits. Enter `4096`

Set the length of time the key should be valid. Let's set it to never expire by pressing `Enter`:

```terminal
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)
```

You'll then see the following warning, simply enter `y`

```terminal
Key does not expire at all
Is this correct? (y/N)
```

Next up, enter the following information:

- Real name: Test User
- Email address: test@example.com
- Comment: you can leave this blank

```terminal
GnuPG needs to construct a user ID to identiy your key.

Real name: Test User
Email address: test@example.org
Comment: 
You selected this USER-ID:
    "Test User <test@example.org>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit?
```

Real name:

## Method 2

## GitHub configuration

## VS Code configuration

## Try it out
