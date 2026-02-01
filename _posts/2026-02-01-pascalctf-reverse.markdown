---
layout: post
title:  "Pascal 2026 Reverse Challenges"
date:   2026-02-01 09:13:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /pascalctf-2026-reverse-challenges/
---
* TOC
{:toc}

### AuraTester2000

**Category**: Reverse

**Author**: Enea Maroncelli `@Zazaman`

**Description**: Will you be able to gain enough aura? `nc auratester.ctf.pascalctf.it 7001`

I get the challenge file `AuraTester2000.gyat`:
```shell
glaze random ahh sonopazzo  
glaze os ahh palle  
  
words = ["tungtung","trallalero","filippo boschi","zaza","lakaka","gubbio","cucinato"]  
  
phrase = " ".join(sonopazzo.sample(words,k=sonopazzo.randint(3, 5)))  
  
steps = sonopazzo.randint(2, 5)  
flag = palle.getenv("FLAG", "pascalCTF{REDACTED}")  
bop encoder(phrase, steps):  
   encoded_phrase = ""  
   mewing i in huzz(0,len(phrase)):  
  
       chat is this real phrase[i] twin " ":  
           encoded_phrase rizz= phrase[i]  
  
       yo chat i% steps twin 0:  
           encoded_phrase rizz= str(ord(phrase[i]))  
  
       only in ohio:  
           encoded_phrase rizz= phrase[i]  
  
  
   its giving encoded_phrase  
bop questions(name):  
   gained_aura = 0  
   questions = [  
       "Do you believe in the power of aura? (yes/no)",  
       "Do you a JerkMate account? (yes/no)",  
       "Are you willing to embrace your inner alpha? (yes/no)",  
       "Do you really like SHYNE from Travis Scott? (yes/no)",  
   ]  
   aura_values = [(150,-50), (-1000,50),(450,-80),(-100,50)]  
   mewing i in huzz(len(questions)):  
       print(f"{name}, {questions[i]}")  
       answer = input("> ").strip().lower()  
       chat is this real answer twin "yes":  
           gained_aura rizz= aura_values[i][0]  
       yo chat answer twin "no":  
           gained_aura rizz=  aura_values[i][1]  
   its giving gained_aura  
  
bop aura_test(name):  
   yap(f"{name}, you have reached the final AuraTest!")  
   yap("If you want to win your prize you need to decode this secret phrase:",encoder(phrase, steps))  
  
   guess = input("Type the decoded phrase to prove your worth:\n> ")  
   chat is this real guess twin phrase:  
       yap(f"Congratulations {name}! You have proven your worth and gained the ultimate aura!\nHere's your price:\n{flag}")  
       exit()  
   only in ohio:  
       yap(f"Dont waste my time {name}, you failed the AuraTest. Try again but this time use all your aura!")    
  
yap("Welcome to the AuraTester2000!\nHere, we will make sure you have enough aura to join our alpha gang.")  
  
let him cook(Aura):  
   name = input("First of all, we need to know your name.\n> ")  
   chat is this real(name.strip() twin ""):  
       yap("You didn't start very well, I asked your named stupid npc.")  
   only in ohio:  
       yap(f"Welcome {name} to the AuraTester2000!")  
       yap("""⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀  
       ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣸⣦⣀⠀⠀⢀⣴⣷⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀  
       ⠀⠀⠀⠀⠀⢀⠀⠀⠀⠀⢀⣿⡿⠟⠛⠛⠻⢿⣿⡄⠀⠀⠀⠀⢀⠀⠀⠀⠀⠀  
       ⠀⠀⠀⠀⠀⠈⢷⣦⣤⣤⡾⠋⠀⣴⣾⣷⣦⠀⠙⢿⣦⣤⣤⣾⠃⠀⠀⠀⠀⠀  
       ⠀⠀⠀⠀⠀⠀⠘⣿⣿⠟⠀⠀⢸⣿⣿⣿⣿⡇⠀⠀⠹⣿⣿⡏⠀⠀⠀⠀⠀⠀  
       ⠀⠀⠀⠀⠀⣀⣴⣿⡏⠀⠀⠀⠘⢿⣿⣿⡿⠃⠀⠀⠀⠹⣿⣷⣀⠀⠀⠀⠀⠀  
       ⠀⠀⠲⣾⣿⣿⣿⣿⠀⠀⠀⢀⣤⣾⣿⣿⣷⣦⡀⠀⠀⠀⢿⣿⣿⣿⣿⠖⠂⠀  
       ⠀⠀⠀⠈⠙⢿⣿⡇⠀⠀⠀⣾⣿⣿⣿⣿⣿⣿⣿⡀⠀⠀⢸⣿⣿⠟⠁⠀⠀⠀  
       ⠀⠀⠀⠀⠀⢨⣿⡇⠀⠀⢰⣿⣿⣿⣿⣿⣿⣿⣿⡇⠀⠀⠘⣿⣏⠀⠀⠀⠀⠀  
       ⠀⠀⠀⢀⣠⣾⣿⡇⠀⠀⢸⣿⣿⣿⣿⣿⣿⣿⣿⡇⠀⠀⢰⣿⣿⣦⡀⠀⠀⠀  
       ⠀⠀⠺⠿⢿⣿⣿⣇⠀⠀⠘⠛⣿⣿⣿⣿⣿⣿⠛⠃⠀⠀⢸⣿⣿⡿⠿⠗⠂⠀  
       ⠀⠀⠀⠀⠀⠈⠻⣿⡀⠀⠀⠀⢹⣿⣿⣿⣿⣿⠀⠀⠀⠀⣿⡿⠉⠀⠀⠀⠀⠀  
       ⠀⠀⠀⠀⠀⠀⢠⣿⣧⠀⠀⠀⢸⣿⣿⣿⣿⡏⠀⠀⠀⣼⣿⡇⠀⠀⠀⠀⠀⠀  
       ⠀⠀⠀⠀⠀⣠⣾⣿⣿⣧⡀⠀⢸⣿⣿⣿⣿⡇⠀⠀⣼⣿⣿⣿⣄⠀⠀⠀⠀⠀  
       ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⠓⠀⠘⠛⠛⠛⠛⠃⠀⠚⠋⠀⠀⠀⠀⠀⠀⠀⠀⠀""")  
       just put the fries in the bag bro  
  
aura = 0  
  
let him cook(Aura):  
   yap("\n\n1. Answer questions to gain or lose aura.\n\n2. Check your current aura.\n\n3. Take the final AuraTest to prove your worth.\n\n4. Exit the AuraTester2000.")  
   choice = input("What do you want to do little Beta?\n> ")  
   chat is this real (choice twin "1"):  
       yap("You choose to answer questions. Let's see how much aura you can gain!")  
       gained_aura = questions(name)  
       chat is this real(aura sigma 0):  
           yap(f"Congratulations {name}! You gained {gained_aura} aura points.")  
       only in ohio:  
           yap(f"Sorry {name}, you lost {gained_aura} aura points. Learn how to be a real Sigma!")  
       aura += gained_aura  
  
   yo chat(choice twin "2"):  
       yap(f"Your current aura is {aura}.")  
   yo chat(choice twin "3"):  
       chat is this real(aura beta 500):  
           yap("You need more aura to even try the final AuraTest.")  
       only in ohio:  
           aura_test(name)  
   yo chat(choice twin "4"):  
       yap("Exiting the AuraTester2000. Goodbye!")  
       exit()  
   only in ohio:  
       yap("Invalid option. Please try again.")
```

