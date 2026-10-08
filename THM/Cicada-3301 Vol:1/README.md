# Cicada-3301 Vol:1

<img width="1901" height="376" alt="image" src="https://github.com/user-attachments/assets/71e58ebc-5a85-4092-9db3-b6c881718be9" />


> [!note]
> Hello, We are looking for highly intelligent
> 
> individuals. To find them, we have devised a test.
> 
> There is a message hidden in this image
> 
> Download and unzip the folder given to begin
> 
> Good Luck
> 
> -3301


---

<details>
  <summary>Analyze The Audio</summary>


## when load audio on audacity found qrcode 

<img width="1919" height="812" alt="image" src="https://github.com/user-attachments/assets/f989d901-5d41-4581-9501-f0e6d88f82e1" />


# after sometime playing filters i got it  

<img width="1913" height="816" alt="image" src="https://github.com/user-attachments/assets/1a5cad07-f71e-4b8b-825e-895d86a48928" />

## i scanned it and found pastebin link


<img width="1639" height="600" alt="image" src="https://github.com/user-attachments/assets/54b81745-999b-4600-8e85-3f1989f0aa16" />

```
https://pastebin.com/wphPq0Aa
```

<img width="1642" height="602" alt="image" src="https://github.com/user-attachments/assets/7327d89f-b74e-4cf9-a8ce-3cf1f085f161" />



  
</details>


<details>
  <summary>Decode the Passphrase</summary>



```
Passphrase: SG01Ul80X1A0NTVtaHA0NTMh
Key: Q2ljYWRh
```

## simple base64

```
Passphrase: Hm5R_4_P455mhp453!
Key: Cicada
```

## now we need to Find and use a cipher along with the key to decipher the passphrase

> ## let's try `Vigenère Cipher`

<img width="1864" height="571" alt="image" src="https://github.com/user-attachments/assets/f2f693bf-62de-4d33-b5d7-18da1273515d" />

```
Ju5T_4_P455phr453!
```
  
</details>



<details>
  <summary>Gather Metadata</summary>


> Use Steganography tools to gather metadata from Welcome.jpg as well as 

> find the hidden message inside of the image file.



## let's try with stighide 

```
steghide --extract -sf welcome.jpg 
```

<img width="1608" height="289" alt="image" src="https://github.com/user-attachments/assets/31ec2c46-005a-416c-9156-a9b8246c17cf" />

```
https://imgur.com/a/c0ZSZga
```


  
</details>



<details>
  <summary>Find Hidden Files</summary>


<img width="1919" height="745" alt="image" src="https://github.com/user-attachments/assets/6945c83f-eebb-4612-a146-7116ef2871d5" />


## i stuck here but i found tool call `outguess`

```
outguess -r "undefined - Imgur.jpg" outputmessage
cat outputmessage
```

<img width="1450" height="645" alt="image" src="https://github.com/user-attachments/assets/eb26333d-260e-45c0-9467-29267175d24a" />

