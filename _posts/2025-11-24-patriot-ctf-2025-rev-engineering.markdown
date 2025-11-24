---
layout: post
title:  "Patriot 2025 Reverse Engineering"
date:   2025-11-24 17:09:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /patriot-ctf-2025-reverse-engineering/
---
* TOC
{:toc}

## Space Pirates

**Category**: Reverse Engineering

**Challenge author**: caffix

**Description**: You've intercepted an encrypted transmission from space pirates! Decode their secret coordinates to find their hidden treasure.

I get the challenge file `challenge.c`:
```shell
#include <stdio.h>  
#include <string.h>  
#include <stdint.h>  
  
/*  
* SPACE PIRATES CTF CHALLENGE  
* ===========================  
* Theme: You've intercepted an encrypted transmission from space pirates!  
* Decode their secret coordinates to find their hidden treasure.  
*    
* CHALLENGE MECHANICS:  
* The input undergoes a series of reversible operations:  
* 1. XOR with a rotating key (reversible by XORing again)  
* 2. Byte swap pairs (reversible by swapping again)  
* 3. Addition modulo 256 (reversible by subtraction)  
* 4. Final XOR with position (reversible by XORing again)  
*    
*/  
  
#define FLAG_LEN 30  
const uint8_t TARGET[FLAG_LEN] = {  
   0x5A,0x3D,0x5B,0x9C,0x98,0x73,0xAE,0x32,0x25,0x47,0x48,0x51,0x6C,0x71,0x3A,0x62,0xB8,0x7B,0x63,0x57,0x25,0x89,0x58,0xBF,0x78,0x34,0x98,0x71,0x68,0x59  
};  
  
// The pirate's rotating XOR key  
const uint8_t XOR_KEY[5] = {0x42, 0x73, 0x21, 0x69, 0x37};  
  
// The magic addition constant  
const uint8_t MAGIC_ADD = 0x2A;  
// PCTF{0x_M4rks_tH3_sp0t_M4t3y}  
void print_flag(char *input) {  
   printf("\n");  
   printf("    *     .    *       .   *    .\n");  
   printf("  .   *        ___---___      *    .\n");  
   printf("    *    .   .'         '.   *   .\n");  
   printf("  .  *   *   /   0     0   \\    *\n");  
   printf("    *   .   |               |  .    *\n");  
   printf("  .    *    |   \\  ___  /   |     .\n");  
   printf("    *    .  |    \\_____/    |  *\n");  
   printf("  *   .      \\             /    .    *\n");  
   printf("    .    *    '._________.'   *   .\n");  
   printf("  *    .   *    TREASURE!   .   *\n");  
   printf("\n");  
   printf("🏴‍☠  DECRYPTION SUCCESSFUL! 🏴‍☠\n");  
   printf("\n");  
   printf("Flag: %s\n",input);  
   printf("\n");  
}  
  
int main(int argc, char *argv[]) {  
   printf("===========================================\n");  
   printf("  SPACE PIRATES TRANSMISSION DECODER v1.0\n");  
   printf("===========================================\n");  
   printf("\n");  
   printf("Mission: Decode the pirate coordinates!\n");  
   printf("\n");  
  
   // Check command line argument  
   if (argc != 2) {  
       printf("  Usage: %s <encrypted_coordinates>\n", argv[0]);  
       printf("   Example: %s GALAXY_SECTOR_ALPHA_1234567\n", argv[0]);  
       return 1;  
   }  
  
   char *input = argv[1];  
   size_t len = strlen(input);  
  
   // Validate length  
   if (len != FLAG_LEN) {  
       printf("  Invalid transmission length!\n");  
       return 1;  
   }  
  
   // Create a working buffer  
   uint8_t buffer[FLAG_LEN];  
   memcpy(buffer, input, FLAG_LEN);  
  
   printf("Processing transmission...\n\n");  
  
   // OPERATION 1: XOR with rotating key  
   // Each byte is XORed with one of 5 key bytes (cycling through them)  
   // This obscures the plaintext with a repeating pattern  
   printf("[1/4] Applying quantum entanglement cipher...\n");  
   for (int i = 0; i < FLAG_LEN; i++) {  
       buffer[i] ^= XOR_KEY[i % 5];  
   }  
  
   // OPERATION 2: Swap adjacent byte pairs  
   // Bytes at positions (0,1), (2,3), (4,5), etc. are swapped  
   // This scrambles the byte order in a predictable way  
   printf("[2/4] Applying spatial transposition...\n");  
   for (int i = 0; i < FLAG_LEN; i += 2) {  
       uint8_t temp = buffer[i];  
       buffer[i] = buffer[i + 1];  
       buffer[i + 1] = temp;  
   }  
  
   // OPERATION 3: Add magic constant (mod 256)  
   // Each byte has MAGIC_ADD added to it (wrapping at 256)  
   // This shifts all values by a constant amount  
   printf("[3/4] Applying gravitational shift...\n");  
   for (int i = 0; i < FLAG_LEN; i++) {  
       buffer[i] = (buffer[i] + MAGIC_ADD) % 256;  
   }  
  
   // OPERATION 4: XOR each byte with its position  
   // Byte at position i is XORed with i  
   // This makes each position's transformation unique  
   printf("[4/4] Applying coordinate calibration...\n");  
   for (int i = 0; i < FLAG_LEN; i++) {  
       buffer[i] ^= i;  
   }  
  
   printf("\n");  
   printf("Verifying coordinates against star charts...\n");  
      
   // Check if the result matches our target  
   if (memcmp(buffer, TARGET, FLAG_LEN) == 0) {  
       print_flag(input);  
       return 0;  
   } else {  
       printf("\n");  
       printf("  DECRYPTION FAILED!\n");  
       printf("\n");  
       printf("The coordinates don't match known pirate sectors.\n");  
       printf("Hint: Pirates love to name their sectors after galaxies...\n");  
       printf("\n");  
          
       // Debug output (helpful for solving)  
       // printf("Your result (hex): ");  
       // for (int i = 0; i < FLAG_LEN; i++) {  
       //     printf("0x%02X,", buffer[i]);  
       // }  
       printf("\n");  
          
       return 1;  
   }  
}
```