I connect to the server and answer some brainrot questions. Once it says I have enough aura I get the encoded message `116u110g116u110g g117b98i111 122a122a c117c105n97t111 108a107a107a`. This decodes into `tungtung gubbio zaza cucinato lakaka` which I send to the server to get the flag:
```shell
nc auratester.ctf.pascalctf.it 7001  
Welcome to the AuraTester2000!  
Here, we will make sure you have enough aura to join our alpha gang.  
First of all, we need to know your name.  
> idk  
Welcome idk to the AuraTester2000!  
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀  
       ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣸⣦⣀⠀⠀⢀⣴⣷⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀  
       ⠀⠀⠀⠀⠀⢀⠀⠀⠀⠀⢀⣿⡿⠟⠛⠛⠻⢿⣿⡄⠀⠀⠀⠀⢀⠀⠀⠀⠀⠀  
       ⠀⠀⠀⠀⠀⠈⢷⣦⣤⣤⡾⠋⠀⣴⣾⣷⣦⠀⠙⢿⣦⣤⣤⣾⠃⠀⠀⠀⠀⠀  
       ⠀⠀⠀⠀⠀⠀⠘⣿⣿⠟⠀⠀⢸⣿⣿⣿⣿⡇⠀⠀⠹⣿⣿⡏⠀⠀⠀⠀⠀⠀  
       ⠀⠀⠀⠀⠀⣀⣴⣿⡏⠀⠀⠀⠘⢿⣿⣿⡿⠃⠀⠀⠀⠹⣿⣷⣀⠀⠀⠀⠀⠀  
       ⠀⠀⠲⣾⣿⣿⣿⣿⠀⠀⠀⢀⣤⣾⣿⣿⣷⣦⡀⠀⠀⠀⢿⣿⣿⣿⣿⠖⠂⠀  
       ⠀⠀⠀⠈⠙⢿⣿⡇⠀⠀⠀⣾⣿⣿⣿⣿⣿⣿⣿⡀⠀⠀⢸⣿⣿⠟⠁⠀⠀⠀  
       ⠀⠀⠀⠀⠀⢨⣿⡇⠀⠀⢰⣿⣿⣿⣿⣿⣿⣿⣿⡇⠀⠀⠘⣿⣏⠀⠀⠀⠀⠀  
       ⠀⠀⠀⢀⣠⣾⣿⡇⠀⠀⢸⣿⣿⣿⣿⣿⣿⣿⣿⡇⠀⠀⢰⣿⣿⣦⡀⠀⠀⠀  
       ⠀⠀⠺⠿⢿⣿⣿⣇⠀⠀⠘⠛⣿⣿⣿⣿⣿⣿⠛⠃⠀⠀⢸⣿⣿⡿⠿⠗⠂⠀  
       ⠀⠀⠀⠀⠀⠈⠻⣿⡀⠀⠀⠀⢹⣿⣿⣿⣿⣿⠀⠀⠀⠀⣿⡿⠉⠀⠀⠀⠀⠀  
       ⠀⠀⠀⠀⠀⠀⢠⣿⣧⠀⠀⠀⢸⣿⣿⣿⣿⡏⠀⠀⠀⣼⣿⡇⠀⠀⠀⠀⠀⠀  
       ⠀⠀⠀⠀⠀⣠⣾⣿⣿⣧⡀⠀⢸⣿⣿⣿⣿⡇⠀⠀⣼⣿⣿⣿⣄⠀⠀⠀⠀⠀  
       ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⠓⠀⠘⠛⠛⠛⠛⠃⠀⠚⠋⠀⠀⠀⠀⠀⠀⠀⠀⠀  
  
  
1. Answer questions to gain or lose aura.  
  
2. Check your current aura.  
  
3. Take the final AuraTest to prove your worth.  
  
4. Exit the AuraTester2000.  
What do you want to do little Beta?  
> 1  
You choose to answer questions. Let's see how much aura you can gain!  
idk, Do you believe in the power of aura? (yes/no)  
> yes  
idk, Do you a JerkMate account? (yes/no)  
> no  
idk, Are you willing to embrace your inner alpha? (yes/no)  
> yes  
idk, Do you really like SHYNE from Travis Scott? (yes/no)  
> no  
Sorry idk, you lost 700 aura points. Learn how to be a real Sigma!  
  
  
5. Answer questions to gain or lose aura.  
  
6. Check your current aura.  
  
7. Take the final AuraTest to prove your worth.  
  
8. Exit the AuraTester2000.  
What do you want to do little Beta?  
> 2  
Your current aura is 700.  
  
  
9. Answer questions to gain or lose aura.  
  
10. Check your current aura.  
  
11. Take the final AuraTest to prove your worth.  
  
12. Exit the AuraTester2000.  
What do you want to do little Beta?  
> 3  
idk, you have reached the final AuraTest!  
If you want to win your prize you need to decode this secret phrase: 116u110g116u110g g117b98i111 122a122a c117c105n97t111 108a107a107a  
Type the decoded phrase to prove your worth:  
> tungtung gubbio zaza cucinato lakaka  
Congratulations idk! You have proven your worth and gained the ultimate aura!  
Here's your price:  
pascalCTF{Y0u_4r3_th3_r34l_4ur4_f1n4l_b0s5}
```

