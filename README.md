#CC

---

### **Experiment 1: Working with EC2 Instances**

#### **Task 1: Launch an EC2 Instance using Ubuntu OS and connect using SSH/EC2 Instance Connect**

1. Go to the AWS Console.
2. Navigate to **EC2 > Instances > Launch Instance**.
3. Choose **Ubuntu Server 22.04 LTS (or similar)**.
4. Choose instance type (e.g., **t2.micro** for free tier).
5. Click **Next: Configure Instance Details**.
6. Leave default VPC and subnet, ensure Auto-assign Public IP is **enabled**.
7. Click **Next** until **Add Tags** – add a Name tag (e.g., UbuntuInstance).
8. In **Configure Security Group**, allow SSH (port 22).
9. Click **Review and Launch**, then **Launch**.
10. Create a new key pair or use existing one.
11. Once instance is running, connect via:
    - **EC2 Instance Connect** (in browser)
    - **SSH** from terminal:
      ```bash
      chmod 400 your-key.pem
      ssh -i "your-key.pem" ubuntu@<Public-IP>
      ```

#### **Task 2: Install Nginx and Access Webpage**

1. SSH into your instance.
2. Run:
   ```bash
   sudo apt update
   sudo apt install nginx -y
   ```
3. Check Nginx status:
   ```bash
   systemctl status nginx
   ```
4. Go to your browser and enter the instance's **Public IP**.
5. You should see the **Nginx welcome page**.

#### **Task 3: Launch EC2 with Amazon Linux and connect using PuTTY**

1. Repeat steps from Task 1, but choose **Amazon Linux 2023 AMI**.
2. Download `.pem` key file.
3. Convert `.pem` to `.ppk` using PuTTYgen.
4. Open **PuTTY**, enter Public IP in Hostname.
5. Go to SSH > Auth, browse for `.ppk` file.
6. Click Open to connect.

#### **Task 4: Launch EC2 Windows Instance and Connect via RDP**

1. Launch new EC2 instance with **Windows Server**.
2. Allow **RDP (3389)** in security group.
3. After instance is running, click **Connect > RDP Client**.
4. Download **Remote Desktop File**.
5. Get **password** using your `.pem` key.
6. Open RDP and connect with username (Administrator) and decrypted password.

---

### **Experiment 2: Deploy a Front-End App using Docker**

#### **Task 1: Install Docker on EC2**

1. Launch an Ubuntu EC2 Instance.
2. SSH into the instance.
3. Run:
   ```bash
   sudo apt update
   sudo apt install docker.io -y
   sudo systemctl start docker
   sudo systemctl enable docker
   sudo usermod -aG docker ubuntu
   ```
4. Log out and back in to apply group changes.

#### **Task 2: Clone Frontend App and Build Docker Image**

1. Install git:
   ```bash
   sudo apt install git -y
   ```
2. Clone the app:
   ```bash
   git clone https://tinyurl.com/demokmit3
   cd <repo-directory>
   ```
3. Create a `Dockerfile`:
   ```Dockerfile
   FROM node:18
   WORKDIR /app
   COPY . .
   RUN npm install
   RUN npm run build
   EXPOSE 3000
   CMD ["npm", "start"]
   ```
4. Build image:
   ```bash
   docker build -t frontend-app .
   ```

#### **Task 3: Create Docker Container**

```bash
   docker run -d -p 80:3000 frontend-app
```

#### **Task 4: Allow Web Traffic**

1. Go to **Security Groups** for your EC2 instance.
2. Edit inbound rules – Add **HTTP (port 80)**.
3. Access the application via Public IP.

---

### **Experiment 3: Amazon EBS**

#### **Task 1: Modify Volume Size**

1. Launch EC2 and go to **Volumes**.
2. Select the root volume > Actions > Modify Volume.
3. Increase the size (e.g., 8 GB → 15 GB).
4. On EC2 terminal:
   ```bash
   sudo growpart /dev/xvda 1
   sudo resize2fs /dev/xvda1
   ```

#### **Task 2: Create, Attach, and Mount Volume**

1. Create new volume in same AZ.
2. Attach to EC2 as `/dev/xvdf`.
3. Format and mount:
   ```bash
   sudo mkfs -t ext4 /dev/xvdf
   sudo mkdir /mnt/mydata
   sudo mount /dev/xvdf /mnt/mydata
   echo "YourRollNo" | sudo tee /mnt/mydata/rollno.txt
   ```

#### **Task 3: Detach and Attach to New Instance**

1. Unmount and detach volume.
   ```bash
   sudo umount /mnt/mydata
   ```
2. Stop instance, then detach from console.
3. Attach to another EC2 and mount again.

#### **Task 4: Snapshot and Copy Across Regions**

1. Select volume > Create Snapshot.
2. After snapshot is ready, copy it to another region.
3. Create volume from snapshot in new region.
4. Attach to EC2 and mount as above.

---

### **Experiment 4: Amazon Elastic File System (EFS)**

#### **Task 1: Create Security Groups**

1. Create SG for EC2 – allow SSH and NFS (2049).
2. Create SG for EFS – allow NFS from EC2 SG.

#### **Task 2: Launch EC2 and Mount EFS**

1. Launch EC2 in same VPC.
2. Create EFS, choose VPC and mount targets.
3. Install EFS utils:
   ```bash
   sudo apt install -y nfs-common
   sudo mkdir /efs
   sudo mount -t nfs4 -o nfsvers=4.1 <EFS-DNS>:/ /efs
   ```

#### **Task 3: Create Directory and File**

```bash
sudo mkdir /efs/data
sudo echo "YourRollNo" > /efs/data/rollno.txt
```

#### **Task 4: Launch Another EC2 and Mount EFS**

1. Launch new EC2 in same subnet/VPC.
2. Repeat mount steps.

#### **Task 5: Access File from Second EC2**

```bash
cat /efs/data/rollno.txt
```

---

### **Experiment 5: Amazon VPC**

#### **Tasks Overview**

1. Create custom VPC.
2. Create public and private subnets.
3. Launch EC2 in each subnet.
4. Create IGW, attach to VPC.
5. Create Route Table for public subnet, associate IGW.
6. SSH into Public EC2, then use it to SSH into Private EC2.

---

### **Experiment 6: VPC with NAT Gateway**

#### **Extra Steps after Exp 5**

1. Create Elastic IP for NAT.
2. Create NAT Gateway in Public Subnet.
3. Create Route Table for Private Subnet – route 0.0.0.0/0 to NAT.
4. Test Private EC2 access to internet:
   ```bash
   curl google.com
   ```

---

### **Experiment 7: S3 Static Website**

#### **Steps**

1. Create S3 bucket with unique name.
2. Upload HTML/images.
3. Set object permissions to public/read.
4. Enable **Static Website Hosting** in Properties.
5. Access via provided endpoint.

---

### **Experiment 8: Lambda with S3 Trigger and DynamoDB Update**

#### **Task 1: Create Lambda Function**

1. Go to Lambda > Create Function.
2. Use runtime (Node.js or Python).
3. Use code from `tinyurl.com/kmitdemocode`.
4. Set permissions for S3 and DynamoDB access.

#### **Task 2: Setup S3 Trigger**

1. Go to your bucket > Properties > Event Notification.
2. Add event to trigger Lambda on **Object Created**.

#### **Task 3: Update DynamoDB**

1. Create DynamoDB table with appropriate schema.
2. In Lambda, use SDK (boto3 for Python) to write data.
3. Test by uploading object to S3 and checking DB.

---

Let me know if you want this guide exported to PDF, or if you want lab automation scripts for any of these experiments!