Use python to reverse the operations in the code:
```python
TARGET = [0x5A,0x3D,0x5B,0x9C,0x98,0x73,0xAE,0x32,0x25,0x47,0x48,0x51,0x6C,0x71,0x3A,0x62,0xB8,0x7B,0x63,0x57,0x25,0x89,0x58,0xBF,0x78,0x34,0x98,0x71,0x68,0x59]
XOR_KEY = [0x42, 0x73, 0x21, 0x69, 0x37]
MAGIC_ADD = 0x2A
FLAG_LEN = 30

buffer = TARGET.copy()

# operation 4: XOR each byte with its position
for i in range(FLAG_LEN):
    buffer[i] ^= i

# operation 3: subtract magic constant (mod 256)
for i in range(FLAG_LEN):
    buffer[i] = (buffer[i] - MAGIC_ADD) % 256

# operation 2: Swap adjacent byte pairs
for i in range(0, FLAG_LEN, 2):
    buffer[i], buffer[i+1] = buffer[i+1], buffer[i]

# operation 1: XOR with rotating key
for i in range(FLAG_LEN):
    buffer[i] ^= XOR_KEY[i % 5]

flag = ''.join(chr(b) for b in buffer)
print(flag)
```

Output:
```shell
python3 solve.py  
PCTF{0x_M4rks_tH3_sp0t_M4t3ys}
```

`PCTF{0x_M4rks_tH3_sp0t_M4t3ys}`

___

## Are You Pylingual?

**Category**: Reverse Engineering

**Challenge author**: DJ Strigel

**Description**: I found this weird binary in my mom's computer. I'm not sure what it even is or why it was there. I heard it was put there from a past member of MasonCC. He did some obfuscation to make his Python code unreadable. Reverse engineer the program and see if anything of interest lies in the output. The flag format will be pctf{flag}

I get the challenge files `pylinguese.pyc` and `output.txt`:
```shell
file pylinguese.pyc  
pylinguese.pyc: Byte-compiled Python module for CPython 3.12 or newer, timestamp-based, .py timestamp: Sat Sep  6 18:41:22 2025 UTC, .py size: 681 bytes

cat output.txt  
output = [-90, -42, -39, -42, -39, -39, -39, -42, -39, -42, -39, -42, -39, -42, -39, -90, -90, -39, -48, -13, -52, -39, -42, -39, -42, -39, -42, -39, -42, -90, -42, -39, -42, -39, -90, -90, -42, -39, -39, -39, -39, -42, -39, -90, -90, -39, -39, -42, -105, -90, -90, -42, -39, -39, -91, -90, -90, -39, -91, -39, -42, -39, -42, -39, -39, -39, -39, -42, -39, -42, -39, -39, -39, -42, -39, -42, -34, -39, -39, -42, -39, -42, -39, -42, -39, -42, -39, -90... (shortened the output is very long)
```

