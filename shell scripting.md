# Important Linux Commands

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
>
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
>
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

- **Comparison Operators:**
  - **Numeric Comparisons:** `-eq` (equal to), `-ne` (not equal to), `-gt` (greater than), `-ge` (greater than or equal to), `-lt` (less than), `-le` (less than or equal to)
  - **String Comparisons:** `==` (equal to), `!=` (not equal to), `-z` (is empty), `-n` (is not empty)

We use the syntax written below:

```bash
if [[ condition ]]    // -eq, -ne, -gt, -lt, ==, !=, etc.
then
   #write your output
elif [[ condition ]]
then
   #write your output
else
   #write your output
fi    // this is for exit the if else condition
```

> [!IMPORTANT]
> **Bash Syntax Note:** In standard Bash syntax, the `else` block does not require a `then` keyword. It is typically written as:
>
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
>
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

_///// always use vim for shell Scripting_

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
>
> - `$0` refers to the name of the script itself.
> - `$?` refers to the exit status of the last executed command (0 for success, non-zero for error).

---

### 12) Error Handling & Strict Mode (`set -eou pipefail`)

By default, Bash continues executing a script line-by-line even if a command fails. To make scripts robust, secure, and production-ready, we use **Strict Mode** with the `set` command.

#### i) The `set -eou pipefail` Strict Mode

Placing `set -eou pipefail` at the top of your script (right below the shebang `#!/bin/bash`) forces Bash to fail fast whenever something goes wrong.

```bash
#!/bin/bash
set -eou pipefail
```

**Detailed Breakdown of Options:**

- **`set -e` (Exit on Error):**
  - **What it does:** Instantly terminates script execution if any command returns a non-zero (failure) exit code.
  - **Why it matters:** Prevents a script from executing dangerous downstream commands (like deleting files) if a prerequisite command (like changing directory) fails.
  - **Exception:** Commands used in conditional statements (e.g., `if command; then ...`, `command || true`, or `while`) will not trigger an exit.

- **`set -o pipefail` (Pipeline Failure Handling):**
  - **What it does:** By default, in a pipeline (`cmd1 | cmd2 | cmd3`), Bash only returns the exit status of the *last* command (`cmd3`). `set -o pipefail` ensures the entire pipeline returns a failure status if *any* command in the chain fails.
  - **Why it matters:** If `cmd1` fails but `cmd2` succeeds, default Bash ignores `cmd1`'s failure. `pipefail` catches it.

- **`set -u` (Unset Variable Error):**
  - **What it does:** Treats references to uninitialized or unset variables as an error and exits immediately.
  - **Why it matters:** Prevents catastrophic bugs like `rm -rf "$DIRECTORY/"` accidentally executing as `rm -rf /` if `$DIRECTORY` is misspelled or unassigned.
  - **Workaround:** If a variable might be optional, use default expansion: `${VAR:-default_value}`.

> [!TIP]
> **Debugging Mode (`set -x` / `set -eoux pipefail`):**
> Adding `-x` enables **tracing mode**. It prints every command along with its expanded arguments to the terminal before executing it, making script debugging much easier.

#### ii) Manual Error Handling (`$?` & `if !`)

Besides Strict Mode, you can manually catch and handle errors using conditionals:

- **Using `!` (NOT operator):**
  ```bash
  if ! mkdir /protected_directory; then
     echo "Failed to create directory!"
     exit 1
  fi
  ```
- **Checking Exit Status (`$?`):**
  ```bash
  ping -c 1 google.com
  if [ $? -ne 0 ]; then
     echo "Network is down!"
     exit 1
  fi
  ```

---

### 13) Linux Standard Streams & Redirections (0, 1, 2 & `>& /dev/null`)

In Linux, every process uses three standard file descriptors (streams) for input, output, and error handling:

| File Descriptor | Stream Name | Description | Default Source / Destination |
| :--- | :--- | :--- | :--- |
| **`0`** | **stdin** (Standard Input) | Data fed into a command | Keyboard / Input file |
| **`1`** | **stdout** (Standard Output) | Normal output of a command | Terminal screen |
| **`2`** | **stderr** (Standard Error) | Error messages & diagnostics | Terminal screen |

#### i) Redirection Operators Summary

- **`>`** (Redirect stdout): Overwrites target file with stdout.
  - *Example:* `echo "Hello" > output.txt`
- **`>>`** (Append stdout): Appends stdout to the end of a file.
  - *Example:* `echo "Log entry" >> log.txt`
