# Etdev's Useful Command Cheatsheet

### Contents
* [Databases](#databases)
  * [MySQL](#mysql)
  * [Postgresql](#postgres)
* [Command line](#command-line)
  * [ag](#ag)
  * [awk](#awk)
  * [curl](#curl)
  * [sed](#sed)
  * [shell](#shell) (bash/zsh scripting)
  * [sort](#sort)
  * [uniq](#uniq)
  * [wc](#wc)
  * [xargs](#xargs)
  * [paste](#paste)
* [Git](#git)
  * [Gitignore patterns](#gitignore-patterns)
* [Javascript](#javascript)
* [One-liners and Misc.](#one-liners)

### <a name="databases">Databases</a>

#### <a name="mysql">MySQL</a>

Create a new user
```sql
CREATE USER 'newuser'@'localhost' IDENTIFIED BY 'password';
```

Grant permissions to user
```sql
GRANT ALL PRIVILEGES ON *.* TO 'newuser'@'localhost';
```

Reload privileges:
```sql
FLUSH PRIVILEGES;
```

Update record(s):
```sql
UPDATE `potluck`
SET
`confirmed` = 'Y'
WHERE `potluck`.`name` = 'Sandy';
```

Delete record(s):
```sql
DELETE FROM `potluck` WHERE `potluck`.`name` = 'John';
```

Add column to table:
```sql
ALTER TABLE potluck ADD email VARCHAR(40) AFTER name;
```

Remove column from table:
```sql
ALTER TABLE potluck DROP email;
```

Analyze (explain) query:
```sql
EXPLAIN SELECT COUNT(*) FROM salaries WHERE salary BETWEEN 60000 AND 70000;
```

Change a column's data type,options:
```sql
ALTER TABLE t1 MODIFY a TINYINT NOT NULL;
```

Add an index to existing column:
```sql
ALTER TABLE t1 ADD index (d);
```

View the indexes on a table:
```sql
SHOW INDEXES FROM table_name;
```

Connect to database from the command line:
```bash
mysql -u USERNAME -pPASSWORD -h HOSTNAMEORIP DATABASENAME
```

Check currently-connected database:
```bash
SELECT DATABASE();
```

#### <a name="postgres">Postgresql</a>

Add column to table
```sql
ALTER TABLE distributors ADD COLUMN address varchar(30);
```

Drop a column from table:
```sql
ALTER TABLE distributors DROP COLUMN address RESTRICT;
```

Add a not-null constraint to a column:
```sql
ALTER TABLE distributors ALTER COLUMN street SET NOT NULL;
```

Add a foreign key constraint:
```sql
ALTER TABLE distributors ADD CONSTRAINT distfk FOREIGN KEY (address) REFERENCES addresses (address) MATCH FULL;
```

Add a (multi-column) unique constraint:
```sql
ALTER TABLE distributors ADD CONSTRAINT dist_id_zipcode_key UNIQUE (dist_id, zipcode);
```

### <a name="command-line">Command line</a>

#### <a name="ag">ag (Silver Searcher)</a>

Agignore patterns

* Basically the same as `gitignore` patterns, but has some issues
* Use the latest version if possible

E.g. Minified css or js files anywhere
```bash
# .agignore
**.min.*
# matches script.min.js, styles.min.css etc.
```

More info: https://github.com/ggreer/the_silver_searcher/issues/385

#### awk
Basic syntax
```bash
awk '/search_pattern/ { action_to_take_on_matches; another_action; }' file_to_parse
```

E.g. print each line in `etc/fstab` beginning with `UUID`
```bash
awk '/^UUID/' /etc/fstab
```

E.g. print the 6th column of netstat output:
```
sudo netstat -pant | awk '{print $6}'
```

Full syntax:
```bash
awk 'BEGIN { action; }
/search/ { action; }
END { action; }' input_file
```

E.g. set FieldSeparator to `:` and then print first column of `/etc/passwd`
```bash
awk 'BEGIN { FS=":"; }
{ print $1; }' /etc/passwd
```

Given `favorite_foods.txt`:
```bash
1 carrot sandy
2 wasabi luke
3 sandwich brian
4 salad ryan
5 spaghetti jessica
```

E.g. Match based on regex per-field
```bash
awk '/$2 ~ ^sa/' favorite_food.txt
# => 3 sandwich brian
# => 4 salad ryan
```

E.g. combined matching, including `!~` to NOT match a regex per-field, and `&&` to combine conditions
```bash
awk 'BEGIN { print "NUM\tFOOD"; } $2 ~ /^(c|w)/ && $1 !~ /1/ { print $1,"\t",$2; }' favorite_foods.txt
# => NUM   FOOD
# => 2     wasabi
```

E.g. Set ORS (output record separator) var to use e.g. commas instead of newlines
```bash
awk 'ORS=", " { print $1; }' favorite_foods.txt
# => 1, 2, 3, 4, 5, 
```

Built-in variables:
```bash
ARGC = input argument count (cols)
ARGV = array storing the command-line args
FILENAME = current filename
FS = input field separator
NF = number of fields in current record{
NR = number of current record
OFS = output field separator (def. space)
RS = input record separator (def. newline)
```

More info: https://www.digitalocean.com/community/tutorials/how-to-use-the-awk-language-to-manipulate-text-in-linux

#### <a name="curl">curl</a>
E.g. Get your current IP
```bash
curl icanhazip.com
```

E.g. Get http headers for URL
```bash
curl -I example.com
```

E.g. Follow redirects
```bash
curl -L google.com
```

E.g. Continue downloading partially-downloaded file
```bash
curl -C - example/com/file.zip
```

E.g. Pass HTTP Basic Auth credentials
```bash
curl -u etdev:pass example.com/members
```

E.g. Download files in range
```bash
curl ftp://ftp.uk.debian.org/debian/pool/main/[a-z]/
```

E.g. POST to a page
```bash
curl -d "name=etdev&password=hunter2" www.example.com/process.php
```

E.g. Set referer
```bash
curl -e "http://someReferringSite.com" http://www.example.com
```

E.g. Set request method
```bash
curl -X POST quiet-waters-1228.herokuapp.com/echo
```

E.g. Add parameters to body
```bash
curl -X POST -d "fname=Mark&lname=Bates" quiet-waters-1228.herokuapp.com/echo
```

E.g. Add parameters to body by specifying a file
```bash
curl -X POST -d "fname=Mark&lname=Bates" quiet-waters-1228.herokuapp.com/echo
```

E.g. POST as a form
```bash
curl -X POST -F user[fname]=Mark -F user[lname]=Bates -F foo=bar \
  quiet-waters-1228.herokuapp.com/echo -H "Accept: application/json"
```

E.g. Set headers
```bash
curl -X POST -d @form_data.json quiet-waters-1228.herokuapp.com/echo \
  -H "Accept: application/json"
```

Adding the `Accept: application/json` header will return JSON instead of HTML.


#### <a name="sed">sed</a>

Sed is a **s**tream **ed**itor.

Basic syntax:
```bash
sed [options] commands [file-to-edit]
```

E.g. print first line of test.txt
```bash
sed -n '1p' test.txt
```
* The `-n` suppresses the default print each line behavior
* The `1` tells sed what line(s) to operate on -- line 1
* The `p` means the `print` command; so `print line 1, suppressing default print`

E.g. print lines 1-5 of test.txt
```bash
sed -n '1,5p' test.txt
```
```bash
sed -n '1,+4p' test.txt
```

E.g. print every other line of test.txt
```bash
sed '1~2p' test.txt
```

E.g. **d**elete every other line of test.txt
```bash
sed '1~2d' test.txt
```

* Doesn't affect the actual file
* You can make it change the source file with the `-i` option though.

E.g. delete every other line of test.txt, actually changing original file
```bash
sed -i.bak '1~2d' file.txt
```
* The `-i.bak` means it will create a backup file with the extension `.bak`

Given `annoying.txt`:
```bash
this is the song that never ends
yes, it goes on and on, my friend
some people started singing it
not knowing what it was
and they'll continue singing it forever
just because...
```

E.g. Substitute `on` with `forward`
```bash
sed 's/on/forward' annoying.txt
```

* Only replaces first match in each line
* Replaces any instance of `on`, even in the middle of other words

E.g. Substitute all instances of `on` in first 3 lines with `forward`
```bash
sed '1,3s/on/forward/g' annoying.txt
```

E.g. Substitute case-insensitively all instances of `SINGING` with `saying`, and only print changed lines
```bash
sed -n 's/SINGING/saying/gip' annoying.txt
```

E.g. Substitute from beginning of line to last instance of `at` with parenthesized version
```bash
sed 's/^.*at/(&)/' annoying.txt
```
* The `&` represents the captured text (which is captured despite lack of parentheses in the regex)

E.g. Switch the first two words in each line
```bash
sed 's/\([^ ][^ ]*\) \([^ ][^ ]*\)/\2 \1/' annoying.txt
```

* You can use escaped parentheses for capture groups, which you can then refer to with `\1`, `\2`, ...

E.g. Substitute `it` with `it loudly` but only on lines that match `/singing/`
```bash
sed '/singing/s/it/& loudly/' annoying.txt
```

E.g. Delete all empty lines
```bash
sed '/^$/d' annoying.txt
```

E.g. Delete all lines that AREN'T empty
```bash
sed '/^$/!d' annoying.txt
```

E.g. Delete all lines between lines starting with `START` and `END` (including the start, end lines)
```bash
sed '/^START/,/^END/d' test.txt
```

E.g. Replace all instances of `routeParams` with `stateParams` (with ag, xargs)
```bash
ag 'routeParams' -l | xargs sed -i 's/routeParams/stateParams/' {}
```

#### <a name="shell">shell</a>

Simple for loop

```bash
for i in 1 2 3 .. N
do
<commands>
done
```

E.g. create files `test1.txt` through `test5.txt`
```bash
for i in 1 2 3 4 5
do
touch test$i.txt
done
```

E.g. print files in directory, but preceeded by some message
```bash
for i in $(ls); do
echo item: $i;
done
```

E.g. rename files in current dir with a `2` in their name, to `item_<old_name>`
```bash
for i in $(ls | grep 2); do
mv $1 item_$1;
done
```

More info: http://www.tecmint.com/wc-command-examples/

#### <a name="sort">sort</a>

Sorts text by line
```bash
echo 'c\nb\na' | sort
# => a
# => b
# => c
```

More info: http://www.computerhope.com/unix/usort.htm

#### <a name="uniq">uniq</a>

Get the unique values from some text stream/file

E.g. simply remove duplicates
```bash
echo 'a\na\nb' | uniq
# => a
# => b
```

E.g. show counts

```bash
echo 'a\na\nb' | uniq -c
# => 2 a
# => 1 b
```

#### <a name="wc">wc</a>

Count **l**ines / **w**ords / **c**haracters in a file or stream

E.g. Show line count, word count, and char count for test.txt
```bash
wc test.txt
# => 15  229 1529 test.txt
```

E.g. Show line count for test.txt
```bash
wc -l test.txt
# => 15 test.txt
```

E.g. Show word count for test.txt
```bash
wc -w test.txt
# => 229 test.txt
```

E.g. Show char count for test.txt
```bash
wc -c test.txt
# => 1529 test.txt


```

#### <a name="xargs">xargs</a>

Get args from standard input and do things with them

E.g. remove files matching `test3`
```bash
find . -name "*test3*" | xargs rm
```

E.g. remove files matching `test3` #2
```bash
find . -name "*test3*" | xargs -i rm {}
```

The `-i` basically replaces `{}` with the current parameter

E.g. Print filenames + 'is a file!' for files matching `/test[1,2]{2}$/`
```bash
ag -g 'test[1,2]{2}\.txt$' | xargs -Iresult echo result is a file!
```

The `-I` lets you refer to the current matched param in the command.

#### <a name="paste">MySQL</a>

E.g. List the files in current directory in three columns
```bash
ls | paste - - -
```

E.g. Combine pairs of lines from a file into single lines
```bash
paste -s -d '\t\n' - - myfile
```

### <a name="git">Git</a>


#### <a name="gitignore-patterns">gitignore patterns</a>

E.g. Any file or directory called `doc`, recursive
```bash
# .gitignore
doc
```

E.g. Any directory called `doc`, recursive
```bash
# .gitignore
doc/
```

E.g. Any file or directory called `doc`, root dir only
```bash
# .gitignore
/doc
```

E.g. Directory called `doc`, root dir only
```bash
# .gitignore
/doc/
```

E.g. Files anywhere ending with `.tmp`
```bash
# .gitignore
*.tmp
```

E.g. Files under the /doc directory ending with `.tmp`
```bash
# .gitignore
/doc/*.tmp
# matches /doc/test.tmp, NOT /doc/stuff/test.tmp
```

E.g. File or directory `bar` whenever its parent directory is `foo`
```bash
# .gitignore
**/foo/bar
```

E.g. All files and directories inside `foo` directory, anywhere
```bash
# .gitignore
foo/**
```

E.g. `baz` directory with ancestor directory `foo`
```bash
# .gitignore
foo/**/baz/
```

E.g. All `.html` files except `index.html`
```bash
# .gitignore
*.html
!index.html
```

More info: https://git-scm.com/docs/gitignore

### <a name="one-liners">One-liners and misc.</a>

Print the last 100 `id`s of a resource (e.g. posts) from a JSON API with metadata, comma-separated
```bash
=
```

Smartresize function with imagemagick
```bash
smartresize() {
   mogrify -path $3 -filter Triangle -define filter:support=2 -thumbnail $2 -unsharp 0.25x0.08+8.3+0.045 -dither None -posterize 136 -quality 82 -define jpeg:fancy-upsampling=off -define png:compression-filter=5 -define png:compression-level=9 -define png:compression-strategy=1 -define png:exclude-chunk=all -interlace none -colorspace sRGB $1
}
=

Usage example:
```bash
smartresize inputfile.png 300 outputdir/
```

Pyenv/Virtualenv - create
```bash
pyenv virtualenv <python_version> <virtualenv_name>
```

E.g.
```bash
pyenv virtualenv 3.5.1 my_virtualenv
```

Pyenv/Virtualenv - activate
```bash
pyenv activate <virtualenv_name>
```

### <a name="javascript">Javascript</a>

#### Iteration

**Array** - Select elements from array where condition is true

```js
arr.filter(element => element.enabled);
```

**Array** - Get array with 5 added to all elements (non-destructive)
```js
arr.map(element => element + 5);
```

**Object** - Iterate through an Object
```js
for (var key in p) {
  if (p.hasOwnProperty(key)) {
    console.log(key + " -> " + p[key]);
  }
}
```

**String** - Iterate through string
```js
const str = 'myString';
for ( const chr of str ){
  do_something(chr);
}
```

You can also use the for-of loop for Maps, Sets, and most other iterable objects.