I went to [https://www.pylingual.io/](https://www.pylingual.io/) and decompiled the `.pyc` file:
```python
# Decompiled with PyLingual (https://pylingual.io)
# Internal filename: pylinguese.py
# Bytecode version: 3.12.0rc2 (3531)
# Source timestamp: 2025-09-06 18:41:22 UTC (1757184082)

import pyfiglet
file = open('flag.txt', 'r')
flag = file.read()
font = 'slant'
words = 'MASONCC IS THE BEST CLUB EVER'
flag_track = 0
art = list(pyfiglet.figlet_format(words, font=font))
i = len(art) % 10
for ind in range(len(art)):
	if ind == i and flag_track < len(flag):
		art[ind] = flag[flag_track]
		i += 28
		flag_track += 1
art_str = ''.join(art)
first_val = 5
second_val = 6
first_half = art_str[:len(art_str) // 2]
second_half = art_str[len(art_str) // 2:]
first = [~ord(char) ^ first_val for char in first_half]
second = [~ord(char) ^ second_val for char in second_half]
output = second + first
print(output)
```

Reverse the operations done in this script to get the flag:
```python
import pyfiglet

output = [-90, -42, -39, -42, -39, -39, -39, -42, -39, -42, -39, -42, -39, -42, -39, -90, -90, -39, -48, -13, -52, -39, -42, -39, -42, -39, -42, -39, -42, -90, -42, -39, -42, -39, -90, -90, -42, -39, -39, -39, -39, -42, -39, -90, -90, -39, -39, -42, -105, -90, -90, -42, -39, -39, -91, -90, -90, -39, -91, -39, -42, -39, -42, -39, -39, -39, -39, -42, -39, -42, -39, -39, -39, -42, -39, -42, -34, -39, -39, -42, -39, -42, -39, -42, -39, -42, -39, -90, -90, -39, -39, -123, -13, -39, -42, -39, -42, -39, -42, -39, -90, -90, -39, -39, -115, -39, -42, -90, -90, -90, -39, -39, -39, -42, -39, -42, -90, -42, -39, -42, -39, -42, -90, -90, -90, -39, -90, -90, -90, -42, -39, -42, -90, -39, -42, -39, -39, -39, -39, -42, -39, -42, -90, -90, -90, -42, -39, -42, -90, -90, -90, -42, -39, -42, -90, -42, -39, -42, -39, -42, -68, -42, -39, -42, -39, -13, -42... # shortened output


half_len = len(output) // 2
second = output[:half_len]
first = output[half_len:]

first_half = ''.join([chr(~(val ^ 5) & 0xFF) for val in first])
second_half = ''.join([chr(~(val ^ 6) & 0xFF) for val in second])

art_str = first_half + second_half

# generate original ascii art to find flag positions
words = 'MASONCC IS THE BEST CLUB EVER'
font = 'slant'
original_art = pyfiglet.figlet_format(words, font=font)

# find where the flag characters were inserted
i = len(original_art) % 10
flag_chars = []
for ind in range(len(art_str)):
    if ind == i:
        flag_chars.append(art_str[ind])
        i += 28
        if len(flag_chars) >= len(art_str) // 28:
            break

flag = ''.join(flag_chars)
print(f"{flag}")
```

Output:
```shell
python3 solve2.py  
pctf{obFusc4ti0n_i5n't_EncRypt1oN}
```

`pctf{obFusc4ti0n_i5n't_EncRypt1oN}`

___

## Space Pirates 2

**Category**: Reverse Engineering

**Challenge author**: caffix

**Description**: You decoded the coordinates and found the pirates' hidden base. Now you've discovered their treasure map, but it's encrypted with an even MORE complex cipher. The pirates learned from their mistake and upgraded their security!

I get the challenge file `main.rs`:
```shell
/*  
* SPACE PIRATES CTF CHALLENGE - LEVEL 2: THE TREASURE MAP  
* ========================================================  
* You decoded the coordinates and found the pirates'    
* hidden base. Now you've discovered their treasure map, but it's encrypted    
* with an even MORE complex cipher. The pirates learned from their mistake    
* and upgraded their security!  
*    
*/  
  
use std::env;  
use std::process;  
  
// The target encrypted treasure map (what we want the transformed input to become)  
const TARGET: [u8; 32] = [  
   0x15, 0x5A, 0xAC, 0xF6, 0x36, 0x22, 0x3B, 0x52, 0x6C, 0x4F, 0x90, 0xD9, 0x35, 0x63, 0xF8, 0x0E, 0x02, 0x33, 0xB0, 0xF1, 0xB7, 0x69, 0x42, 0x67, 0x25, 0xEA, 0x96, 0x63, 0x1  
B, 0xA7, 0x03, 0x0B  
];  
  
// The pirate's NEW rotating XOR key (upgraded security!)  
const XOR_KEY: [u8; 5] = [0x7E, 0x33, 0x91, 0x4C, 0xA5];  
  
// NEW: Rotation amounts for each position modulo 7  
const ROTATION_PATTERN: [u32; 7] = [1, 3, 5, 7, 2, 4, 6];  
  
// The magic subtraction constant (they switched from addition!)  
const MAGIC_SUB: u8 = 0x5D;  
  
//input: &str  
fn print_flag(buffer: &str) {  
   println!("\n╔══════════════════════════════════════════╗");  
   println!("║                                          ║");  
   println!("║     _______________                      ║");  
   println!("║    /               \\                     ║");  
   println!("║   /          \\                    ║");  
   println!("║  |            |                   ║");  
   println!("║  |   TREASURE!   |                   ║");  
   println!("║  |            |                   ║");  
   println!("║   \\          /                    ║");  
   println!("║    \\_______________ /                    ║");  
   println!("║         |     |                          ║");  
   println!("║         |_____|                          ║");  
   println!("║                                          ║");  
   println!("╚══════════════════════════════════════════╝");  
   println!("\n🏴‍☠  LEVEL 2 COMPLETE! YOU FOUND THE TREASURE! 🏴‍☠\n");  
   println!("The pirates' secret hoard is yours!\n");  
   println!("Flag: {0}\n", buffer);  
   println!("You've mastered the advanced pirate cipher!");  
   println!("The treasure contains riches beyond imagination!\n");  
}  
  
/// Rotate a byte left by n positions  
/// This is a bijection because all 8 bit rotations of a byte are unique  
/// ROL(ROL(x, n), 8-n) = x, proving invertibility  
fn rotate_left(byte: u8, n: u32) -> u8 {  
   byte.rotate_left(n % 8)  
}  
  
/// OPERATION 1: XOR with NEW rotating key  
/// Each byte is XORed with one of 5 NEW key bytes (cycling through them)  
/// Bijection proof: (x ⊕ k) ⊕ k = x (XOR involution)  
fn apply_quantum_cipher_v2(buffer: &mut [u8]) {  
   for (i, byte) in buffer.iter_mut().enumerate() {  
       *byte ^= XOR_KEY[i % 5];  
   }  
}  
  
/// OPERATION 2 (NEW!): Rotate Left with varying amounts  
/// Each byte is rotated left by an amount determined by its position  
/// Bijection proof: ROL⁻¹ = ROR with same amount  
/// The rotation amount varies: position mod 7 selects from ROTATION_PATTERN  
fn apply_stellar_rotation(buffer: &mut [u8]) {  
   for (i, byte) in buffer.iter_mut().enumerate() {  
       let rotation = ROTATION_PATTERN[i % 7];  
       *byte = rotate_left(*byte, rotation);  
   }  
}  
  
/// OPERATION 3: Swap adjacent byte pairs  
/// Bytes at positions (0,1), (2,3), (4,5), etc. are swapped  
/// Bijection proof: Swapping twice returns original (f ∘ f = identity)  
fn apply_spatial_transposition(buffer: &mut [u8]) {  
   for i in (0..buffer.len()).step_by(2) {  
       buffer.swap(i, i + 1);  
   }  
}  
  
/// OPERATION 4: Subtract magic constant (mod 256) - CHANGED FROM ADDITION!  
/// Each byte has MAGIC_SUB subtracted from it (wrapping at 256)  
/// Bijection proof: (x - k) + k ≡ x (mod 256)  
/// Subtraction forms a group, every element has unique inverse  
fn apply_gravitational_shift_v2(buffer: &mut [u8]) {  
   for byte in buffer.iter_mut() {  
       *byte = byte.wrapping_sub(MAGIC_SUB);  
   }  
}  
  
/// OPERATION 5 (NEW!): Reverse bytes in chunks of 5  
/// Splits the 30-byte buffer into 6 chunks of 5, reverses each chunk  
/// Chunk 0: [0,1,2,3,4] -> [4,3,2,1,0]  
/// Chunk 1: [5,6,7,8,9] -> [9,8,7,6,5], etc.  
/// Bijection proof: Reversal is self-inverse, f(f(x)) = x  
fn apply_temporal_inversion(buffer: &mut [u8]) {  
   const CHUNK_SIZE: usize = 5;  
   for chunk_start in (0..buffer.len()).step_by(CHUNK_SIZE) {  
       let chunk_end = (chunk_start + CHUNK_SIZE).min(buffer.len());  
       buffer[chunk_start..chunk_end].reverse();  
   }  
}  
  
/// OPERATION 6 (NEW!): XOR each byte with its position SQUARED (mod 256)  
/// Byte at position i is XORed with i² mod 256  
/// Bijection proof: (x ⊕ k) ⊕ k = x (XOR involution)  
/// While i² grows, mod 256 keeps values in range, and XOR remains invertible  
fn apply_coordinate_calibration_v2(buffer: &mut [u8]) {  
   for (i, byte) in buffer.iter_mut().enumerate() {  
       // Square the position and take mod 256 to keep it in u8 range  
       let position_squared = ((i * i) % 256) as u8;  
       *byte ^= position_squared;  
   }  
}  
  
fn process_transmission(input: &str) -> Result<[u8; 32], String> {  
   // Validate length  
   if input.len() != 32 {  
       return Err(format!(  
           "Invalid treasure map length!\n   Got {} bytes, expected 32 bytes\n   Hint: Treasure maps have specific coordinate formats",  
           input.len()  
       ));  
   }  
  
   // Convert to byte array  
   let mut buffer = [0u8; 32];  
   buffer.copy_from_slice(input.as_bytes());  
  
   println!("Decrypting treasure map...\n");  
  
   println!("[1/6] Applying advanced quantum entanglement cipher...");  
   apply_quantum_cipher_v2(&mut buffer);  
  
   println!("[2/6] Applying stellar rotation transformation...");  
   apply_stellar_rotation(&mut buffer);  
  
   println!("[3/6] Applying spatial transposition...");  
   apply_spatial_transposition(&mut buffer);  
  
   println!("[4/6] Applying inverse gravitational shift...");  
   apply_gravitational_shift_v2(&mut buffer);  
  
   println!("[5/6] Applying temporal inversion...");  
   apply_temporal_inversion(&mut buffer);  
  
   println!("[6/6] Applying advanced coordinate calibration...");  
   apply_coordinate_calibration_v2(&mut buffer);  
  
   Ok(buffer)  
}  
  
fn main() {  
   println!("╔═══════════════════════════════════════════╗");  
   println!("║  SPACE PIRATES TREASURE MAP DECODER v2.0  ║");  
   println!("║              LEVEL 2: ADVANCED            ║");  
   println!("╚═══════════════════════════════════════════╝");  
   println!("\n Welcome back, treasure hunter!");  
   println!("You cracked the coordinates, but now you need");  
   println!("to decrypt the actual TREASURE MAP!");  
   println!("\n  WARNING: The pirates upgraded their cipher!");  
  
   // Get command line arguments  
   let args: Vec<String> = env::args().collect();  
  
   if args.len() != 2 {  
       eprintln!(" Usage: {} <encrypted_treasure_map>", args[0]);  
       eprintln!("   Example: {} TREASURE_MAP_ALPHA_12345678901", args[0]);  
       process::exit(1);  
   }  
  
   let input = &args[1];  
  
   // Process the transmission  
   match process_transmission(input) {  
       Ok(buffer) => {  
           println!("\nVerifying treasure map authenticity...");  
  
           // Check if the result matches our target  
           if buffer == TARGET {  
               print_flag(input);  
           } else {  
               println!("\n DECRYPTION FAILED!\n");  
               println!("This doesn't match any known treasure map format.");  
               println!("Hint: Pirates use numeric codes mixed with text...\n");  
  
               // Debug output (helpful for solving)  
               // print!("Your result (hex): ");  
               // for byte in buffer.iter() {  
               //     print!("0x{:02X}, ", byte);  
               // }  
               println!("\n");  
               process::exit(1);  
           }  
       }  
       Err(err) => {  
           eprintln!(" {}", err);  
           process::exit(1);  
       }  
   }  
}
```

I run this python script which reverses the operations of the rust code:
```python
TARGET = bytes([0x15, 0x5A, 0xAC, 0xF6, 0x36, 0x22, 0x3B, 0x52, 0x6C, 0x4F, 0x90, 0xD9, 0x35, 0x63, 0xF8, 0x0E, 0x02, 0x33, 0xB0, 0xF1, 0xB7, 0x69, 0x42, 0x67, 0x25, 0xEA, 0x96, 0x63, 0x1B, 0xA7, 0x03, 0x0B])

XOR_KEY = [0x7E, 0x33, 0x91, 0x4C, 0xA5]
ROTATION_PATTERN = [1, 3, 5, 7, 2, 4, 6]
MAGIC_SUB = 0x5D

def rotate_right(byte, n):
    n = n % 8
    return ((byte >> n) | (byte << (8 - n))) & 0xFF

def decrypt(buffer):
    buffer = bytearray(buffer)
    
    # operation 6: XOR each byte with its position squared
    for i in range(len(buffer)):
        position_squared = ((i * i) % 256)
        buffer[i] ^= position_squared
    
    # operation 5: reverse bytes in chunks of 5 
    CHUNK_SIZE = 5
    for chunk_start in range(0, len(buffer), CHUNK_SIZE):
        chunk_end = min(chunk_start + CHUNK_SIZE, len(buffer))
        buffer[chunk_start:chunk_end] = reversed(buffer[chunk_start:chunk_end])
    
    # operation 4: add magic constant
    for i in range(len(buffer)):
        buffer[i] = (buffer[i] + MAGIC_SUB) & 0xFF
    
    # operation 3: Swap adjacent byte pairs
    for i in range(0, len(buffer), 2):
        if i + 1 < len(buffer):
            buffer[i], buffer[i + 1] = buffer[i + 1], buffer[i]
    
    # operation 2: rotate right
    for i in range(len(buffer)):
        rotation = ROTATION_PATTERN[i % 7]
        buffer[i] = rotate_right(buffer[i], rotation)
    
    # operation 1: XOR with rotating key
    for i in range(len(buffer)):
        buffer[i] ^= XOR_KEY[i % 5]
    
    return bytes(buffer)

def main():
    decrypted = decrypt(TARGET)
    
    try:
        flag = decrypted.decode('ascii')
        print(f"{flag}")
    except:
        print(f"Could not decode as ASCII :( hex: {decrypted.hex()}")

if __name__ == "__main__":
    main()
```

Output:
```shell
python3 solve3.py  
PCTF{Y0U_F0UND_TH3_P1R4T3_B00TY}
```

`PCTF{Y0U_F0UND_TH3_P1R4T3_B00TY}`

___

## Space Pirates 3

**Category**: Reverse Engineering

**Challenge author**: caffix

**Description**: Incredible work! You found the treasure, but wait... there's a note: "This be but a fraction of me fortune! The REAL hoard lies in me secret vault, protected by the most devious cipher ever created by pirate-kind. Only the cleverest of sea dogs can crack it. - Captain Blackbyte"

I get the challenge file `space_pirates3.go`:
```shell
/*  
* SPACE PIRATES CTF CHALLENGE - LEVEL 3: THE PIRATE KING'S VAULT  
* ===============================================================  
* You found the treasure, but wait... there's a note:  
* "This be but a fraction of me fortune! The REAL hoard lies in me secret vault,  
* protected by the most devious cipher ever created by pirate-kind. Only the    
* cleverest of sea dogs can crack it. - Captain Blackbyte"  
*    
* The Pirate King has combined the same 6 operations BUT with completely  
* different keys and parameters. This is the ultimate test!  
*    
*/  
  
package main  
  
import (  
       "fmt"  
       "os"  
)  
  
// The target encrypted vault combination (what we want the transformed input to become)  
var target = [30]byte{  
       0x60, 0x6D, 0x5D, 0x97, 0x2C, 0x04, 0xAF, 0x7C, 0xE2, 0x9E, 0x77, 0x85, 0xD1, 0x0F, 0x1D, 0x17, 0xD4, 0x30, 0xB7, 0x48, 0xDC, 0x48, 0x36, 0xC1, 0xCA, 0x28, 0xE1, 0x37, 0x58, 0x0F,  
}  
  
// The Pirate King's ULTIMATE XOR key (7 bytes - prime number for better mixing!)  
var xorKey = [7]byte{0xC7, 0x2E, 0x89, 0x51, 0xB4, 0x6D, 0x1F}  
  
// NEW: Rotation pattern (8 bytes, includes rotation by 0 which is identity)  
var rotationPattern = [8]uint{7, 5, 3, 1, 6, 4, 2, 0}  
  
// The Pirate King's subtraction constant (much larger than before!)  
const magicSub byte = 0x93  
  
// Chunk size for reversal (changed from 5 to 6!)  
const chunkSize = 6  
  
func printFlag(input string) {  
       fmt.Println()  
       fmt.Println("╔═══════════════════════════════════════════════════╗")  
       fmt.Println("║                                                   ║")  
       fmt.Println("║           ⚜  THE PIRATE KING'S VAULT  ⚜          ║")  
       fmt.Println("║                                                   ║")  
       fmt.Println("║              _.-^^^^`````````````^^-.             ║")  
       fmt.Println("║         _.-`^`                       `^-._        ║")  
       fmt.Println("║      ,-`       ⚜    ⚜        `-,     ║")  
       fmt.Println("║     /          ⚜     ⚜           \\    ║")  
       fmt.Println("║    /          ⚜    ⚜    ⚜          \\   ║")  
       fmt.Println("║   |      ⚜  THE ULTIMATE HOARD ⚜      |  ║")  
       fmt.Println("║    \\          ⚜    ⚜    ⚜          /   ║")  
       fmt.Println("║     \\          ⚜     ⚜           /    ║")  
       fmt.Println("║      `-,       ⚜    ⚜        ,-`     ║")  
       fmt.Println("║         `-._                       _,-`          ║")  
       fmt.Println("║             `--..___________..--`                ║")  
       fmt.Println("║                                                   ║")  
       fmt.Println("╚═══════════════════════════════════════════════════╝")  
       fmt.Println()  
       fmt.Println("🏴‍☠ ⚔  LEVEL 3 COMPLETE! MASTER OF THE SEVEN SEAS! ⚔  🏴‍☠")  
       fmt.Println()  
       fmt.Println("Ye've cracked the Pirate King's most devious cipher!")  
       fmt.Println("The greatest treasure known to pirate-kind is yours!")  
       fmt.Println()  
       fmt.Println("Flag: ", input)  
       fmt.Println()  
       fmt.Println()  
}  
  
