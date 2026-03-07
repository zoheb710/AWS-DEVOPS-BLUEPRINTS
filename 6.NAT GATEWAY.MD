# 🌐 Build a VPC with Public & Private Subnets, NAT Gateway, and EC2 Instances

> **Goal:** Create a secure AWS VPC with a public Ubuntu EC2 (bastion host) and a private Ubuntu EC2 that can access the internet via a NAT Gateway—without having its own public IP.  

---

## 🎯 Target Architecture

- **VPC:** `10.0.0.0/16`  
- **Public Subnet:** `10.0.1.0/24` – hosts NAT Gateway + public Ubuntu EC2 (bastion)  
- **Private Subnet:** `10.0.2.0/24` – hosts private Ubuntu EC2  
- **Internet Gateway (IGW)** attached to VPC  
- **NAT Gateway** in public subnet with Elastic IP  
- **Route Tables:**  
  - Public → IGW for internet  
  - Private → NAT Gateway for outbound internet  

---

## 🧱 Step 1: Create VPC and Subnets

1. Go to **VPC Dashboard** → **Create VPC**  
   - **Resources:** `VPC only`  
   - **Name:** `lab-nat-vpc`  
   - **IPv4 CIDR:** `10.0.0.0/16`  

2. Create **Subnets** (under VPC → Subnets → Create subnet)  
   - Select `lab-nat-vpc`  

   | Name               | Availability Zone | CIDR Block     |
   |--------------------|-------------------|----------------|
   | `public-ubuntu-a`  | us-east-1a        | `10.0.1.0/24`  |
   | `private-ubuntu-a` | us-east-1a        | `10.0.2.0/24`  |

3. **Enable auto-assign public IPv4** on the public subnet:  
   - Select `public-ubuntu-a` → **Actions** → **Edit subnet settings** → Enable **Auto-assign public IPv4 address** → Save.

---

## 🌐 Step 2: Internet Gateway and Route Tables

1. **Create Internet Gateway**  
   - Name: `lab-igw` → Create → Attach to VPC `lab-nat-vpc`.

2. **Create Route Tables**  
   - `rt-public` (for public subnet)  
   - `rt-private` (for private subnet)

3. **Associate Subnets**  
   - `rt-public` → **Subnet associations** → Add `public-ubuntu-a`  
   - `rt-private` → **Subnet associations** → Add `private-ubuntu-a`

4. **Add Routes**  
   - **Public route table (`rt-public`)**:  
     - `10.0.0.0/16` → `local` (already there)  
     - `0.0.0.0/0` → **Target:** `lab-igw`  
   - **Private route table (`rt-private`)**: (will be updated after NAT creation)

---

## 🔌 Step 3: NAT Gateway + Private Routes

1. **Allocate an Elastic IP**  
   - EC2 Dashboard → **Elastic IPs** → Allocate → Name: `nat-eip`

2. **Create NAT Gateway**  
   - VPC → **NAT Gateways** → Create  
   - Name: `lab-nat-gw`  
   - Subnet: `public-ubuntu-a`  
   - Connectivity type: **Public**  
   - Elastic IP: Select `nat-eip`  
   - Wait for status to become **Available**.

3. **Update Private Route Table**  
   - Go to `rt-private` → **Routes** → Edit  
   - Add route:  
     - Destination: `0.0.0.0/0`  
     - Target: `lab-nat-gw` (NAT Gateway ID)  

✅ Now instances in private subnet will route internet traffic through the NAT Gateway.

---

## 🔒 Step 4: Define Security Groups

Create two security groups (under EC2 → Security Groups):

| Security Group        | Inbound Rules                                      | Outbound Rules         |
|-----------------------|----------------------------------------------------|------------------------|
| `sg-public-bastion`   | SSH (TCP 22) from **your IP** (e.g. `x.x.x.x/32`) | All traffic to `0.0.0.0/0` |
| `sg-private-ubuntu`   | SSH (TCP 22) from **`sg-public-bastion`**         | All traffic to `0.0.0.0/0` |

> **Why?**  
> - Public instance only allows SSH from your IP.  
> - Private instance only allows SSH from the public instance (bastion).  
> - Both can reach the internet for updates (outbound all).

---

## 🚀 Step 5: Launch EC2 Instances

### 🖥️ Public Ubuntu (Bastion) – `ubuntu-public`
- **AMI:** Ubuntu 22.04 LTS  
- **Instance type:** `t3.micro` (Free tier eligible)  
- **Network:** `lab-nat-vpc`  
- **Subnet:** `public-ubuntu-a`  
- **Auto-assign public IP:** Enabled  
- **Security group:** `sg-public-bastion`  
- **Key pair:** Select existing (e.g., `znew.pem`)  

> After launch, note its **public IP** (for SSH) and **private IP** (for internal reference).

### 🔒 Private Ubuntu – `ubuntu-private`
- **AMI:** Ubuntu 22.04 LTS  
- **Instance type:** `t3.micro`  
- **Network:** `lab-nat-vpc`  
- **Subnet:** `private-ubuntu-a`  
- **Auto-assign public IP:** Disabled  
- **Security group:** `sg-private-ubuntu`  
- **Key pair:** Same as above (`znew.pem`)  

> Note its **private IP** (e.g., `10.0.2.23`) – you'll need this to SSH from the bastion.

---

## 🔑 Step 6: SSH into Private Instance via Bastion

From your local machine (using WSL, Git Bash, or Linux/Mac):

```bash
# Fix key permissions (if needed)
chmod 400 znew.pem

# SSH into public bastion
ssh -i znew.pem ubuntu@<public-ec2-ip>

# From bastion, SSH into private instance
ssh ubuntu@10.0.2.23   # replace with private IP of ubuntu-private
```

> **Pro tip:** Use `ssh-agent` to forward your key so you don’t need to copy the .pem file to the bastion:
> ```bash
> ssh-add znew.pem
> ssh -A ubuntu@<public-ec2-ip>
> ```

---

## ✅ Step 7: Test NAT Gateway Functionality

On the **private Ubuntu** instance:

```bash
# Check outbound public IP (should be the NAT Gateway's Elastic IP)
curl https://ifconfig.me

# Test internet connectivity
ping 8.8.8.8
sudo apt update   # should work
```

You should see the NAT Gateway's public IP returned.  
Also verify that no inbound connections are possible: there is no public IP assigned, and the private route table only allows outbound via NAT.

---

## 💻 (Optional) Step 8: AWS CLI Snippet for NAT Route

If you prefer the CLI, after creating the NAT Gateway, you can add the route with:

```bash
aws ec2 create-route \
  --route-table-id <rt-private-id> \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id <nat-gw-id>
```

---

## 🧹 Step 9: Cleanup – Avoid Unwanted Costs

🚨 **Important:** NAT Gateways and Elastic IPs incur charges even when not in use. Always clean up after your lab.

1. Terminate both EC2 instances.  
2. Delete the NAT Gateway (`lab-nat-gw`).  
3. Release the Elastic IP (`nat-eip`).  
4. Delete the route tables (`rt-public`, `rt-private`).  
5. Detach and delete the Internet Gateway (`lab-igw`).  
6. Delete the subnets (`public-ubuntu-a`, `private-ubuntu-a`).  
7. Finally, delete the VPC (`lab-nat-vpc`).  

---

## 🎉 You Did It!

You’ve successfully built a secure, two-tier VPC with a NAT Gateway, allowing a private instance to access the internet while remaining inaccessible from outside. This architecture is commonly used for backend servers, databases, or any workload that needs outbound connectivity without exposing a public IP.