# Bad Compiler - Writeup by balpreet3761

## Task
Fix broken_compiler.exe so program.wut produces expected_output.txt

## Expected Output
"This is right! Congratulations!"

## WUT Language Analysis
After analyzing program.wut byte by byte:

- (N#%^ = add N to accumulator, print chr(acc)
- (N$%  = set accumulator to N directly  
- (N&@*% = multiply accumulator by N (acts as separator/reset)
- ~~ = print space
- @@ = print space
- !^ = print !
- @^ = print space
- ` = newline
- Base accumulator starts at 38

## How I Found It
1. Ran strings on program.wut - saw encoded tokens
2. Ran xxd to see raw bytes
3. Mapped token patterns to expected output characters
4. T=84, first token (46#, 38+46=84 = T - confirmed base=38
5. Rewrote compiler in Python as fixed_compiler.py

## My WUT Program
~~%(28#%^(13#%^(95#%^~~#%
Prints: "Ba " (start of Balpreet)

## Files
- fixed_compiler.py = rewritten compiler
- myprogram.wut = custom program
          