// rotateLeft rotates a byte left by n positions  
// Bijection: ROL has inverse ROR  
// Even rotation by 0 (identity) is bijective: ROL(x,0) = x, ROR(x,0) = x  
func rotateLeft(b byte, n uint) byte {  
       n = n % 8 // Ensure n is in range [0,7]  
       return (b << n) | (b >> (8 - n))  
}  
  
// OPERATION 1: XOR with rotating key (7 bytes now for better coverage)  
// Bijection: (x ⊕ k) ⊕ k = x  
// Longer key doesn't affect bijectivity, just cycling pattern  
func applyUltimateQuantumCipher(buffer []byte) {  
       for i := range buffer {  
               buffer[i] ^= xorKey[i%len(xorKey)]  
       }  
}  
  
// OPERATION 2: Rotate Left with new pattern (includes rotation by 0)  
// Bijection: ROL⁻¹ = ROR with same amount  
// Rotation by 0 at position 7 is identity but still bijective  
func applyStellarRotationV2(buffer []byte) {  
       for i := range buffer {  
               rotation := rotationPattern[i%len(rotationPattern)]  
               buffer[i] = rotateLeft(buffer[i], rotation)  
       }  
}  
  
// OPERATION 3: Swap adjacent byte pairs  
// Bijection: Self-inverse, f(f(x)) = x  
func applySpatialTransposition(buffer []byte) {  
       for i := 0; i < len(buffer)-1; i += 2 {  
               buffer[i], buffer[i+1] = buffer[i+1], buffer[i]  
       }  
}  
  