```ruby

Reading undefined - Imgur.jpg....
Extracting usable bits:   29035 bits
Steg retrieve: seed: 38, len: 1351
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA1

Welcome again.

Here is a book code.  To find the book, break this hash:

b6a233fb9b2d8772b636ab581169b58c98bd4b8df25e452911ef75561df649edc8852846e81837136840f3aa453e83d86323082d5b6002a16bc20c1560828348

Use positive integers to go forward in the text use negative integers to go backwards in the text.

I:1:6
I:2:15
I:3:26
I:5:4 
I:6:15
I:10:26
/
/
I:13:5
I:13:1
I:14:7
I:3:29
I:19:8 
I:22:25
/
I:23:-1
I:19:-1
I:2:21
I:5:9
I:24:-2
I:22:1 
I:38:1


Good luck.

3301

-----BEGIN PGP SIGNATURE-----
Version: GnuPG v1.4.11 (GNU/Linux)

iQIcBAEBAgAGBQJQ5QoZAAoJEBgfAeV6NQkPf2IQAKWgwI5EC33Hzje+YfeaLf6m
sLKjpc2Go98BWGReikDLS4PpkjX962L4Q3TZyzGenjJSUAEcyoHVINbqvK1sMvE5
9lBPmsdBMDPreA8oAZ3cbwtI3QuOFi3tY2qI5sJ7GSfUgiuI6FVVYTU/iXhXbHtL
boY4Sql5y7GaZ65cmH0eA6/418d9KL3Qq3qkTcM/tRAHhOZFMZfT42nsbcvZ2sWi
YyrAT5C+gs53YhODxEY0T9M2fam5AgUIWrMQa3oTRHSoNAefrDuOE7YtPy40j7kk
5/5RztmAzeEdRd8QS1ktHMezXEhdDP/DEdIJCLT5eA27VnTY4+x1Ag9tsDFuitY4
2kEaVtCrf/36JAAwEcwOg2B/stdjXe10RHFStY0N9wQdReW3yAOBohvtOubicbYY
mSCS1Bx91z7uYOo2QwtRaxNs69beSSy+oWBef4uTir8Q6WmgJpmzgmeG7ttEHquj
69CLSOWOm6Yc6qixsZy7ZkYDrSVrPwpAZdEXip7OHST5QE/Rd1M8RWCOODba16Lu
URKvgl0/nZumrPQYbB1roxAaCMtlMoIOvwcyldO0iOQ/2iD4Y0L4sTL7ojq2UYwX
bCotrhYv1srzBIOh+8vuBhV9ROnf/gab4tJII063EmztkBJ+HLfst0qZFAPHQG22
41kaNgYIYeikTrweFqSK
=Ybd6
-----END PGP SIGNATURE-----


```

1- A hash to crack — to identify the book

2- A book code — using the format I:line:word



```
echo "b6a233fb9b2d8772b636ab581169b58c98bd4b8df25e452911ef75561df649edc8852846e81837136840f3aa453e83d86323082d5b6002a16bc20c1560828348" > hash.txt
hashid hash.txt
```


<img width="1414" height="279" alt="image" src="https://github.com/user-attachments/assets/5bbe6f52-cbe5-4a1c-aff5-8120ba19974f" />


  
</details>


<details>
  <summary>Book Cipher</summary>


## after crack it in `https://md5hashing.net/`

<img width="707" height="316" alt="image" src="https://github.com/user-attachments/assets/a3d323ce-a04c-4f82-a278-066515c0fa8b" />

```
https://pastebin.com/6FNiVLh5
```

<img width="1919" height="842" alt="image" src="https://github.com/user-attachments/assets/57146bc9-abdf-4c17-8015-585a072a1269" />




### The Book Code

From the PGP message:

```text

I:1:6
I:2:15
I:3:26
I:5:4
I:6:15
I:10:26
/
/
I:13:5
I:13:1
I:14:7
I:3:29
I:19:8
I:22:25
/
I:23:-1
I:19:-1
I:2:21
I:5:9
I:24:-2
I:22:1
I:38:1

```

### 🔑 Decoding Rules

The format is `I:line:character` — meaning **Chapter I, line number, character position**.

**Critical rules from the original puzzle:**

- **Spaces are NOT counted** as characters
    
- **Punctuation IS counted** (dashes, periods, etc.)
    
- **Negative numbers** count backwards from the end of the line


---

```
https://bit.ly/39pw2NH
```

<img width="1919" height="710" alt="image" src="https://github.com/user-attachments/assets/46685319-0503-4dbb-80f4-135d83961b61" />




  
</details>





<details>
  <summary>The Final Song</summary>


<img width="1919" height="710" alt="image" src="https://github.com/user-attachments/assets/ce08ea9d-8799-43f9-96fe-f3cfd7adc6ab" />


```
The Instar Emergence
```
  
</details>































