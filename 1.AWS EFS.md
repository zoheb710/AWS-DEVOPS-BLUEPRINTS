# 🚀 AWS EFS & EC2 Setup Guide
### *A Visual Walkthrough to Shared Storage*

---

## 🟢 Phase 1: The Foundation (Infrastructure)
*Let's set up the servers and the storage service.*

- [ ] **1.** **Login** to your AWS Account using the **Root User**.
- [ ] **2.** **Launch 2 EC2 Instances** (Linux) in the **EC2 Dashboard**.
- [ ] **3.** Select **Linux AMIs** (Amazon Machine Image).
- [ ] **4.** **Crucial:** Select **Different Availability Zones (AZs)** for the two instances (e.g., one in `us-east-1a`, one in `us-east-1b`).

---

## 🟡 Phase 2: Creating the EFS (Cloud Storage)
*Now we configure the file system.*

- [ ] **5.** Search for **EFS** in the AWS Search Bar.
- [ ] **6.** Click **"Create File System"**.
- [ ] **7.** Give your EFS a **Name** (e.g., `Shared-Storage`).
- [ ] **8.** **VPC:** Select your default VPC.
- [ ] **9.** Click **"Customize"**.
- [ ] **10.** **Regional:** Select **Regional** (This enables multi-AZ access).
- [ ] **11.** Add **Tags** for better grouping (optional but recommended).
- [ ] **12.** Click **"Next"**.
- [ ] **13.** **Select AZs:** Choose only the **2 AZs** you used for your instances. *Remove the rest.*
- [ ] **14.** **Security Groups:** Select the Security Group attached to your instances. *Remove the default security groups.*
- [ ] **15.** Click **"Next"**.
- [ ] **16.** If you are a new user, you can stick to default settings or select a policy you are familiar with.
- [ ] **17.** **Review** details and click **"Create"**.
- [ ] **18.** ✅ **EFS is Created!** Wait for the status to become "Available".

---

## 🔵 Phase 3: Instance Prep (Terminal Work)
*We need to prepare the servers to speak the "NFS" language.*

- [ ] **19.** **SSH** into both instances (One via CLI/putty, one via another terminal).
- [ ] **20.** **Install APT:**
    ```bash
    sudo apt install -y
    ```
- [ ] **21.** **Update APT:**
    ```bash
    sudo apt update -y
    ```
- [ ] **22.** **Install NFS Client:**
    ```bash
    sudo apt install nfs-common -y
    ```

---

## 🟣 Phase 4: Mounting the Drive
*Connecting the storage to the servers.*

- [ ] **23.** Go back to **EFS Dashboard** → Click **"View Details"**.
- [ ] **24.** Click the **"Attach"** button.
- [ ] **25.** Select **"Mount via IP"** (Recommended for Ubuntu/Linux).
- [ ] **26.** **Check AZs:** Ensure you are copying the IP specific to the Availability Zone of the instance you are mounting.
- [ ] **27.** In **Instance 1** (SSH Terminal), create the mount folder:
    ```bash
    sudo mkdir efs
    ```
    *(Repeat this command in **Instance 2** as well).*

- [ ] **28 & 29.** **Mount Command:**
    *Copy the command from the AWS Attach window. It will look like this (example):*

    ```bash
    sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport <EFS-IP>:/ efs
    ```
    *Run this in **both** instances, ensuring you use the IP relevant to that instance's AZ.*

---

## 🟥 Phase 5: Testing (Verification)
*Making sure the magic happens.*

- [ ] **30.** Go to **Instance 1**:
    ```bash
    cd efs
    sudo touch file{1..10}
    ls -l
    ```
- [ ] **31.** Go to **Instance 2** and check:
    ```bash
    cd efs
    ls -l
    ```
- [ ] **32.** 🎉 **Success!** You will see `file1` through `file10` in the second instance as well.

---

### 📝 **Pro Tip:**
To ensure the EFS mounts automatically every time you reboot your instances, you must add an entry to your `/etc/fstab` file. Run this in both instances:

```bash
sudo nano /etc/fstab
```
*Add this line to the bottom (replace with your IP):*
```text
172.31.xx.xx:/ /efs nfs4 defaults,_netdev 0 0
```