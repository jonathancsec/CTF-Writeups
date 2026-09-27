# Bad Practices

## Challenge Description

`During a routine security audit of the SkyTrack ground terminal, the security team flagged an encrypted archive left behind by the aircraft maintenance crew.

The file is locked tight with password protection, and the crew left no notes on their desk. Good luck getting into it.

Download maintenance.zip to begin the investigation. `

We were given a password-protected ZIP archive named `maintenance.zip`.

The goal was to recover the contents of the archive and obtain the flag.

## Identifying the Encryption

First, I checked the ZIP file and noticed that it was using **legacy ZipCrypto encryption** rather than modern AES encryption.

This was important because ZipCrypto has known weaknesses that allow a **known-plaintext attack**.

I also had some information about the flag format: I knew the flag started with:

```text
flightPath{
```

This gave me known plaintext that could be used to attack the encrypted file.

## Using bkcrack

I used [`bkcrack`](https://github.com/kimci86/bkcrack), a tool designed to exploit weaknesses in legacy ZIP encryption.

Since I knew the beginning of the plaintext, I could use it as the known plaintext for the attack.

I first ran `bkcrack` against the encrypted ZIP using the known `flightPath{` bytes.

The attack successfully recovered the internal ZipCrypto keys:

```text
e0db5ef8 eef6fd48 d9490129
```

These are the three 32-bit internal keys used by ZipCrypto.

## Decrypting the ZIP

With the recovered keys, I could then use `bkcrack` to decrypt the archive.

After decrypting the file, I extracted the contents and found the flag.

## Flag

```text
flightPath{...}
```

## Takeaways

The important part of this challenge was recognizing that **ZipCrypto is vulnerable to known-plaintext attacks**.

The key pieces of information were:

* The archive used **legacy ZipCrypto**
* I knew part of the plaintext (`flightPath{`)
* `bkcrack` can recover the internal ZipCrypto keys from known plaintext
* The recovered keys can then be used to decrypt the archive

This made the ZIP password itself unnecessary to recover.
