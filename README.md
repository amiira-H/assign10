AMIRA HAMOUD-1014

---

##  1. Project Overview

- **Frontend**: React
- **Backend**: Express.js + Node.js
- **Database**: MongoDB Atlas
- **Media Storage**: AWS S3
- **Hosting**:  
  - **Frontend**: AWS S3 Static Website  
  - **Backend**: AWS EC2 Instance

---

## 🌐 2. Frontend S3 Bucket Configuration

### ✅ Goal

Host the React frontend on AWS S3 as a static website.

### 🔧 Steps

- Created S3 bucket: `amira-blogapp-frontend` in region `eu-north-1`.
- Enabled static website hosting.
- Uploaded `index.html` and `error.html`.
- Enabled public access and applied bucket policy.

### 🔒 Bucket Policy

```json
{

  "Version": "2012-10-17", 
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*", 
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::amira-blogapp-frontend/*"
    }
  ]
}
```
3. Configure S3 Bucket for Media Uploads
Goal: Set up a second S3 bucket to store media (e.g., images or files uploaded by users).

What we did:
- Created another S3 bucket named amira-blogapp-media to store user-uploaded files and media.
- Configured the bucket with the same public access settings and added CORS (Cross-Origin Resource Sharing) - configuration to allow all origins and methods for file uploads and access.

### CORS Configuration:

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
    "AllowedOrigins": ["*"],
    "ExposeHeaders": ["ETag"]
  }
]
```
### 4. IAM Configuration
Goal: Create an IAM user with permissions to access both frontend and media S3 buckets.

What we did:
- Created an IAM user with programmatic access.
- Attached a custom policy that allows the user to perform actions like uploading, getting, deleting, and listing objects for both amira-blogapp-frontend and amira-blogapp-media S3 buckets.

##### IAM Policy:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::amira-blogapp-media",
        "arn:aws:s3:::amira-blogapp-media/*",
        "arn:aws:s3:::amira-blogapp-frontend",
        "arn:aws:s3:::amira-blogapp-frontend/*"
      ]
    }
  ]
}
```

#### 5. EC2 Instance Setup
Goal:
 Host the backend (Express API) on an EC2 instance.

What we did:
- Launched an EC2 instance with Ubuntu 22.04 LTS in the eu-north-1 region.
- Configured EC2 security groups to allow inbound traffic on port 22 (for SSH access) and port 5000 (for backend API).
- Configured the EC2 instance with a startup script that installs necessary dependencies like Node.js, PM2 (for process management), and AWS CLI.

#### startup script
```
#!/bin/bash
apt update -y
apt install -y git curl unzip tar gcc g++ make unzip

su - ubuntu << 'EOF'   
export NVM_DIR="$HOME/.nvm"
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source "$NVM_DIR/nvm.sh"
nvm install --lts
nvm use --lts
npm install -g pm2
EOF
```

### MongoDB Shell
```
curl -L https://downloads.mongodb.com/compass/mongosh-2.1.1-linux-x64.tgz -o mongosh.tgz
tar -xvzf mongosh.tgz
mv mongosh-*/bin/mongosh /usr/local/bin/
chmod +x /usr/local/bin/mongosh
rm -rf mongosh*
```

#### AWS CLI
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
./aws/install
rm -rf aws awscliv2.zip
```

#### 6. Deploy Backend (Express API)

Goal: 
Set up and deploy the backend API to EC2.

What we did:
- Cloned the MERN stack blog application repo to the EC2 instance.
- Configured the environment variables for the backend in .env (e.g., PORT, MONGODB, AWS credentials).
- Used PM2 to manage the Express API process and ensure it continues running in the background even after server restarts.

#### PM2 Commands:
```
cd /home/ubuntu/blog-app/backend
npm install
pm2 start index.js --name "blog-backend"
pm2 startup
sudo pm2 startup systemd -u ubuntu
sudo env PATH=$PATH:/home/ubuntu/.nvm/versions/node/v22.15.0/bin /home/ubuntu/.nvm/versions/node/v22.15.0/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu
pm2 save
```
 
 #### 7. Deploy Frontend (React)
Goal:
 Build the React frontend and deploy it to S3.
What we did:
- Built the React frontend using pnpm run build.
- Uploaded the dist/ directory (built React files) to the amira-blogapp-frontend S3 bucket.

###### S3 Sync Command:
```
aws s3 sync dist/ s3://amira-blogapp-frontend/
```



#### 8. Validation
- Backend Server Running via PM2:
 Verified that the backend was running correctly using PM2.
- Frontend S3 Web Page:
   Verified that the frontend S3 bucket was accessible via its public URL, ensuring that it served the React application

#### 9. Deliverables
- Backend Server Running via PM2:
Screenshot showing the PM2 process for the backend.
- Frontend S3 Web Page:
Screenshot showing the public URL for the frontend S3 website.
- Successful Request from curl -I:
Output showing the 200 OK status, confirming that the frontend S3 static site is accessibl