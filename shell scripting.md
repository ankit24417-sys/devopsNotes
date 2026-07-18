# 📝 Text Editors

### 1) nano
First text Editor is nano => it is simple to modify any file use the command => `nano <file name>`. It will open the file, after some changes press:
- `ctrl+s` (to save)
- `ctrl+x` (to exit the nano)

---

### 2) gedit
Second is text Editor is gedit => it's full form is graphical edit . it opens the file is graphical interface and you can edit file very easily .

---

### 3) vim
Third is text Editor is vim => if you have a file use command `vim <filename>` and modify or if your file doesn't exist use same command => `vim <filename>` it will create a one and open it for the modifications.
- Pressing the `i` on keyboard, vim goes in the inserting mode means we can write something on it.
- Pressing the `esc` i.e. escape button, vim goes to the command mode where you can give command like:
  - `:wq` => means save the written work and quit
  - `:qa` => quit without saving the work

> [!TIP]
> **Vim Pro Tip:**
> - Press `u` in command mode to **undo** your last change.
> - Press `dd` in command mode to **delete (cut) a line**.
> - Press `:q!` to force quit without saving if `:qa` doesn't let you exit.

---
---

# 🐚 Shell Scripting Start

### 1) File Extension
All shell Scripting files should be ends with `.sh` extension (but it is optional).

### 2) The Shebang Header
All the shell Scripting files contains the header which tell the kernel , how should these files be executed which is:
```bash
#!/bin/bash
```

> [!NOTE]
> This special line is called a **Shebang**. It tells your operating system to use the Bash interpreter located at `/bin/bash` to run the file.

### 3) Comments
To make comments on the file we use the `#` symbol.
```bash
# This is a comment and will be ignored by the terminal
```

### 4) Executable Permissions
After writing the script, it will not be executable. To give it that permission, we simply use the command =>
```bash
chmod +x <filename>
```

### 5) Run/Execute
To execute the file, we write =>
```bash
./filename
```

---

### 6) Variables
- **i)** We can assign value to a Variables using equal to operator (`=`).
- **ii)** To print the Variables (for which we use `echo` command), we use the dollar sign (`$`) before the Variables.
- **iii)** Example:
  ```bash
  name="ankit"
  echo "$name is genius" # or: echo $name
  ```
- **iv)** If you want to execute any command in the script file, you have to write it in the following format:
  ```bash
  $(name of command as usual);
  # Example:
  echo $(whoami)
  ```

> [!WARNING]
> **No Spaces Allowed:** In Bash, you **must not** put spaces around the `=` sign when declaring variables. 
> - Correct: `name="ankit"`
> - Incorrect: `name = "ankit"` (this will cause a command not found error!)

---

### 7) Taking Input
- **i)** To take input in shell Scripting we use the `read` command or to give message in the read using option => `-p`.
- **ii)** `read <VariablesName>`
- **iii)** `read -p <message > <VariablesName>`
- **iv)** Example:
  ```bash
  echo "enter your name"
  read username # or read -p "name: " username
  echo "you entered $username"
  ```

> [!TIP]
> To hide input (e.g. when typing passwords), you can use the silent option `-s`:
> `read -sp "Enter Password: " password`

---

### 8) Conditionals

#### i) For Numbers:
We use the syntax written below:
```bash
if ((condition))    // >, <, ==, !=, >=, <=
then 
   #write your output
elif ((condition))
then 
   #write your output
else 
then 
   #write your output
fi    // this is for exit the if else condition
```

#### ii) For Strings:
We use the syntax written below:
```bash
if [[condition]]    // ==, !=
then 
   #write your output
elif [[condition]]
then 
   #write your output
else 
then 
   #write your output
fi    // this is for exit the if else condition
```

> [!IMPORTANT]
> **Bash Syntax Note:** In standard Bash syntax, the `else` block does not require a `then` keyword. It is typically written as:
> ```bash
> else
>    # write your output
> ```

---

### 9) Loops
It contains two type of loops:
- **i)** for loop
- **ii)** while loop

#### i) For Loop:
```bash
for (( initialisation; condition; updation ))
do
   #output code 
done 
```

> [!TIP]
> **Alternative List-Based For Loop:**
> Often in scripts, you loop through a list of items:
> ```bash
> for item in apple banana cherry
> do
>    echo "Fruit: $item"
> done
> ```

#### ii) While Loop:
```bash
initialisation
while (( condition ))
do
   #output
   (( updatation ))
done 
```

*///// always use vim for shell Scripting*

---

### 10) Functions
We define the function in shell script in a normal way.

- **i) Example:**
  ```bash
  greet (){
   echo "ankit is genius"
  }
  greet
  ```
