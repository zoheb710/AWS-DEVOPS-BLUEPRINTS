# 🔗 AWS VPC Peering – Practical Lab Guide

Connect two VPCs so EC2 instances talk over **private IPs only**.

***

## 1️⃣ Lab Design

| Item        | Value                  |
|------------|------------------------|
| Region     | `ap-south-1` (any)     |
| VPC-A CIDR | `10.0.0.0/16`          |
| VPC-B CIDR | `10.1.0.0/16` (no overlap) |
| Subnets    | `10.0.1.0/24`, `10.1.1.0/24` |
| EC2        | 1 instance per VPC     |

> ✅ **Goal:** EC2 in VPC-A can reach EC2 in VPC-B via **private IP**.

Reference: [How VPC peering works](https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-basics.html)

***

## 2️⃣ Create The VPC Peering

**Console path:**  
`VPC Console → Peering connections → Create peering connection`

1. **Requester VPC** = VPC-A  
2. **Accepter VPC** = VPC-B (same account/Region for this lab)  
3. After creation:  
   - Select the peering connection  
   - **Actions → Accept request**  
   - Status should become `active`  

> ⚠️ Peering alone does **nothing** until you add routes.

Refs:  
- [VPC Peering (PDF)](https://docs.amazonaws.cn/en_us/vpc/latest/peering/vpc-pg.pdf)  
- [Basics](https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-basics.html)

***

## 3️⃣ Configure Route Tables (Critical)

AWS: you must update route tables **associated with the subnets** where your instances live.  
Ref: [Route tables for peering](https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-routing.html)

### 🔹 VPC-A routes

1. `VPC Console → Route tables`
2. Select route table for subnet of VPC-A EC2  
3. **Edit routes → Add route**  
   - **Destination:** `10.1.0.0/16` (or `10.1.1.0/24`)  
   - **Target:** `pcx-…` (your peering connection)  
4. Save

### 🔹 VPC-B routes

1. Same steps on VPC-B’s subnet route table  
2. **Destination:** `10.0.0.0/16`  
3. **Target:** same `pcx-…`  
4. Save

> 🔁 Until **both** sides have routes, traffic is one‑way or broken.

***

## 4️⃣ Security Groups & NACLs

### 🔸 Security Groups (SG)

On each instance SG (VPC-A and VPC-B):

- Allow **inbound** from peer VPC (or narrower):  
  - `ICMP` (Echo Request) from `10.0.0.0/8` or exact peer CIDR  
  - `TCP 22`, `80`, `443` as needed  

You can also use **SG-to-SG** rules on same-Region peering:  
Ref: [Peer SG references](https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-security-groups.html)

### 🔸 NACLs

On both subnets:

- Ensure NACLs allow the same traffic (remember NACLs are **stateless** → allow both directions).  

Ref (route/NACL basics):  
- [Route tables](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)

***

## 5️⃣ DNS Behaviour (Nice To Have)

By default, VPC peering does **not** give cross‑VPC DNS.  
Ref: [Ensure DNS resolution](https://www.stream.security/rules/ensure-vpc-peering-dns-resolution-is-enabled)

To make instance **public DNS names** resolve to **private IPs across peering**:

1. Ensure both VPCs:  
   - **DNS hostnames** = enabled  
   - **DNS resolution** = enabled  
2. `VPC Console → Peering connections → select pcx → Actions → Edit DNS settings`  
3. Enable DNS resolution for **requester** and **accepter**  

Refs:  
- [VPC peering DNS](https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-dns.html)

> 🧠 Result: when calling the instance’s public DNS from the peer VPC, you get its **private IP** via the peering link.

***

## 6️⃣ Testing & Troubleshooting

From **VPC-A instance**:

```bash
ping <VPC-B-private-ip>

curl http://<VPC-B-private-ip>   # if HTTP server running
```

If it fails, check in this order (matches AWS troubleshooting flow):

1. **Peering status** is `active`.  
   - [Troubleshoot peering](https://docs.aws.amazon.com/vpc/latest/peering/troubleshoot-vpc-peering-connections.html)
2. **Routes** exist in the **right subnet route tables** on both VPCs.  
   - [Routing guide](https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-routing.html)
3. **Security groups** allow traffic from peer CIDR/SG.  
   - [Peer SG refs](https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-security-groups.html)
4. **NACLs** allow traffic (in+out).  
5. **DNS settings** only matter if testing via hostnames.  
   - [DNS over peering](https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-dns.html)

***

## ✅ Next Lab Ideas

- 🔐 **Cross-account peering** (same Region)  
- 🌍 **Inter-Region peering** (CIDR-based SG rules only)  
- 🕸 **Non-transitive demo**: VPC-A↔B and A↔C, show why B can’t reach C