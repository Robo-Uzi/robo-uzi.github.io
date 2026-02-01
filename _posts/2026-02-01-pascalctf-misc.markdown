---
layout: post
title:  "Pascal 2026 Misc Challenges"
date:   2026-02-01 09:43:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /pascalctf-2026-misc-challenges/
---
* TOC
{:toc}

### Very Simple Framer

**Category**: Misc

**Author**: Marco Balducci `@Mark-74`

**Description**: I decided to make a simple framer application, obviously with the help of my dear friend, you really think I would write that stuff?

I get the challenge file `chal.py`:
```shell
#!/usr/bin/env python3  
import sys  
import argparse  
from PIL import Image  
  
def message_to_binary(message):  
   return ''.join(format(ord(c), '08b') for c in message)  
  
def generate_border_coordinates(width, height):  
   coords = []  
      
   for x in range(width):  
       coords.append((x, 0))  
      
   for y in range(1, height-1):  
       coords.append((width-1, y))  
      
   if height > 1:  
       for x in range(width-1, -1, -1):  
           coords.append((x, height-1))  
      
   if width > 1:  
       for y in range(height-2, 0, -1):  
           coords.append((0, y))  
      
   return coords  
  
def add_binary_frame(input_image_path, message, output_image_path):  
   img = Image.open(input_image_path)  
   img = img.convert("RGB")  
   orig_width, orig_height = img.size  
  
   binary_str = message_to_binary(message)  
   if not binary_str:  
       raise ValueError("The provided message is empty; cannot create a binary frame.")  
  
   new_width = orig_width + 2  
   new_height = orig_height + 2  
   new_img = Image.new("RGB", (new_width, new_height), "white")  
      
   new_img.paste(img, (1, 1))  
  
   border_coords = generate_border_coordinates(new_width, new_height)  
   border_length = len(border_coords)  
  
   for i, coord in enumerate(border_coords):  
       bit = binary_str[i % len(binary_str)]  
       # '0' becomes black, '1' becomes white.  
       color = (0, 0, 0) if bit == '0' else (255, 255, 255)  
       new_img.putpixel(coord, color)  
      
   new_img.save(output_image_path)  
   print(f"Image saved with binary frame to {output_image_path}")  
  
def main():  
   parser = argparse.ArgumentParser(description="Add a 1px binary frame to an image using a message.")  
   parser.add_argument("input_image", help="Path to the input image file.")  
   parser.add_argument("message", help="Message to convert into a binary frame.")  
   parser.add_argument("output_image", help="Path to save the output image file.")  
   args = parser.parse_args()  
      
   add_binary_frame(args.input_image, args.message, args.output_image)  
  
if __name__ == "__main__":  
   main()  
(venv)
```

We are also given `output.jpg` which has the flag encoded around the border:

![Alt text](/images/output.jpg)

Solve script:
```python
from PIL import Image

def generate_border_coordinates(width, height):
    coords = []
    for x in range(width):
        coords.append((x, 0))
    for y in range(1, height - 1):
        coords.append((width - 1, y))
    for x in range(width - 1, -1, -1):
        coords.append((x, height - 1))
    for y in range(height - 2, 0, -1):
        coords.append((0, y))
    return coords

img = Image.open("output.jpg").convert("RGB")
w, h = img.size

# Convert border to bits using brightness threshold
bits = ""
for x, y in generate_border_coordinates(w, h):
    r, g, b = img.getpixel((x, y))
    brightness = (r + g + b) / 3
    bits += "1" if brightness > 128 else "0"

# Convert all bits to characters
msg = ""
for i in range(0, len(bits), 8):
    byte = bits[i:i+8]
    if len(byte) < 8:
        break
    msg += chr(int(byte, 2))

# Step 3: Print the flag
print(msg)
```

