# ELF-x86-CrackMe-Walkthrough-for-Absolute-Beginners
This guide will show you, even if you are totally new to malware analysis or reverse engineering, how to solve an ELF x86 CrackMe challenge using only free command-line tools. No prior reverse engineering experience needed! Every command is explained: what it does and why we use it.

---

## ❓ I'm a Complete Beginner. How Should I Approach Malware Analysis?

1. **Stay Calm!**  
   Everyone starts as a beginner. You don't need to know assembly or be a hacker to start.
2. **Start with the Basics:**  
   Learn what files you have and what they do, before trying to "break" anything.
3. **Break Down the Problem:**  
   Focus on one step at a time—identify, observe, analyze, and test.
4. **Use the Right Tools:**  
   Each tool gives you a different "view" into the program. We'll explain why and how to use each.

---

## 📝 What is a CrackMe?

A CrackMe is a small program designed for learning reverse engineering. Your goal: **Figure out the password or key that solves the challenge.**

---

## 🛠️ Tools You'll Need (and Why)

- **A Linux system** (you used Parrot OS — perfect!)
- **Command-line tools:**
  - `file` — *Identify file type*  
    > Why: Tells you what kind of program you're dealing with (Linux, Windows, etc).
  - `strings` — *Find readable text in binaries*  
    > Why: Shows hidden clues, messages, or even passwords inside the binary.
  - `objdump` — *Disassemble code and examine data*  
    > Why: Lets you peek inside the program's logic and see how it works.
  - `nm` — *List symbols (functions, variables)*  
    > Why: Reveals names of functions, like "main", so you can focus analysis.
  - `xxd` — *Hex dump utility*  
    > Why: Lets you see the raw bytes in the binary—sometimes clues are only visible in hex.
  - (`gdb` and other debuggers are handy but not strictly needed for this challenge.)
    > Why: Used for dynamic analysis—step through the program as it runs. Not needed for this basic walkthrough.

---

## 1. **Identify the File**

```sh
file ch2.bin
```
> **Why?**  
This command tells you what kind of file you're working with. It confirms if it's an ELF executable (Linux binary), 32-bit or 64-bit, etc.

**Expected Output:**
```
ch2.bin: ELF 32-bit LSB executable, Intel 80386, ... statically linked ...
```
> This confirms you're dealing with a 32-bit Linux executable.

---

## 2. **Run the Program**

```sh
./ch2.bin
```
> **Why?**  
See what the program does! This is always your first move—observe its normal behavior.

You'll see something like:
```
############################################################
##        Bienvennue dans ce challenge de cracking        ##
############################################################

username:
```
- Enter any username, e.g. `test`
- It will respond:  
  `Bad username`

> **What did you learn?**  
The program asks for a username and doesn't accept random input. There's a correct username (and maybe a password) to find.

---

## 3. **Look for Clues with `strings`**

```sh
strings ch2.bin | less
```
> **Why?**  
Lists all readable text in the binary. Many beginners' CrackMes hide clues, hardcoded usernames, passwords, or hints here.

**Look for:**
- Prompts: `username:`, `password:`
- Messages: `Bad username`, `Bien joue, vous pouvez valider l'epreuve avec le mot de passe : %s !`

> **What did you learn?**  
There is both a username and a password check, and a success message (in French).

---

## 4. **Find How Username is Checked**

**First, find where the "main" function is:**
```sh
nm ch2.bin | grep main
```
> **Why?**  
Lists all function names and addresses. We want to find "main" because that's where the program starts.

Sample output:
```
08048309 T main
```

**Now disassemble the main function:**
```sh
objdump -d --start-address=0x08048309 --stop-address=0x08048400 ch2.bin
```
> **Why?**  
Shows the assembly code for main—so you can see the logic, like where it checks your input.

**You're looking for:**
- Calls to `strcmp` (string compare)
- Addresses like `0x80a6b19` or `0x80a6b1e`
- These addresses store the correct username and password

---

## 5. **Extract Username and Password from Binary**

**Check the read-only data section for these addresses:**
```sh
objdump -s -j .rodata ch2.bin | grep -A10 0a6b10
```
> **Why?**  
Looks directly at the section of the binary where constant strings (like usernames and passwords) are stored.

**You might see:**
```
80a6b10 67206d65 6d6f7279 006a6f68 6e007468  g memory.john.th
80a6b20 65207269 70706572 00000000 ...       e ripper....
```
This means:
- At `0x80a6b19`: `john` (username)
- At `0x80a6b1e`: `the ripper` (password)

**Or run:**
```sh
strings -td ch2.bin | grep 6b1
```
> **Why?**  
This shows strings with their memory address/offset, making it easier to match to what you saw in the code.

---

## 6. **Try the Credentials**

Run:
```sh
./ch2.bin
```
- When prompted for username, enter: `john`
- When prompted for password, enter: `the ripper`

**Expected Output:**
```
Bien joue, vous pouvez valider l'epreuve avec le mot de passe : [hidden password] !
```
*(The actual password/validation number is intentionally hidden for privacy or challenge integrity.)*

> **Why?**  
You are testing your findings! If you get the success message, you did it right.

---

## 7. **Translate the Output**

French:  
> Bien joue, vous pouvez valider l'epreuve avec le mot de passe : [hidden password] !

English:  
> Well done, you can validate the challenge with the password: [hidden password] !

---

## 8. **Submit the Solution**

**The password to submit for validation is:**  
```
[hidden password]
```

---

## 🎉 **Summary Table**

| Step                           | Command Example                                               | Why Do This?                      |
|---------------------------------|--------------------------------------------------------------|-----------------------------------|
| Identify file                   | `file ch2.bin`                                               | Confirm it's an ELF binary        |
| Run program                     | `./ch2.bin`                                                  | Observe its behavior              |
| List strings                    | `strings ch2.bin | less`                                     | Find readable clues               |
| Find main function              | `nm ch2.bin | grep main`                                     | Get main() function address       |
| Disassemble main function       | `objdump -d --start-address=0x08048309 --stop-address=... ch2.bin` | See logic in code         |
| Find username/password in data  | `objdump -s -j .rodata ch2.bin | grep -A10 0a6b10`         | Reveal correct credentials        |
| Try credentials                 | `./ch2.bin`                                                  | Validate solution                 |

---

## 📚 **Tips for Absolute Beginners**

- **Always start by running the program!**  
  See what it wants before you start poking around.
- **Use `strings` for easy wins.**  
  Many simple CrackMes hide the answer in plain sight.
- **Don’t be afraid of disassembly.**  
  You don’t need to understand every instruction—just look for patterns, like function calls and string addresses.
- **Practice makes perfect!**  
  Each challenge you solve teaches you another trick or pattern to look for.

---

## 🚀 **Next Steps**

- Try harder CrackMes!
- Learn to use `gdb` for step-by-step debugging.
- Check out GUI tools like Ghidra for a friendlier interface.

---

**You cracked your first ELF x86 challenge—congratulations!**