- **ii) Example:**
  ```bash
  add (){
   echo $(( $1 + $2 ))  // $1,$2 => first ,Second arguments
  }
  add 10 20   // 10 and 20 are the arguments
  ```

---

### 11) Arguments
Arguments are the value which are passed to function when calling it:
- `$1` => first arguments
- `$2` => Second arguments
- `$3` => third arguments
- `$#` => number of arguments
- `$@` => all arguments

> [!NOTE]
> - `$0` refers to the name of the script itself.
> - `$?` refers to the exit status of the last executed command (0 for success, non-zero for error).

---

### 12) Error Handling
To handle errors in shell script, we use the `if` Conditionals.
- `!` => tell us that there is an error on the code
```bash
if ! command ;
then 
   #output message
   exit 1
fi
```

---

### 13) Fallback & Chaining Operators
- `=====> fallback operator => ||`
- `cd ankit || mkdir ankit` => means go to ankit dir, if not existed then create it

> [!NOTE]
> - `||` (OR operator): Runs the second command only if the first command **fails**.
> - `&&` (AND operator): Runs the second command only if the first command **succeeds**.
>   - *Example:* `cd ankit && touch notes.txt` (only creates notes.txt if cd is successful).

---
---

# ☁️ AWS CLI

### Why use AWS CLI?
- **i) First question:** why use aws cli when we can connect to our aws instance using virtualisation (local linux setup)?
- **So the answer is:** to create, start, stop an instance or performing other operations with aws, we have to go their website and perform those task, but you can't automate these tasks. Hence we required AWS CLI.

---

## 🛠️ Complete Guide to AWS CLI (Important Commands)

### 1. Configuration
Before using the AWS CLI, you must authenticate with your AWS credentials:
```bash
aws configure
```
This interactive prompt will ask you for:
- **AWS Access Key ID**: Your programmatic access key.
- **AWS Secret Access Key**: Your programmatic secret key.
- **Default region name**: e.g., `us-east-1`, `ap-south-1`.
- **Default output format**: `json`, `text`, or `table`.

---

### 2. Amazon EC2 (Virtual Servers)
Manage EC2 instances directly from your terminal:

* **List all instances** (with filter and table output):
  ```bash
  aws ec2 describe-instances --output table
  ```
* **Start an instance**:
  ```bash
  aws ec2 start-instances --instance-ids <instance-id>
  ```
* **Stop an instance**:
  ```bash
  aws ec2 stop-instances --instance-ids <instance-id>
  ```
* **Reboot an instance**:
  ```bash
  aws ec2 reboot-instances --instance-ids <instance-id>
  ```
* **Terminate/Delete an instance**:
  ```bash
  aws ec2 terminate-instances --instance-ids <instance-id>
  ```

---

### 3. Amazon S3 (Simple Storage Service)
Store and retrieve files in the cloud:

* **List all S3 buckets**:
  ```bash
  aws s3 ls
  ```
* **Create a new S3 bucket**:
  ```bash
  aws s3 mb s3://<your-unique-bucket-name>
  ```
* **Upload a file to a bucket**:
  ```bash
  aws s3 cp <local-file-path> s3://<your-unique-bucket-name>/
  ```
* **Download a file from a bucket**:
  ```bash
  aws s3 cp s3://<your-unique-bucket-name>/<file-name> <local-destination-path>
  ```
* **Sync a local folder with an S3 bucket**:
  ```bash
  aws s3 sync <local-folder-path> s3://<your-unique-bucket-name>/
  ```
* **Delete a file from a bucket**:
  ```bash
  aws s3 rm s3://<your-unique-bucket-name>/<file-name>
  ```
* **Delete a bucket** (must be empty or use `--force`):
  ```bash
  aws s3 rb s3://<your-unique-bucket-name> --force
  ```

---

### 4. AWS IAM (Identity & Access Management)
Manage users, groups, and permissions:

* **List all IAM users**:
  ```bash
  aws iam list-users
  ```
* **Create a new IAM user**:
  ```bash
  aws iam create-user --user-name <username>
  ```
* **Add user to a group**:
  ```bash
  aws iam add-user-to-group --user-name <username> --group-name <groupname>
  ```

---

### 5. Advanced Querying and Filtering (JMESPath)
You can filter the output of any AWS command to get exactly what you need.

* **Find only the Instance IDs and their current status**:
  ```bash
  aws ec2 describe-instances --query "Reservations[*].Instances[*].[InstanceId,State.Name]" --output table
  ```
* **List the names of all your S3 Buckets**:
  ```bash
  aws s3api list-buckets --query "Buckets[].Name"
  ```
