
On your Ubuntu machine, I recommend installing **AWS CLI v2 using AWS’s official installer**, rather than `apt install awscli`, because AWS notes third-party package repositories may not contain the latest CLI version. ([AWS Documentation][1])

### 1. Check whether AWS CLI is already installed

```bash
aws --version
```

If you get:

```text
aws: command not found
```

continue below.

### 2. Check your CPU architecture

```bash
uname -m
```

Most Ubuntu laptops will return:

```text
x86_64
```

If yours says `x86_64`, run:

```bash
sudo apt update
sudo apt install -y curl unzip
```

Then download AWS CLI v2:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

Unzip it:

```bash
unzip awscliv2.zip
```

Install:

```bash
sudo ./aws/install
```

AWS CLI v2 officially supports recent 64-bit Ubuntu/Linux distributions. ([AWS Documentation][2])

### 3. Verify installation

```bash
aws --version
```

You should get output similar to:

```text
aws-cli/2.x.x Python/3.x.x Linux/x86_64
```

Also check where it is installed:

```bash
which aws
```

Normally:

```text
/usr/local/bin/aws
```

### 4. Clean up the installer

```bash
rm -rf aws awscliv2.zip
```

### If your machine says `aarch64`

For ARM Ubuntu, use:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install

aws --version
```

AWS CLI v2 supports Linux ARM as well. ([AWS Documentation][2])

### Final verification

Run these:

```bash
aws --version
which aws
```

Once this works, the **next important step is configuring authentication**. AWS CLI v2 can then be configured with IAM credentials or other supported authentication mechanisms. ([AWS Documentation][3])

For now, run:

```bash
aws --version
```

and send me the output. Then I can take you through AWS CLI configuration and your **first EC2 commands step-by-step**.

[1]: https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html?utm_source=chatgpt.com "Getting started with the AWS CLI"
[2]: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-version.html?utm_source=chatgpt.com "Installing past releases of the AWS CLI version 2"
[3]: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-prereqs.html?utm_source=chatgpt.com "Prerequisites to use the AWS CLI version 2"