- **`<`** (Redirect stdin): Reads stdin from a file instead of keyboard.
  - *Example:* `cat < input.txt`
- **`2>`** (Redirect stderr): Redirects error messages to a file.
  - *Example:* `ls /nonexistent 2> error.log`
- **`2>>`** (Append stderr): Appends error messages to a file.
  - *Example:* `ls /nonexistent 2>> error.log`

#### ii) Understanding `/dev/null` and Output Suppression (`>& /dev/null`)

`/dev/null` is a special virtual device file in Linux known as the **"black hole"**. Any data written to it is permanently discarded.

##### 1. Syntax Variations
- **`>/dev/null 2>&1` (POSIX standard syntax):**
  - `>/dev/null` redirects stdout (descriptor 1) to `/dev/null`.
  - `2>&1` redirects stderr (descriptor 2) to wherever stdout is currently pointing (`/dev/null`).
- **`>& /dev/null` or `&> /dev/null` (Bash Shorthand):**
  - Concise syntax in Bash to redirect both stdout (1) and stderr (2) to `/dev/null` simultaneously.

##### 2. Practical Linux Command Use Cases

- **Silent Command Execution (Check command existence without output):**
  ```bash
  if command -v git >/dev/null 2>&1; then
      echo "Git is installed!"
  fi
  ```

- **Suppressing Error Messages Only:**
  ```bash
  # Search system files while ignoring 'Permission denied' stderr messages
  find / -name "config" 2>/dev/null
  ```

- **Logging stdout & stderr to Separate Files:**
  ```bash
  # Save clean logs to app.log and errors to error.log
  ./deploy.sh > app.log 2> error.log
  ```

- **Combining stdout & stderr into One Log File:**
  ```bash
  # Redirect both normal output and errors into combined.log
  ./deploy.sh > combined.log 2>&1
  # Or using Bash shortcut:
  ./deploy.sh &> combined.log
  ```

- **Feeding Input via File (stdin redirection):**
  ```bash
  # Send SQL commands from file directly into MySQL database
  mysql -u root -p my_database < schema.sql
  ```

---

### 14) Fallback & Chaining Operators

- `=====> fallback operator => ||`
- `cd ankit || mkdir ankit` => means go to ankit dir, if not existed then create it

> [!NOTE]
>
> - `||` (OR operator): Runs the second command only if the first command **fails**.
> - `&&` (AND operator): Runs the second command only if the first command **succeeds**.
>   - _Example:_ `cd ankit && touch notes.txt` (only creates notes.txt if cd is successful).

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

  after this run the command => aws configure list
  if this works , everything is good else not

---

### 2. Amazon EC2 (Virtual Servers)

Manage EC2 instances directly from your terminal:

- **List all instances** (with filter and table output):
  ```bash
  aws ec2 describe-instances --output table
  ```
- **Launch a new instance**:
  ```bash
  aws ec2 run-instances --image-id <ami-id> --count 1 --instance-type t2.micro --key-name <key-pair-name> --security-group-ids <sg-id>
  ```
- **Start an instance**:
  ```bash
  aws ec2 start-instances --instance-ids <instance-id>
  ```
- **Stop an instance**:
  ```bash
  aws ec2 stop-instances --instance-ids <instance-id>
  ```
- **Reboot an instance**:
  ```bash
  aws ec2 reboot-instances --instance-ids <instance-id>
  ```
- **Terminate/Delete an instance**:
  ```bash
  aws ec2 terminate-instances --instance-ids <instance-id>
  ```
- **Get Public IP address of running instances**:
  ```bash
  aws ec2 describe-instances --query "Reservations[*].Instances[*].[InstanceId,PublicIpAddress]" --output text
  ```

> [!TIP]
> **EC2 Scripting Notes:**
>
> - **Filtering by State:** Use `--filters "Name=instance-state-name,Values=running"` to fetch only running instances.
> - **Extracting Instance ID in Shell Scripts:**
>   `INSTANCE_ID=$(aws ec2 describe-instances --query "Reservations[0].Instances[0].InstanceId" --output text)`
> - Always double-check security group rules and key pairs before launching instances programmatically.

---

### 3. Amazon S3 (Simple Storage Service)

Store and retrieve files in the cloud:

- **List all S3 buckets**:
  ```bash
  aws s3 ls
  ```
- **Create a new S3 bucket**:
  ```bash
  aws s3 mb s3://<your-unique-bucket-name>
  ```
- **Upload a file to a bucket**:
  ```bash
  aws s3 cp <local-file-path> s3://<your-unique-bucket-name>/
  ```