The words were encoded like this:
```python
def encoder(phrase, steps):
    encoded = ""
    for i in range(len(phrase)):
        if phrase[i] == " ":
            encoded += " "
        elif i % steps == 0:
            encoded += str(ord(phrase[i]))
        else:
            encoded += phrase[i]
    return encoded
```
The spaces stay spaces. Every other character is replaced with its ASCII value (`ord(char)`). All other characters are unchanged and concatenate the output. 

`pascalCTF{Y0u_4r3_th3_r34l_4ur4_f1n4l_b0s5}`

___

### Albo delle Eccellenze

**Category**: Reverse

**Author**: Alan Davide Bovo `@AlBovo`

**Description**: One of our former Blaisone CTF Team members 🚩 has just earned a medal in the Cyberchallenge.IT contest 🥳. He’s now wondering whether he also received a prize 🏆, could you help him find out? `nc albo.ctf.pascalctf.it 7004`

I get this file:
```shell
file albo  
albo: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, BuildID[sha1]=b507905c69e8663dce82ccdbb15a8c58873550f8, for GNU/Linux 3.2.0, stripped, too many notes (256)
```

I connected to the server and sent it these random values. I honestly didnt do ANYTHING for this challenge. It simply gave me the flag:
```shell
nc albo.ctf.pascalctf.it 7004  
Welcome into the latest version of  
     Albo delle Eccellenze          
  (PascalCTF Beginners 2026)        
  
Enter your name: idk  
Enter your surname: idk  
Enter your date of birth (DD/MM/YYYY): 10/10/1910  
Enter your sex (M/F): F  
Enter your place of birth: none  
Code matched!  
Here is the flag: pascalCTF{g00d_luck_g3tt1ng_your_pr1zes_n0w}
```

I put the binary into ghidra and there were 126 funtions. Yeah I honestly have no idea why it works. No idea what is going on here. I gave it those random responses and got the flag back. I'll take it I guess.

`pascalCTF{g00d_luck_g3tt1ng_your_pr1zes_n0w}`

___