Output:
```shell
python3 solve5.py  
pascalCTF{Wh41t_wh0_4r3_7h0s3_9uy5???}pascalCTF{Wh41t_wh0_4r3_7h0s3_9uy5???}pascalCTF{Wh41t_wh0_4r3_7h0s3_9uy5???}pascalCTF{Wh41t_wh0_4r3_7h0s3_9uy5???}pascalCTF{Wh41t_wh0_4r3_7h0s3_9uy5???}pascalCTF{Wh41t_wh0_4r3_7h0s3_9uy5???}pascalCTF{Wh41t_wh0_4r3_7h0s3_9uy5???}pascalCTF{Wh41t_wh0_4r3_7h0s3_9uy5???}pascalCTF{Wh41t_wh0_4r3_7h0s3_9uy5???}pascalCTF{Wh41t_wh0_4r3_7h0s3_9uy5???}pascalCTF{Wh41t_wh0_4r3_7h0s3_9uy5???}pascalCTF{Wh41t_wh0_4r3_7h0s3_9uy5???}pascalCTF{Wh41t_wh0_4r
```

`pascalCTF{Wh41t_wh0_4r3_7h0s3_9uy5???}`

___

### Stinky Slim

**Category**: Misc

**Author**: Enea Maroncelli `@ZazaMan`

**Description**: I don't trust Patapim; I think he is hiding something from me.

I get the challenge file (`.wav`) audio file:
```shell
file pieno-di-slim.wav  
pieno-di-slim.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, stereo 44100 Hz
```

I put the file into audacity and put it into spectrogram view:

![Alt text](/images/audio-spectrogram-challenge.png)

Better view of message:

![Alt text](/images/spectrogram-better-view.png)

It tells me to open a ticket with the message `I love Blaise Pascal` to get the flag. I do that and they message me back with the flag!

`pascalCTF{th3_k1ng_0f_th3_f0r3st_w1th_s0m3_d1rty_f3et}`

___

### SurgoCompany

**Category**: Misc

**Author**: Fabio Fantini `@fobiathegamer`

**Description**: **SurgoCompany™** has just launched a brand-new online Customer Service platform 🧩. The system is still under development, and you've been asked to test it 😈. Simply enter your email, describe your issue, and optionally upload a file related to your problem 🤨.

The source code of the challenge is saved somewhere on the filesystem... and rumor has it that a file named _flag.txt_ is hiding in that very same folder 😜.

Can you find a way to read it?

