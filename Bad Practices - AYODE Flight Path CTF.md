
# Bad Practices

## Challenge Description

> During a routine security audit of the SkyTrack ground terminal, the
> security team flagged an encrypted archive left behind by the aircraft
> maintenance crew.
> 
> The file is locked tight with password protection, and the crew left
> no notes on their desk. Good luck getting into it.
> 
> Download maintenance.zip to begin the investigation.

I was given a password-protected ZIP archive named `maintenance.zip`, which was password protected...

![](https://gcdnb.pbrd.co/images/J3yb4bdENecs.png)

## First Steps

Not gonna lie, I didn't really know what the description was talking about or if it could've helped me on this challenge

I originally thought you had to brute force the password by using tools like hashcat or john, but I tried a bunch of wordlists and it gave no results

After some research, I found that legacy ZipCrypto encryption of zip files were vulnerable to **known-plaintext attack**

I used 7zip to check the encryption on the given ZIP file and lo and behold, it was encrypted with **ZipCrypto** rather than modern AES encryption.

![](https://img2link.epictech29999.workers.dev/image/c2b98fqw568zf77arz7lp9/0)

I also had some information about the flag format: I knew the flag started with:

```text
flightPath{
```

This gave me known plaintext that could be used to attack the encrypted file.

## Using bkcrack

I used the tool [`bkcrack`](https://github.com/kimci86/bkcrack), to exploit the vulnerability.

Since I knew the beginning of the plaintext, I could use it as the known plaintext for the attack.

I first ran `bkcrack` against the encrypted ZIP using the known `flightPath{` bytes.

    bkcrack.exe -C maintenance.zip -c flag.txt -x 0 666C69676874506174687B

(`666C69676874506174687B` is `flightPath{` in bytes)

After around 10 minutes the attack successfully recovered the internal ZipCrypto keys:

```text
e0db5ef8 eef6fd48 d9490129
```

These are the three 32-bit internal keys used by ZipCrypto.

## Decrypting the ZIP

With the recovered keys, I could then use `bkcrack` to decrypt the archive.

    bkcrack.exe -C maintenance.zip -k e0db5ef8 eef6fd48 d9490129 -D decrypted.zip

After decrypting the file, I extracted the contents and found the flag! :D

## Flag

```text
flightPath{r34d_th3_c0mm3nt_f13ld}
```

## Stuff I learned!
A cool thing I learned from this chall was that ZipCrypto encrypted Zip files were vulnerable to a **known-plaintext attack**

It was important that the archive used **legacy ZipCrypto** to encrypt and I knew part of the plaintext (`flightPath{`)

Using bkcrack, this made the ZIP password itself unnecessary to recover.

If you know a specific filetype in the zip file, you can easily use that file header to crack the zip!

Testing this on some files, it seems that 7zip automatically defaults to this 0-0 (something I will note in the future)

(P.S, looking at the flag I may have overcomplicated it, but still nice to learn something new!)

Pretty nice challenge! (also my first writup) :)