// OPERATION 4: Subtract magic constant (NEW VALUE: 0x93)  
// Bijection: (x - k) + k ≡ x (mod 256)  
// Larger constant increases difficulty but not mathematical properties  
func applyGravitationalShiftV3(buffer []byte) {  
       for i := range buffer {  
               buffer[i] -= magicSub  
       }  
}  
  
// OPERATION 5: Reverse bytes in chunks of 6 (CHANGED from 5!)  
// Splits 30 bytes into 5 chunks of 6 bytes each  
// Bijection: Reversal is self-inverse, f(f(x)) = x  
// 30 = 6 * 5, so all bytes are processed (no remainder)  
func applyTemporalInversionV2(buffer []byte) {  
       for chunkStart := 0; chunkStart < len(buffer); chunkStart += chunkSize {  
               chunkEnd := chunkStart + chunkSize  
               if chunkEnd > len(buffer) {  
                       chunkEnd = len(buffer)  
               }  
               // Reverse the chunk in place  
               for i, j := chunkStart, chunkEnd-1; i < j; i, j = i+1, j-1 {  
                       buffer[i], buffer[j] = buffer[j], buffer[i]  
               }  
       }  
}  
  
// OPERATION 6: XOR each byte with (position² + position) mod 256  
// Enhanced from Level 2's position²  
// Bijection: (x ⊕ k) ⊕ k = x (XOR involution)  
// The function (i² + i) mod 256 is deterministic and unique per position  
func applyCoordinateCalibrationV3(buffer []byte) {  
       for i := range buffer {  
               // Calculate i² + i mod 256  
               positionValue := ((i * i) + i) % 256  
               buffer[i] ^= byte(positionValue)  
       }  
}  
  
