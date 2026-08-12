Yes — the AWS documentation you pasted reflects the **current recommended Linux installation method**, and it is simpler than the ZIP-based method I gave earlier.

AWS now recommends the **install script** for installing the latest AWS CLI v2 on Linux. The ZIP installer is still valid, especially if you need to pin a specific CLI version, but for your local Ubuntu machine I would use the install script. ([AWS Documentation][1])

For your Ubuntu machine, just do this:

```bash
curl -fsSL https://awscli.amazonaws.com/v2/install.sh | bash
```

This installs AWS CLI for **your current user** under:

```text
$HOME/.local/share/aws-cli
```

and creates the command symlink under:

```text
$HOME/.local/bin
```

Then verify:

```bash
aws --version
```

If you get:

```text
aws: command not found
```

check:

```bash
echo $PATH
```

and:

```bash
ls ~/.local/bin/aws
```

If `~/.local/bin` is not in your PATH, run:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Then make it permanent:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Then:

```bash
aws --version
```

### If you want AWS CLI available for all users

Use the system installation from the AWS documentation:

```bash
curl -fsSL https://awscli.amazonaws.com/v2/install.sh | sudo bash -s -- --system
```

That installs under:

```text
/usr/local/aws-cli
```

with the command available through:

```text
/usr/local/bin/aws
```

Then:

```bash
which aws
aws --version
```

You should see something similar to:

```text
/usr/local/bin/aws

aws-cli/2.x.x Python/3.x.x Linux/x86_64
```

### Which method should you use?

For **your local Ubuntu learning machine**, I suggest the current-user installation:

```bash
curl -fsSL https://awscli.amazonaws.com/v2/install.sh | bash
```

Why? You don't need `sudo`, it doesn't modify the system installation, and it is easy to manage.

For a **shared Jenkins/EC2 server**, I'd normally use:

```bash
curl -fsSL https://awscli.amazonaws.com/v2/install.sh | sudo bash -s -- --system
```

because Jenkins and other service users may need access to the `aws` executable.

The older ZIP approach:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" ...
unzip ...
sudo ./aws/install
```

is **not wrong**. AWS still documents the command-line installer; it is particularly useful when you want explicit version control. ([AWS Documentation][1])

So for you now, start with:

```bash
curl -fsSL https://awscli.amazonaws.com/v2/install.sh | bash
```

then:

```bash
aws --version
```

Once that works, the next useful lesson is **how AWS CLI authentication actually works: `aws configure` vs IAM users vs AWS SSO vs EC2 IAM roles**, because that directly connects to your Jenkins/EC2 curriculum.

[1]: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html?utm_source=chatgpt.com "Installing or updating to the latest version of the AWS CLI"