- **Download a file from a bucket**:
  ```bash
  aws s3 cp s3://<your-unique-bucket-name>/<file-name> <local-destination-path>
  ```
- **Sync a local folder with an S3 bucket**:
  ```bash
  aws s3 sync <local-folder-path> s3://<your-unique-bucket-name>/
  ```
- **Delete a file from a bucket**:
  ```bash
  aws s3 rm s3://<your-unique-bucket-name>/<file-name>
  ```
- **Delete all contents inside a bucket**:
  ```bash
  aws s3 rm s3://<your-unique-bucket-name> --recursive
  ```
- **Delete a bucket** (must be empty or use `--force`):
  ```bash
  aws s3 rb s3://<your-unique-bucket-name> --force
  ```

> [!NOTE]
> **S3 Scripting Notes:**
>
> - **Bucket Naming Rules:** Bucket names must be **globally unique** across all AWS accounts worldwide, lowercase, and contain no underscores or spaces.
> - **High-Level vs Low-Level CLI:** `aws s3` provides simple high-level file operations (`cp`, `sync`, `mv`, `rm`), whereas `aws s3api` provides direct, low-level access to all S3 API capabilities (like configuring ACLs or bucket policies).
> - **Checking Bucket Existence in Scripts:**
>   ```bash
>   if aws s3 ls "s3://my-bucket-name" 2>&1 | grep -q 'NoSuchBucket'; then
>       echo "Bucket does not exist!"
>   fi
>   ```

---

### 4. AWS IAM (Identity & Access Management)

Manage users, groups, and permissions:

- **List all IAM users**:
  ```bash
  aws iam list-users
  ```
- **Create a new IAM user**:
  ```bash
  aws iam create-user --user-name <username>
  ```
- **Create Access Key for a user** (for CLI access):
  ```bash
  aws iam create-access-key --user-name <username>
  ```
- **Add user to a group**:
  ```bash
  aws iam add-user-to-group --user-name <username> --group-name <groupname>
  ```
- **Attach a managed policy to a user**:
  ```bash
  aws iam attach-user-policy --user-name <username> --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
  ```
- **Delete an IAM user**:
  ```bash
  aws iam delete-user --user-name <username>
  ```

> [!WARNING]
> **IAM Security Notes:**
>
> - **Principle of Least Privilege:** Grant users only the minimum permissions required to complete their job. Avoid granting `AdministratorAccess` unless strictly necessary.
> - **Root Account Usage:** Never use Root Account credentials for daily CLI operations or scripts. Always create dedicated IAM users or roles.
> - **Access Key Management:** Treat Access Key ID and Secret Access Key like passwords. Never hardcode credentials directly inside shell scripts—use `aws configure`, environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`), or IAM Roles.

---

### 5. AWS Lambda (Serverless Computing)

Run code without provisioning or managing servers. You only pay for the compute time consumed:

- **List all Lambda functions**:
  ```bash
  aws lambda list-functions
  ```
- **Create/Deploy a new Lambda function**:
  ```bash
  aws lambda create-function --function-name <function-name> --runtime python3.9 --role <iam-role-arn> --handler <file-name.method-name> --zip-file fileb://<path-to-zip-file>
  ```
- **Invoke a Lambda function**:
  ```bash
  aws lambda invoke --function-name <function-name> response.json
  ```
- **Update Lambda function code**:
  ```bash
  aws lambda update-function-code --function-name <function-name> --zip-file fileb://<path-to-new-zip-file>
  ```
- **Get details of a function**:
  ```bash
  aws lambda get-function --function-name <function-name>
  ```
- **Delete a Lambda function**:
  ```bash
  aws lambda delete-function --function-name <function-name>
  ```

> [!NOTE]
> **Lambda Scripting Notes:**
>
> - **The `fileb://` prefix:** When passing binary zip files in AWS CLI commands (like `create-function` or `update-function-code`), you **must** use the `fileb://` prefix (e.g. `fileb://my-function.zip`). Using standard `file://` will lead to encoding errors.
> - **Logs:** Execution logs for Lambda functions are automatically sent to **Amazon CloudWatch Logs** under the log group `/aws/lambda/<function-name>`.

---

### 6. Amazon DynamoDB (NoSQL Database)

Fully managed, serverless NoSQL database designed for fast performance at scale:

- **List all tables**:
  ```bash
  aws dynamodb list-tables
  ```