func processVault(input string) ([30]byte, error) {  
       var result [30]byte  
  
       // Validate length  
       if len(input) != 30 {  
               return result, fmt.Errorf(  
                       "Invalid vault combination",  
               )  
       }  
  
       // Convert to byte array  
       copy(result[:], input)  
       buffer := result[:] // Create slice for easier manipulation  
  
       fmt.Println("Decrypting the Pirate King's vault...\n")  
  
       fmt.Println("[1/6] Applying ultimate quantum entanglement cipher...")  
       applyUltimateQuantumCipher(buffer)  
  
       fmt.Println("[2/6] Applying advanced stellar rotation...")  
       applyStellarRotationV2(buffer)  
  
       fmt.Println("[3/6] Applying spatial transposition...")  
       applySpatialTransposition(buffer)  
  
       fmt.Println("[4/6] Applying extreme gravitational shift...")  
       applyGravitationalShiftV3(buffer)  
  
       fmt.Println("[5/6] Applying enhanced temporal inversion...")  
       applyTemporalInversionV2(buffer)  
  
       fmt.Println("[6/6] Applying master coordinate calibration...")  
       applyCoordinateCalibrationV3(buffer)  
  
       return result, nil  
}  
  
func main() {  
       fmt.Println("╔════════════════════════════════════════════════════╗")  
       fmt.Println("║   PIRATE KING'S VAULT DECODER v3.0 - MASTER LEVEL  ║")  
       fmt.Println("║                LEVEL 3: ULTIMATE                   ║")  
       fmt.Println("╚════════════════════════════════════════════════════╝")  
       fmt.Println()  
       fmt.Println()  
       fmt.Println("You've found the treasure, but Captain Blackbyte")  
       fmt.Println("left a note: 'Me REAL fortune lies in me secret vault!'")  
       fmt.Println()  
  
       // Check arguments  
       if len(os.Args) != 2 {  
               fmt.Fprintf(os.Stderr, "  Usage: %s <vault_combination>\n", os.Args[0])  
               os.Exit(1)  
       }  
  
       input := os.Args[1]  
  
       // Process the vault combination  
       result, err := processVault(input)  
       if err != nil {  
               fmt.Fprintf(os.Stderr, " %v\n", err)  
               os.Exit(1)  
       }  
  
       fmt.Println("\nVerifying vault combination against the master lock...")  
  
       // Check if result matches target  
       if result == target {  
               printFlag(input)  
       } else {  
               fmt.Println("\nVAULT REMAINS SEALED!\n")  
               fmt.Println("The combination doesn't match the Pirate King's lock.")  
  
               // Debug output  
               // fmt.Print("Your result (hex): ")  
               // for _, b := range result {  
               //      fmt.Printf("0x%02X, ", b)  
               // }  
               fmt.Println("\n")  
               os.Exit(1)  
       }  
}
```

I run this python script to reverse the operations and get the flag:
```python
TARGET = bytes([0x60, 0x6D, 0x5D, 0x97, 0x2C, 0x04, 0xAF, 0x7C, 0xE2, 0x9E, 0x77, 0x85, 0xD1, 0x0F, 0x1D, 0x17, 0xD4, 0x30, 0xB7, 0x48, 0xDC, 0x48, 0x36, 0xC1, 0xCA, 0x28, 0xE1, 0x37, 0x58, 0x0F])

