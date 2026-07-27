2. What does the -l flag to ls do? 
The -l flag list files in the long format.
The Long Format is the following information is displayed for each file: file mode, number of links, owner name, group name, number of bytes in the file, abbreviated month, day-of-month file was lat modified, hour file last modified, minute file last modified, and the pathname....

3. What is a glob?
"Glob" is the pattern for the command to match when finding base on name of the object (file/directory).

Pattern Matching in the Bash manual https://www.gnu.org/software/bash/manual/html_node/Pattern-Matching.html
* matches any string, including the null string.
? matches any single character.
[...] matches any one of the characters enclosed between the brackets.

4. What’s the difference between 'single quotes', "double quotes", and $'ANSI quotes'? 
Single quotes preserves the literal value of each character within the quotes. A single quote may not occur between single quotes, even when preceded by a backslash.

Double quotes preserves the literal value of all characters within the quotes, with the exceptions of '$', '`', '\', and, when history expansion is enable, '!'.
The characters '$' and '`' retain their special meaning within double quotes. The backslash retains its special meaning only when followed by one of the following characters: '$', '`', '"', '\' or newline.

$'ANSI quotes' are treated as a special kind of single quotes.
Example
```
echo $'aa\bb\Exx\tcc\vgg'
<<< OUTPUT
abx	cc
          gg
```

5. The shell has three standard streams: stdin (0), stdout (1), and stderr (2). Run ls /nonexistent /tmp and redirect stdout to one file and stderr to another. How would you redirect both to the same file?
```
ls /nonexistent /tmp > stdout 2> stderr
ls /nonexistent /tmp > output 2>&1 # redirect the stderr into stdout's target
ls /nonexistent /tmp &> output
```

6. Write one-liner that creates /tmp/mydir only if it doesn't already exist.
```
test -d /tmp/mydir || mkdir /tmp/mydir
```

7. Why does cd have to be built into the shell itself rather than a standalone program? (Hint: think about what a child process can and cannot affect in its parent.)
I think it is because the child process cannot affect pwd of its parent so the cd must be built into the shell itself.

8. Write a script that takes a filename as an argument ($1) and checks whether the file exists using test -f or [ -f ... ]. It should print different messages depending on whether the file exists.
```
#!/bin/bash
if [ -f "$1" ]; then
echo "$1 exists";
else
echo "$1 doesn't exist";
fi
```

9. Save the script from the previous exercise to a file (e.g., check.sh). Try running it with ./check.sh somefile. What happens? Now run chmod +x check.sh and try again. Why is this step necessary?

Because before chmod +x check.sh, the user don't have permission to execute the file.

10. What happens if you add -x to the set flags in a script?
It print a trace of simple commands

11. Write a command that copies a file to a backup with today’s date in the filename.
```
cp notes.txt notes_$(date +%Y-%m-%d).txt
```

12. Modify the flaky test script from the lecture to accept the test command as an argument instead of hardcoding cargo test my_test.
```test.sh
#!/bin/bash
set -euo pipefail

# Start CPU stress in background
stress --cpu 8 &
STRESS_PID=$!

# Setup log file
LOGFILE="test_runs_$(date +%s).log"
echo "Logging to $LOGFILE"

# Run tests until one fails
RUN=1
while "$@" > "$LOGFILE" 2>&1; do
    echo "Run $RUN passed"
    ((RUN++))
done

# Cleanup and report
kill $STRESS_PID
echo "Test failed on run $RUN"
echo "Last 20 lines of output:"
tail -n 20 "$LOGFILE"
echo "Full log: $LOGFILE"
```
execute: ./test.sh cargo

13. Use pipes to find the 5 most common file extensions in your home directory. (Hint: combine find, grep or sed or awk, sort, uniq -c, and head.)
`find . -name "test*" | awk -F '.' '{print $NF}' | sort | uniq -c | sort --reverse | head -n 5`

14. xargs converts lines from stdin into command arguments. Use find and xargs together (not find -exec) to find all .sh files in a directory and count the lines in each with wc -l.
```
find . -name "*.sh" -print0 | xargs -0 wc -l
```
-print0 to output each pathname followed by a NUL (\0) character instead of a new line.
xargs -0 to separate input items by the NUL character (\0), not by whitespace or newlines.

15. Use curl to fetch the HTML of the course website (https://missing.csail.mit.edu/) and pipe it to grep to count how many lectures are listed.
```
curl -s https://missing.csail.mit.edu/ | grep '<strong>' -A 2 | grep '<a href="/20' | wc -l
<<<OUTPUT
19
```

16. Fetch the sample data at https://microsoftedge.github.io/Demos/json-dummy-data/64KB.json with curl and use jq to extract just the names of people whose version is greater than 6.
`curl https://microsoftedge.github.io/Demos/json-dummy-data/64KB.json | jq '.[] | select(.version > 6) | .name'`

17. awk can filter lines based on column values and manipulate output. For example, awk '$3 ~ /pattern/ {$4=""; print}' prints only lines where the third column matches pattern, while omitting the fourth column. Write an awk command that prints only lines where the second column is greater than 100, and swaps the first and third columns. Test with: printf 'a 50 x\nb 150 y\nc 200 z\n'
```
printf 'a 50 x\nb 150 y\nc 200 z\n' | awk '$2 > 100 {s=$1;$1=$3; $3=s; print}' 
<<<OUTPUT
y 150 b
z 200 c
```

18. Dissect the SSH log pipeline from the lecture: what does each step do? Then build something similar to find your most-used shell commands from ~/.bash_history (or ~/.zsh_history).
```
missing:~$ ssh myserver 'journalctl -u sshd -b-1 | grep "Disconnected from"' \
  | sed -E 's/.*Disconnected from .* user (.*) [^ ]+ port.*/\1/' \
  | sort | uniq -c \
  | sort -nk1,1 | tail -n10 \
  | awk '{print $2}' | paste -sd,
postgres,mysql,oracle,dell,ubuntu,inspur,test,admin,user,root
```
journalctl = query the systemd journal then pass it to the grep to grep all "Disconnected from"
pass it to the sed, sed match "Disconnected from" plus anything before it
then match "user" plus anything before it
the (.*) capture the user
[^ ]+ mean 1 ore more characters that are NOT spaces
\1 mean replace entire line with capture group 1
sort -nk1,1 mean sort numerically (-n) by the first field(-k1,1)

My answer:
```
export LC_ALL=C
sed -E 's/^[^;]*;([^ ]+).*/\1/' ~/.zsh_history | sort | uniq -c | sort -nk1,1 | tail -n10
<<<OUTPUT
  77 vim
  81 ssh
  86 man
  87 echo
 104 
 179 cat
 245 uv
 283 ls
 400 cd
 501 git
```
LC_ALL tells programs to treat the input as raw bytes instead of UTF-8 caharacters
