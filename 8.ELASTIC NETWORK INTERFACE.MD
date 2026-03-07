# 🚀 AWS ENI Guide -✨

## 🔧 **What is ENI?**
```
💻 Elastic Network Interface = Virtual NIC for EC2
🔄 Create → Attach → Detach → Move (IPs/SGs follow!)
📍 Must be SAME Availability Zone
```

**🎯 ENI Basics:**
| Feature | Primary (eth0) | Secondary (eth1+) |
|---------|----------------|-------------------|
| 🔒 Detach? | ❌ **NO** | ✅ **YES** |
| 📡 IPs | Primary + many | Primary + many |
| 🛡️ SGs | ✅ Yes | ✅ Yes |
| ⚙️ MAC | Fixed | Fixed |

## 💡 **DevOps Use Case** 
```
🌐 ENI-0 (Primary):  App traffic → Subnet-A + app-sg 🚀
🔒 ENI-1 (Mgmt):     Backup/SSH → Subnet-B + mgmt-sg 🔐
```
**Different SGs + Routes per NIC!** 🎉

## 📱 **Console - 4 Steps** 
```
1️⃣ CREATE: EC2 → Network Interfaces → Create (subnet+SG+IP)
   ↓
2️⃣ ATTACH: Actions → Attach → Instance + Device "1"
   ↓  
3️⃣ OS: SSH → `ip a` (ens6 shows?) → Usually auto! ⚡
   ↓
4️⃣ DETACH: Actions → Detach → Move to new EC2 🔄
```

## 💻 **CLI - Copy-Paste Ready**
```bash
# 🟢 CREATE
eni=$(aws ec2 create-network-interface \
  --subnet-id subnet-xxx \
  --groups sg-xxx \
  --description "mgmt" \
  --query NetworkInterfaceId --output text)

# 🔵 ATTACH (hot OK!)
aws ec2 attach-network-interface \
  --network-interface-id $eni \
  --instance-id i-xxx \
  --device-index 1
```
**Hot attach = no reboot!** 🚀

## ⚠️ **Gotchas - Don't Break!**
```
🔴 Same subnet multi-ENI → Asymmetric routing → Use secondary IPs on primary
🟡 Source/Dest Check ON → Disable for NAT: `--no-source-dest-check`
❌ No bandwidth bonding/team
```

## 🎨 **Quick Test**
```bash
ssh ubuntu@primary-ip
ip a          # ens5 + ens6? ✅
ping -I ens6 8.8.8.8  # Works? 🎉
```

**Which OS?** 🐧Ubuntu / 🐳Amazon Linux / 🪟Windows  
**Pick one → Get exact OS config!**