XOR_KEY = [0xC7, 0x2E, 0x89, 0x51, 0xB4, 0x6D, 0x1F]
ROTATION_PATTERN = [7, 5, 3, 1, 6, 4, 2, 0]
MAGIC_SUB = 0x93
CHUNK_SIZE = 6

def rotate_right(byte, n):
    n = n % 8
    return ((byte >> n) | (byte << (8 - n))) & 0xFF

def decrypt(buffer):
    buffer = bytearray(buffer)
    
    # operation 6: XOR with (position squared + position) mod 256
    for i in range(len(buffer)):
        position_value = ((i * i) + i) % 256
        buffer[i] ^= position_value
    
    # operation 5: reverse bytes into chunks of 6
    for chunk_start in range(0, len(buffer), CHUNK_SIZE):
        chunk_end = min(chunk_start + CHUNK_SIZE, len(buffer))
        buffer[chunk_start:chunk_end] = reversed(buffer[chunk_start:chunk_end])
    
    # operation 4: add magic constant
    for i in range(len(buffer)):
        buffer[i] = (buffer[i] + MAGIC_SUB) & 0xFF
    
    # operation 3: swap adjacent byte pairs
    for i in range(0, len(buffer) - 1, 2):
        buffer[i], buffer[i + 1] = buffer[i + 1], buffer[i]
    
    # operation 2: rotate each byte right
    for i in range(len(buffer)):
        rotation = ROTATION_PATTERN[i % len(ROTATION_PATTERN)]
        buffer[i] = rotate_right(buffer[i], rotation)
    
    # operation 1: XOR with rotating key
    for i in range(len(buffer)):
        buffer[i] ^= XOR_KEY[i % len(XOR_KEY)]
    
    return bytes(buffer)