Email box: [https://surgo.ctf.pascalctf.it](https://surgo.ctf.pascalctf.it)  
Email account generator: [https://surgoservice.ctf.pascalctf.it](https://surgoservice.ctf.pascalctf.it)

`nc surgobot.ctf.pascalctf.it 6005`

I went to `https://surgoservice.ctf.pascalctf.it/` and generated an email and password. Then go to `https://surgo.ctf.pascalctf.it` and login to roundcube webmail. 

I get the challenge file `src.py` which contains this:
```shell
from email.message import EmailMessage  
from email.utils import parseaddr  
from tempfile import TemporaryDirectory  
import imaplib, smtplib, time  
import email, os, re  
  
# Email configuration  
COMPANY_EMAIL = os.getenv('EMAIL_USERNAME') + "@" + os.getenv('EMAIL_DOMAIN')  
PASSWORD = os.getenv('EMAIL_PASSWORD')  
EMAIL_REGEX = r'user-\w+@%s' % os.getenv('EMAIL_DOMAIN')  
IMAP_SERVER = "mail" # mail.skillissue.it  
SMTP_SERVER = "mail" # mail.skillissue.it  
  
# Polling configuration  
MAX_WAIT = 2 * 60    
INTERVAL = 10    
  
subject_prefix = 'Surgo Company Customer Support - Request no.'  
body = 'Hello dear customer! Thank you for contacting us.\n\nReply to this email describing your problem.\nTo help us better understand your issue, please attach any relevant  
file related to the problem.\n\nThank you for your cooperation!\nBest regards,\nSurgo Company'  
  
def send_email(recipient_address, pid):  
   msg = EmailMessage()  
   msg['From'] = COMPANY_EMAIL  
   msg['To'] = recipient_address  
   msg['Subject'] = subject_prefix + str(pid)  
   msg.set_content(body)  
  
   if COMPANY_EMAIL is None or PASSWORD is None:  
       raise ValueError("Environment error.")  
  
   with smtplib.SMTP_SSL(SMTP_SERVER, 465) as smtp:  
       smtp.login(COMPANY_EMAIL, PASSWORD)  
       smtp.send_message(msg)  
  
def find_email(session, email_ids, sender_address, pid):  
   for email_id in email_ids:  
       _, msg_data = session.uid('fetch', email_id, '(RFC822)')  
  
       # Check, to avoid parsing errors  
       if not msg_data or not isinstance(msg_data[0], tuple) or not isinstance(msg_data[0][1], (bytes, bytearray)):  
           continue  
  
       msg = email.message_from_bytes(msg_data[0][1])  
       current_sender = parseaddr(msg.get("From", ""))[1]  
       current_subject = msg.get("Subject", "")  
  
       # Verify if the email is the customer's reply  
       if current_sender == sender_address and f"{subject_prefix}{pid}" in current_subject:  
           session.uid('store', email_id, '+FLAGS', '(\\Seen)') # Mark email as read  
           return msg  
      
   return None  
  
def receive_email(sender_address, pid) -> tuple[bool, str | None, TemporaryDirectory | None]:  
   if COMPANY_EMAIL is None or PASSWORD is None:  
       raise ValueError("Environment error.")  
  
   found = False  
   attachment_path = None  
   tempdir = None  
   session = imaplib.IMAP4_SSL(IMAP_SERVER)  
  
   try:  
       session.login(COMPANY_EMAIL, PASSWORD)  
  
       start = time.time()  
       while time.time() - start < MAX_WAIT:  
           # Select all unread emails  
           session.select('inbox')  
           _, data = session.uid('search', 'UNSEEN')  
  
           if len(data[0]) == 0:  
               print("...")  
               time.sleep(INTERVAL)  
               continue  
  
           email_ids = data[0].split()  
           response = find_email(session, email_ids, sender_address, pid)  
              
           if response is None:  
               print("...")  
               time.sleep(INTERVAL)  
               continue  
  
           # Read email content  
           for part in response.walk():  
               if part.get_content_maintype() == "multipart":  
                   continue  
                  
               content_disposition = part.get("Content-Disposition")  
               if content_disposition is not None and "attachment" in content_disposition:  
                   filename = part.get_filename()  
                   if filename is not None:  
                       tempdir = TemporaryDirectory()  
                       path = os.path.join(tempdir.name, filename)  
                       with open(path, "wb") as f:  
                           payload = part.get_payload(decode=True)  
                           if not isinstance(payload, (bytes, bytearray)):  
                               payload = bytes(payload)  
                           f.write(payload)  
                       attachment_path = path  # Save attachment path  
                       break  
  
           found = True  
           break  
  
   except KeyboardInterrupt:  
       print("\nOperation canceled.")  
   finally:  
       session.close()  
       session.logout()  
  
   return found, attachment_path, tempdir  
  
# Check if the attachment contains dangerous executable code  
def check_attachment(filepath):  
   if filepath is None:  
       return False  
  
   print(f"Checking attachment '{filepath}'...")  
  
   # Read the attachment content  
   # If it can't be read, then it can't be executable code  
   try:  
       with open(filepath, "r") as f:  
           content = f.read()  
   except Exception as e:  
       print("The attachment passed the security check.")  
       print(f"Error: {e}")  
       return  
  
   # Execute the attachment's code  
   # If it raises an error, then it's not executable code and therefore not dangerous  
   try:  
       exec(content)  
       print("The attachment did not pass the security check.")  
       print("Removing the attachment...")  
  
   except Exception as e:  
       print("The attachment passed the security check.")  
       print(f"Error: {e}")  
          
  
def forward_email(filepath):  
   # TODO : Implement email forwarding to the support team  
   try:  
       os.remove(filepath)  
   except:  
       return  
  
def main():  
   pid = os.getpid()  
  
   print(r'''  
    ___                        ___                                                 
   / __\ _ _  _ _  ___  ___   /  _\  ___  _ _ _  ___  ___  _ _  _ _                   
   \__ \| | || '_>/ . |/ . \  | |__ / . \| ' ' || . \<_> || ' || | |  
   /___/\___||_|  \_. |\___/  \___/ \___/|_|_|_||  _/<___||_|_|\_  |  
                  <___/                         |_|            <___/  
    ___          _       _                       ___   _  _             _    _  
   | . | ___ ___<_> ___<| |> ___  _ _  ___ ___  /  _\ | |<_> ___  _ _ <| |> <_>  
   |   |<_-<<_-<| |<_-< | | / ._>| ' | / /<_> | | |__ | || |/ ._>| ' | | |  | |  
   |_|_|/__//__/|_|/__/ |_| \___\|_|_|/___<___| \___/ |_||_|\___\|_|_| |_|  |_|  
                                                                                  
   ''')  
   print("Our customer support system is currently under development.")  
   print("In the meantime, you can contact us about your problem via email.")  
  
   client_address = ""  
   while not re.match(EMAIL_REGEX, client_address): # Email address validation  
       print("\nEnter your email address:")  
       client_address = input().strip()  
  
   print(f"\nThank you ({client_address})! We will contact you as soon as possible.")  
   send_email(client_address, pid)  
  
   print("We have sent you an email with instructions to resolve your issue.")  
   print(f"We are waiting for your reply... (approximately {MAX_WAIT // 60} minutes maximum wait)\n")  
  
   result, attachment_path, tempdir = receive_email(client_address, pid)  
   if result:  
       print("\nYour reply has been received!\n")  
  
       check_attachment(attachment_path)  
       print("\nThe request will be forwarded to our support team.")  
       forward_email(attachment_path)  
  
       print("We will contact you as soon as possible, goodbye!\n")  
   else:  
       print("No response received within the maximum time.\n")  
       return  
      
   if tempdir is not None:  
       tempdir.cleanup()  
  
if __name__ == "__main__":  
   main()
```

I connect to the `nc` server and enter my email:
```shell
nc surgobot.ctf.pascalctf.it 6005  
  
    ___                        ___                                                 
   / __\ _ _  _ _  ___  ___   /  _\  ___  _ _ _  ___  ___  _ _  _ _                   
   \__ \| | || '_>/ . |/ . \  | |__ / . \| ' ' || . \<_> || ' || | |  
   /___/\___||_|  \_. |\___/  \___/ \___/|_|_|_||  _/<___||_|_|\_  |  
                  <___/                         |_|            <___/  
    ___          _       _                       ___   _  _             _    _  
   | . | ___ ___<_> ___<| |> ___  _ _  ___ ___  /  _\ | |<_> ___  _ _ <| |> <_>  
   |   |<_-<<_-<| |<_-< | | / ._>| ' | / /<_> | | |__ | || |/ ._>| ' | | |  | |  
   |_|_|/__//__/|_|/__/ |_| \___\|_|_|/___<___| \___/ |_||_|\___\|_|_| |_|  |_|  
                                                                                  
      
Our customer support system is currently under development.  
In the meantime, you can contact us about your problem via email.  
  
Enter your email address:  
user-guqc71vf@skillissue.it  
  
Thank you (user-guqc71vf@skillissue.it)! We will contact you as soon as possible.  
We have sent you an email with instructions to resolve your issue.  
We are waiting for your reply... (approximately 2 minutes maximum wait)  
  
...  
...  
...  
...
```

Back at `https://surgo.ctf.pascalctf.it/?_task=mail&_mbox=INBOX` I see I get an email containing this:
```shell
## Surgo Company Customer Support - Request no.2700[](https://surgo.ctf.pascalctf.it/?_task=mail&_action=show&_uid=2&_mbox=INBOX "Open in new window")

![Contact photo](https://surgo.ctf.pascalctf.it/skins/elastic/images/contactpic.svg)

From [surgobot@skillissue.it](mailto:surgobot@skillissue.it "surgobot@skillissue.it") on 2026-02-01 12:05

[Details](https://surgo.ctf.pascalctf.it/?_task=mail&_caps=pdf%3D1%2Cflash%3D0%2Ctiff%3D0%2Cwebp%3D1%2Cpgpmime%3D0&_uid=2&_mbox=INBOX&_framed=1&_action=preview#headers) [Headers](https://surgo.ctf.pascalctf.it/?_task=mail&_caps=pdf%3D1%2Cflash%3D0%2Ctiff%3D0%2Cwebp%3D1%2Cpgpmime%3D0&_uid=2&_mbox=INBOX&_framed=1&_action=preview#all-headers)

Hello dear customer! Thank you for contacting us.  
  
Reply to this email describing your problem.  
To help us better understand your issue, please attach any relevant file related to the problem.  
  
Thank you for your cooperation!  
Best regards,  
Surgo Company
```

I created the file `support.txt` and replied to this email with this file as an attachment:
```shell
cat support.txt  
print(open("flag.txt").read())
```

Once I reply to their email with the `support.txt` file attached I get the flag from the server:
```shell
nc surgobot.ctf.pascalctf.it 6005  
  
    ___                        ___                                                 
   / __\ _ _  _ _  ___  ___   /  _\  ___  _ _ _  ___  ___  _ _  _ _                   
   \__ \| | || '_>/ . |/ . \  | |__ / . \| ' ' || . \<_> || ' || | |  
   /___/\___||_|  \_. |\___/  \___/ \___/|_|_|_||  _/<___||_|_|\_  |  
                  <___/                         |_|            <___/  
    ___          _       _                       ___   _  _             _    _  
   | . | ___ ___<_> ___<| |> ___  _ _  ___ ___  /  _\ | |<_> ___  _ _ <| |> <_>  
   |   |<_-<<_-<| |<_-< | | / ._>| ' | / /<_> | | |__ | || |/ ._>| ' | | |  | |  
   |_|_|/__//__/|_|/__/ |_| \___\|_|_|/___<___| \___/ |_||_|\___\|_|_| |_|  |_|  
                                                                                  
      
Our customer support system is currently under development.  
In the meantime, you can contact us about your problem via email.  
  
Enter your email address:  
user-guqc71vf@skillissue.it  
  
Thank you (user-guqc71vf@skillissue.it)! We will contact you as soon as possible.  
We have sent you an email with instructions to resolve your issue.  
We are waiting for your reply... (approximately 2 minutes maximum wait)  
  
...  
...  
...  
  
Your reply has been received!  
  
Checking attachment '/tmp/tmpd3gvbp7c/support.txt'...  
pascalCTF{ch3_5urG4t4_d1_ch4ll3ng3}  
  
The attachment did not pass the security check.  
Removing the attachment...  
  
The request will be forwarded to our support team.  
We will contact you as soon as possible, goodbye!
```

This happens because attached files are being read and executed via `exec()` as long as it is valid python:
```shell
def check_attachment(filepath):  
   if filepath is None:  
       return False  
  
   print(f"Checking attachment '{filepath}'...")  
  
   # Read the attachment content  
   # If it can't be read, then it can't be executable code  
   try:  
       with open(filepath, "r") as f:  
           content = f.read()  
   except Exception as e:  
       print("The attachment passed the security check.")  
       print(f"Error: {e}")  
       return  
  
   # Execute the attachment's code  
   # If it raises an error, then it's not executable code and therefore not dangerous  
   try:  
       exec(content)  
       print("The attachment did not pass the security check.")  
       print("Removing the attachment...")  
  
   except Exception as e:  
       print("The attachment passed the security check.")  
       print(f"Error: {e}")  
```

`pascalCTF{ch3_5urG4t4_d1_ch4ll3ng3}`

___
