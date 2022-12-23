# PicoCTF-Transformation
Write-up for a PicoCTF challenge

Location: https://play.picoctf.org/practice/challenge/104

Category: Reverse Engineering

Description: I wonder what this really is... [enc] ''.join([chr((ord(flag[i]) << 8) + ord(flag[i + 1])) for i in range(0, len(flag), 2)])

My solving approach:
  1. The first thing I did was download the [enc] file and open it in notepad, revealing "灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸彤㔲挶戹㍽"
  2. Second, I looked into the code given in the description. I recognized this as Python, but had to look into what some of the function were. After learning that ord() is used to convert a Unicode character into its Unicode code number, and chr() does the opposite (coverting a code into a character), I realized we were taking the unicode, chinese-like unicode characters, converting them to a numeric code, manipulating these code numbers, then converting these new codes back to Unicode characters to get the flag.
  3. My first attempt was to open up VSCode, set a variable named "flag" = "灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸彤㔲挶戹㍽", then a variable "answer" = "". then run answer.join([chr((ord(flag[i]) << 8) + ord(flag[i + 1])) for i in range(0, len(flag), 2)]) and print out "answer." This gave an error: ValueError: chr() arg not in range(0x110000)
  4. I then tried to convert the flag to bytes with #flag = bytes(灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸彤㔲挶戹㍽), this gave the error "SyntaxError: invalid character '㌴' (U+3334)"
  5. Next I converted the chinese characters to binary online and tried setting flag equal to that like so: "flag = bin(0o01101..." but this gave the error "IndexError: string index out of range"
  6. Finally, I tried putting the pull binary string into my search engine, i got "i o T { 6 b t _ n t 4 _ f 8 d 2 6 9 }" which finally started looking promising. I put this into CyberChef and got a bunch of options, but none of them were correct (I know the flag format, and that it starts with "picoCTF{"). At this point, I recognized that I was flailing and decided to just search the solution online.
  7. Interestingly enough, if I had just put the chinese characters into CyberChef and used intensive mode, I would have found that the algorithm given in python is the same as what's known as UTF-16BE, and been able to get the flag this way.

How to solve via reverse engineering:
  1. First, throw the hint ("You may find some decoders online") out the window, it's completely useless.
  2. Second, recognize and understand the python line as python, and learn what the "join", "ord" and "chr" functions do, as well as how "for loops" work and what the << operator does. 
  3. You'll also need to understand that the python line given ENCODES the texts into the funky chinese characters. This is where the reverse engineering comes into play. You'll need to understand how the code works so that you can write a script that reverses what it does (IOW, decodes what the given python encodes). Here's how to get there, looking at the python code:
 
    a. Assuming "flag" in the code is "picoCTF{...}", we can start pluggin in those letters into the python line 
    
    b. If we are following convention, flag[0] should be 'p' and flag[1] should be 'i'. Therefore, the following code should give us the first line of our "mandarin" (灩): chr((ord('p') << 8) + ord('i')). Oh good, it does. We can continue on trying to reverse the given python algo.
    
    c. It is important to understand what's happening in terms of binary before we can reverse engineer this with confidence. let's look at it again with ord('p') and ord('i'). if you type print(ord('p')) in python you'll get 112. 112 base10 is 01110000 in binary. Shifting this left by 8 bits (<<8 in python) gives us 01110000 00000000. now let's add ord('i'), which is 01101001, to get 01110000 01101001. if we were to chr() that, we'd get our first mandarin character. In essence, eflag[0] is a 16-bit amalgum of the 8-bit flag[0] for bits 16 thru 9 and 8-bit flag[1] for bits 8 thru 1.
    
    d. The important thing needed to reverse engineer this, is noticing that the information for flag[0] is still available to us even after adding flag[1], so long as we stay in binary and reverse that bit-shifting we did. For example, (01110000 01101001 >> 8) = (01110000 11111111 >> 8) = (01110000 00000000 >> 8) = 01110000 = 112 base10 = 'p' after you do chr(112). Therefore, to get flag[0], all we have to do is bit-shift the first mandarin character right by 8 like this: chr(ord(eflag[0]) >> 8). this gives us "p".
  4. Again, our encoding algorithm is 灩 = (p << 8) + i, and since we now have p, we can go backwards to get i by rearranging that equation thusly: i = 灩 - (p << 8). 灩 is chr(ord(eflag[0])). p is chr(ord(eflag[0]) >> 8). Therefore i = chr(ord(eflag[0]) - ((ord(eflag[0]) >> 8) << 8)).
  5. Now we can write a basic script in python to loop this over the entire string and print it out:
    	eflag = '灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸弲㘶㠴挲ぽ'
      flag = ''

      for i in range(len(enc)):
        flag += chr(ord(eflag[i]) >> 8)
        flag += chr(ord(eflag[i]) - ((ord(eflag[i]) >> 8) << 8)) #this can be simplified to i = chr(ord(eflag[0]) - (ord(flag[-1]) << 8)), by using python's negative indexing.
      print(flag)

Flag: picoCTF{16_bits_inst34d_of_8_e141a0f7}
