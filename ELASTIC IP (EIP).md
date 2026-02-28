# 🚀 **AWS EIP ✨

```
🌐 Elastic IP = Static Public IPv4 
🔄 Survives EC2 stop/start! 
💰 Charges even if idle ❌
⚖️ 5 quota/Region (request more)
```

## 📊 **EIP Rules**
| ✅ Can | ❌ Can't |
|--------|----------|
| EC2 instance | Multiple to 1 IP |
| ENI private IP | EC2-Classic (old) |
| Quick remap | Free forever |

## 🎯 **Console - 3 Clicks**
```
1️⃣ EC2 → Elastic IPs → Allocate → VPC → ✅
   📍 IP: 52.94.12.34
   
2️⃣ Actions → Associate → Instance → Pick EC2 → ✅
   
3️⃣ Test: ssh ubuntu@52.94.12.34  ✅
```

## 💻 **CLI - Copy Paste**
```bash
# 🟢 ALLOCATE
alloc=$(aws ec2 allocate-address --domain vpc \
  --query AllocationId --output text)

# 🔵 ASSOCIATE
aws ec2 associate-address \
  --instance-id i-xxx \
  --allocation-id $alloc
```
**EIP now on instance!** ⚡ 

## 🔗 **With ENI (Secondary NIC)**
```
ENI ens6 (10.0.2.10) → Associate EIP here
CLI: --network-interface-id eni-xxx --private-ip-address 10.0.2.10
```
Public mgmt on private subnet! 🎉
## ⚠️ **Quick Commands**
```bash
# ❌ REMOVE
aws ec2 disassociate-address --association-id eipassoc-xxx
aws ec2 release-address --allocation-id eipalloc-xxx

# 🔍 LIST
aws ec2 describe-addresses
```

```
🎯 Status Check:
- `curl ifconfig.me` = Your EIP? ✅
- Stop/start EC2 = IP same? ✅
```

**Perfect for your ENI multi-NIC setup!** 🛠️