def main():
    decrypted = decrypt(TARGET)
    
    try:
        flag = decrypted.decode('ascii')
        print(f"{flag}")
    except:
        print(f"Could not decode as ASCII :( hex: {decrypted.hex()}")

if __name__ == "__main__":
    main()
```

Output:
```shell
python3 solve4.py  
PCTF{M4ST3R_0F_TH3_S3V3N_S34S}
```

`PCTF{M4ST3R_0F_TH3_S3V3N_S34S}`

___

## Vorpal Masters

**Category**: Reverse Engineering

**Challenge author**: Matthew Johnson (CACI)

**Description**: I'm excited to announce the launch of my brand new video game Vorpal Masters. In light of recent pirating attacks, I've implemented a copy protection by requiring a license key when you launch the game. Before I publish the game to Steam, I just want to have someone test it to make sure it's as uncrackable as I think it is. The flag is the license key, put into the format `CACI{Key}`.

I get a binary:
```shell
file license  
license: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=6f5355f7386afc8a69ef3a8c448622b4d46aea62, for GNU/Linux 3.2.0, not stripped
```

I run it:
```shell
./license  
Welcome to {insert game here}  
Please enter the license key from the 3rd page of the booklet.  
test    
Please enter you key in the format xxxx-xxxx-xxxx
```

After opening it in ghidra I see this as the main function:
```shell
void main(void)

{
  int iVar1;
  int local_20;
  char local_1c [11];
  char local_11;
  char local_10;
  char local_f;
  char local_e;
  int local_c;
  
  puts(
      "Welcome to {insert game here}\nPlease enter the license key from the 3rd page of the booklet. "
      );
  local_c = __isoc99_scanf("%4s-%d-%10s",&local_11,&local_20,local_1c);
  if (local_c != 3) {
    puts("Please enter you key in the format xxxx-xxxx-xxxx");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  if ((((local_11 != 'C') || (local_f != 'C')) || (local_e != 'I')) || (local_10 != 'A')) {
    womp_womp();
  }
  if ((-0x1389 < local_20) && (local_20 < 0x2711)) {
    if ((local_20 + 0x16) % 0x6ca == ((local_20 * 2) % 2000) * 6 + 9) goto LAB_00101286;
  }
  womp_womp();
LAB_00101286:
  iVar1 = strcmp(local_1c,"PatriotCTF");
  if (iVar1 != 0) {
    womp_womp();
  }
  puts("Lisence key registered, you may play the game now!");
  return;
}
```

The first 4 characters of the license (`CACI`) are hardcoded. 

The second part (`2025`) is done with this math: `(local_20 + 0x16) % 0x6ca == ((local_20 * 2) % 2000) * 6 + 9)`.

The last 10 characters must be `PatriotCTF`: `strcmp(local_1c,"PatriotCTF");`:
```shell
./license  
Welcome to {insert game here}  
Please enter the license key from the 3rd page of the booklet.  
CACI-2025-PatriotCTF  
Lisence key registered, you may play the game now!
```

`CACI{CACI-2025-PatriotCTF}`

___

## ReadMyNote

**Category**: Reverse Engineering

**Challenge author**: x_ref

**Description**: A fun little walk in the woods! Everything you need is located within the binary! The binary was poorly obfuscated with Qengine 2.0! It can be solved either dynamically or statically. The binary has a static base! Flag format is pctf{...}

I get the challenge file `ReadMyNote.exe`:
```shell
file ReadMyNote.exe  
ReadMyNote.exe: PE32+ executable (console) x86-64, for MS Windows, 5 sections
```

I ran `strings` on the binary as usual just to see and I thought I saw an encoded string:
```shell
strings -n 8 ReadMyNote.exe
...
MEMORY_ALTERATION  
THREAD_VIOLATION  
ACCESS_VIOLATION  
HOOK_DETECTED  
FN_HASH_CORRUPT  
Hello kind traveler! I have something you seek. But first i need something in return.    
Could you remind me what year it is? All this time spent here, and I've forgotten :(  
Thank you! I think i wrote it down somewhere .... Ahh, here it is  
The note reads:    
vector too long  
unordered_map/set too long  
invalid hash bucket count  
ufqc~LZI5S6ZR4KA5R!Z=6g3a=`2x  
00000000000000005555555555555555@  
C:\Users\legob\source\repos\ConsoleApplication1\x64\Release\ConsoleApplication1.pdb  
.text$di  
...
```

I tested XOR keys on the string and found the flag:
```shell
python3  
Python 3.11.2 (main, Apr 28 2025, 14:11:48) [GCC 12.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> encoded = "ufqc~LZI5S6ZR4KA5R!Z=6g3a=`2x"  
>>> decoded = "".join(chr(ord(c) ^ 5) for c in encoded)  
>>> print(decoded)  
pctf{I_L0V3_W1ND0W$_83b6d8e7}  
>>> exit()
```

`pctf{I_L0V3_W1ND0W$_83b6d8e7}`

___
