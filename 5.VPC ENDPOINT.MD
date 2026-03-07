# 🚀 Ultra-Simple 5-Step Lab: VPC Peering + SSM Endpoint (No DNS!) ⏱️ 10 Min

**🔥 Mumbai Lab Ready (ap-south-1)** — Spoke EC2 talks to SSM privately via Hub endpoint! 💰 ~₹0.8/hr

## 🏗️ Minimal Setup ⚙️
```
🔵 Hub VPC: 10.0.0.0/16 (Endpoint Palace 👑)
🔴 Spoke VPC: 10.1.0.0/16 (EC2 Home 🏠)
📍 Private subnets: 10.0.1.0/24 & 10.1.1.0/24
🚀 t3.micro EC2 (Spoke private) + SSM IAM role ✅
```

## 📋 Step 1: Peer VPCs 🔗
```
VPC Console ➜ Peering ➜ ✨ Create
Requester: Spoke | Accepter: Hub ➜ ✅ Create
Actions ➜ 🎉 Accept ➜ Active! 🟢
```

## 📋 Step 2: Add Routes 🛣️
```
🔴 Spoke RT: 10.0.0.0/16 ➜ pcx-xxx
🔵 Hub RT: 10.1.0.0/16 ➜ pcx-xxx
```

## 📋 Step 3: Hub Endpoint Magic 🎇
```
Endpoints ➜ ✨ Create
👉 Service: com.amazonaws.ap-south-1.ssm
🌐 VPC: Hub | Subnet: 10.0.1.0/24
❌ DNS: DISABLE both! 🚫
🛡️ SG: New ➜ TCP 443 from 10.1.0.0/16

📝 Copy: vpce-0123456789abcdef0.ssm.ap-south-1.vpce.amazonaws.com 🪄
```

## 📋 Step 4: Test from Spoke EC2 🧪
```bash
# 🌐 Network ping
nc -zv vpce-0123456789abcdef0.ssm.ap-south-1.vpce.amazonaws.com 443
✅ "succeeded"

# ⚡ SSM magic
aws ssm describe-instance-information \
  --endpoint-url https://vpce-0123456789abcdef0.ssm.ap-south-1.vpce.amazonaws.com \
  --region ap-south-1
✅ Shows YOUR EC2! 🎊
```

## 📋 Step 5: Verify Traffic 📊
```
Hub VPC ➜ Network Interfaces ➜ Filter "vpc-endpoint" ➜ 📈 Metrics spike! 🚀
```

```
🎯 MISSION COMPLETE! 🏆
🔒 Private SSM access | ❌ No NAT/IGW/DNS | 📱 Works Mumbai instantly
💰 ₹0.8/hr endpoint + tiny data xfer
```

## 🔧 CLI Turbo Mode ⚡
```bash
aws ec2 create-vpc-peering-connection --vpc-id vpc-spoke --peer-vpc-id vpc-hub --peer-region ap-south-1
aws ec2 accept-vpc-peering-connection --vpc-peering-connection-id pcx-xxx
# Routes/endpoint next ➜ same pattern!
```

**🌟 Scale:** Add 10+ spokes to 1 Hub endpoint! 👥

**DevOps Pro Tip:** Terraform this for IaC 🚀