
# Cyber Security Lab 1
## Secret Key Cryptography

## Lab Description
This lab aims to understand fundamental algorithms of cryptography, as well as appropriate analytical methods to study and break weakly encrypted messages.

## key Components and Tools
- Cryptool: a handy tool to get a better understanding of the complex world of cryptography. 

## Task 1 Ciphertext-only Analysis - Brute-force Weak AES
Cryptanalysis based on knowledge of the ciphertext only, i.e. without knowing any part of the plaintext, is the most difficult type of analysis. This task provided a file called Subst-Rijndael.crypt, which contains the binary data of a text that has been encrypted twice, using monoalphabetic substitution first and then the AES algorithm in CBC mode. The AES encryption used a weak key, where from the 128 bit key only the first 16 bits were chosen and the rest of the key was padded with zeros. Here, the initialization vector (IV) is the first block of the ciphertext. This task requires, using brute-force attack on Subst-Rijndael.crypt becomes, to break the outer AES encryption layer with a short prototype script to automate the attack. The decryption shall be computed and stored in Subst.txt (in plaintext format) for further processing in the next task. 

## Task 2 Hill-Climbing Method to Break Monalphabetic Substitution
This tasks continue to break the inner monoalphabetic substitution, a naive brute-force attack would require to test up to 26! ≈ 4 · 1026 possible permutations of the alphabet (assuming the alphabet A-Z only). Assuming each probe takes just 1 μs, approximately 1013 years would be required and you would likely miss the deadline for this task. Although frequency analysis can help us guess the scheme of the monoalphabetic substitution quickly, there is no guarantee that the key gained from this is 100% correct. 

## Task 3 Search the Right Algorithm 
The objective of this task is to determine if a given ciphertext can produce two distinct plaintexts using the **Vernam cipher**. The ciphertext is stored in `cipher.crypt`, while the possible plaintexts are located in `plaintext1.txt` and `plaintext2.txt`. The aim is to identify two keys, k1 and k2, such that:
- Decrypting `cipher.crypt` with k1 results in plaintext1.
- Decrypting `cipher.crypt` with k2 results in plaintext2.

## Task 4 Partial Known Plain-text Analysis
This analysis utilizes a partial known plain-text attack, specifically targeting the 11th and 12th bytes of a 96-bit XOR key, which remain unknown. The strategy involves a brute-force method to explore all possible values for these two bytes while employing the known ZIP header for version 3.0. 

XOR algorithm is used to encrypt the zip file using a 96-bit-long XOR key
Brute Force approach is used to try out all possible values of the unknown part of the XOR key (the 11th and 12th bytes) 
Linux command 'zip -t' is used to verify the decrypted zip file is correctly decrypted

The ZIP header for version 3.0 contains several following fields (first 12 bytes):

- **Signature**: `0x50, 0x4B, 0x03, 0x04`
- **Version Needed to Extract**: `0x1E, 0x00` (version 3.0)
- **General Purpose Bit Flag**: `0x00, 0x00` (no comment)
- **Compression Method**: `0x08, 0x00` (Deflate - default compression)
- **Last Mod File Time**: `0x00, 0x00` (unknown)

During the analysis, the first 10 bytes of the ZIP header are known and can be XORed with the corresponding bytes from the encrypted data (`XOR.zip.crypt`) to recover these fixed parts of the XOR key. The remaining lies in determining the values of the 11th and 12th bytes. a brute-force approach is employed to try through all possible pairs of values (0-255) for the 11th and 12th bytes. For each combination, the XOR key is used to decrypt the entire encrypted ZIP file. Once decrypted, a `zip -t` command is executed to verify whether the resulting file can be unzipped successfully.

Upon finding a valid combination that allows for successful extraction, the corresponding XOR key is saved as `XOR.key`, and the decrypted ZIP file is saved as `decrypted.zip`.

## Task 5 Mathematics of Hill-Cipher
This task explore how a 2×2 Hill cipher key was calculated using partial known plaintext and ciphertext. The ciphertext is:
VBIDUXANLFPYPUSFGPIDTYUHDY

It is known that the plaintext begins with:
THE FOX

Spaces are omitted, so the actual plaintext used is:
THEFOX

The goal is to recover the encryption key and decrypt the full ciphertext.