- **Create a new table**:
  ```bash
  aws dynamodb create-table --table-name <table-name> --attribute-definitions AttributeName=<PartitionKey>,AttributeType=S --key-schema AttributeName=<PartitionKey>,KeyType=HASH --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
  ```
- **Describe table details**:
  ```bash
  aws dynamodb describe-table --table-name <table-name>
  ```
- **Insert or update an item (PutItem)**:
  ```bash
  aws dynamodb put-item --table-name <table-name> --item '{"<PartitionKey>": {"S": "<value>"}, "<AttributeName>": {"S": "<value>"}}'
  ```
- **Retrieve an item by primary key (GetItem)**:
  ```bash
  aws dynamodb get-item --table-name <table-name> --key '{"<PartitionKey>": {"S": "<value>"}}'
  ```
- **Scan all items in a table**:
  ```bash
  aws dynamodb scan --table-name <table-name>
  ```
- **Query items with a specific Partition Key**:
  ```bash
  aws dynamodb query --table-name <table-name> --key-condition-expression "<PartitionKey> = :val" --expression-attribute-values '{":val":{"S":"<value>"}}'
  ```
- **Delete a table**:
  ```bash
  aws dynamodb delete-table --table-name <table-name>
  ```

> [!TIP]
> **DynamoDB Scripting Notes:**
>
> - **Query vs Scan:** Always prefer `query` over `scan` when searching for items. `query` looks up data directly using index keys, whereas `scan` reads the entire table, consuming significantly more Read Capacity Units (RCUs) and execution time.
> - **Data Types in CLI:** DynamoDB requires explicit type wrappers in CLI parameters: `S` for String, `N` for Number, `B` for Binary, `BOOL` for Boolean, `M` for Map, and `L` for List.

---

### 7. Amazon ECS & Amazon EKS (Container Services)

Manage and deploy containerized workloads using AWS Elastic Container Service (ECS) or Elastic Kubernetes Service (EKS).

#### i) Amazon ECS (AWS Elastic Container Service)

- **List ECS Clusters**:
  ```bash
  aws ecs list-clusters
  ```
- **Create an ECS Cluster**:
  ```bash
  aws ecs create-cluster --cluster-name <cluster-name>
  ```
- **Register a Task Definition**:
  ```bash
  aws ecs register-task-definition --cli-input-json file://<task-definition.json>
  ```
- **List running tasks in a cluster**:
  ```bash
  aws ecs list-tasks --cluster <cluster-name>
  ```
- **Create a Service**:
  ```bash
  aws ecs create-service --cluster <cluster-name> --service-name <service-name> --task-definition <task-def-name> --desired-count 1
  ```
- **Update a Service (Force New Deployment)**:
  ```bash
  aws ecs update-service --cluster <cluster-name> --service <service-name> --force-new-deployment
  ```

#### ii) Amazon EKS (AWS Elastic Kubernetes Service)

- **List EKS Clusters**:
  ```bash
  aws eks list-clusters
  ```
- **Create an EKS Cluster**:
  ```bash
  aws eks create-cluster --name <cluster-name> --role-arn <role-arn> --resources-vpc-config subnetIds=<subnet-1>,<subnet-2>
  ```
- **Configure `kubectl` to connect to EKS**:
  ```bash
  aws eks update-kubeconfig --region <region-name> --name <cluster-name>
  ```
- **Describe EKS Cluster Status**:
  ```bash
  aws eks describe-cluster --name <cluster-name>
  ```
- **List Node Groups in a cluster**:
  ```bash
  aws eks list-nodegroups --cluster-name <cluster-name>
  ```
- **Delete an EKS Cluster**:
  ```bash
  aws eks delete-cluster --name <cluster-name>
  ```

> [!NOTE]
> **Container Service Notes:**
>
> - **EKS & `kubectl` Integration:** Running `aws eks update-kubeconfig` creates/updates your local `~/.kube/config` file so you can manage your EKS Kubernetes cluster using standard `kubectl` commands.
> - **EKS Tooling:** For managing EKS clusters via CLI, `eksctl` is often used alongside the standard AWS CLI to simplify cluster creation, node group provisioning, and IAM roles for service accounts (IRSA).

---

### 8. Advanced Querying and Filtering (JMESPath)

You can filter the output of any AWS command to get exactly what you need.

- **Find only the Instance IDs and their current status**:
  ```bash
  aws ec2 describe-instances --query "Reservations[*].Instances[*].[InstanceId,State.Name]" --output table
  ```
- **List the names of all your S3 Buckets**:
  ```bash
  aws s3api list-buckets --query "Buckets[].Name"
  ```
