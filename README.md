{\rtf1\ansi\ansicpg1252\cocoartf2822
\cocoatextscaling0\cocoaplatform0{\fonttbl\f0\fnil\fcharset0 Georgia;}
{\colortbl;\red255\green255\blue255;\red0\green0\blue0;}
{\*\expandedcolortbl;;\csgenericrgb\c0\c0\c0;}
\margl1440\margr1440\vieww11520\viewh8400\viewkind0
\deftab720
\pard\pardeftab720\sa210\pardirnatural\partightenfactor0

\f0\fs26 \cf2 # Assignment 4: Deploy Gitea in a Custom VPC with Multiple EC2 Instances\
\
**Course:** CSC 510 Cloud Computing  \
**Author:** Sandeep Shiraskar  \
\
---\
\
## Overview\
\
For this assignment, the following architecture was used:\
\
- **VPC:** `10.0.0.0/16`\
- **Public Subnet:** `10.0.1.0/24`  \
  - Contains nginx reverse proxy EC2 (only instance with public IP)\
- **Private Subnet:** `10.0.2.0/24`  \
  - Contains Gitea EC2 and FastAPI audit/event API EC2 (no public IPs)\
- **Internet Gateway (IGW)** attached to VPC\
- **NAT Gateway** in public subnet for outbound internet access\
- **Security Groups:**\
  - `sg-nginx-public`: Ports 80, 22 (SSH restricted to my IP)\
  - `sg-backend-private`: Ports 22, 3000, 5000 (only from nginx SG)\
\
---\
\
## PART 1: Create VPC and Subnets\
\
### 1.1 Create VPC\
\
A custom VPC (`assignment4-vpc`) was created with CIDR block `10.0.0.0/16`.\
\
![Screenshot A1](images/Screenshot A1.png)\
\
---\
\
### 1.2 Create Subnets\
\
The VPC contains:\
\
- Public subnet: `10.0.1.0/24`\
- Private subnet: `10.0.2.0/24`\
\
Both are in the same availability zone.\
\
![Screenshot A2](images/Screenshot A2.png)\
\
---\
\
## PART 2: Internet Gateway and NAT Gateway\
\
### 2.1 Internet Gateway\
\
Created and attached `assignment4-igw` to the VPC.\
\
![Screenshot A3](images/Screenshot A3.png)\
\
---\
\
### 2.2 NAT Gateway\
\
A NAT Gateway (`assignment4-nat`) was created in the public subnet to allow private instances outbound internet access.\
\
> \uc0\u9888 \u65039  Note: NAT Gateways incur hourly and data-processing charges. Delete it after testing to avoid unnecessary costs.\
\
![Screenshot A4](images/Screenshot A4.png)\
\
---\
\
## PART 3: Route Tables\
\
- **Public Route Table:** `0.0.0.0/0 \uc0\u8594  IGW`\
- **Private Route Table:** `0.0.0.0/0 \uc0\u8594  NAT Gateway`\
\
![Screenshot A5](images/Screenshot A5.png)\
\
---\
\
## PART 4: Security Groups\
\
Two security groups were created:\
\
- **sg-nginx-public**\
  - Ports: 80 (HTTP), 22 (SSH from my IP)\
\
- **sg-backend-private**\
  - Ports: 22, 3000, 5000 (only from nginx SG)\
\
![Screenshot A61](images/Screenshot A61.png)  \
![Screenshot A62](images/Screenshot A62.png)\
\
---\
\
## PART 5: Launch EC2 Instances\
\
Three EC2 instances were launched:\
\
- **nginx (public subnet)** \uc0\u8594  has public IPv4\
- **Gitea (private subnet)** \uc0\u8594  no public IP\
- **audit-api (private subnet)** \uc0\u8594  no public IP\
\
![Screenshot A71](images/Screenshot A71.png)  \
![Screenshot A72](images/Screenshot A72.png)  \
![Screenshot A73](images/Screenshot A73.png)\
\
---\
\
## PART 6: SSH Access (Local PC \uc0\u8594  nginx \u8594  backends)\
\
### 6.1 SSH to nginx\
\
Connected to nginx instance from local machine.\
\
![Screenshot A8](images/Screenshot A8.png)\
\
---\
\
### 6.2 SSH to Private Instances via Jump Host\
\
Used nginx as a jump host to access private instances.\
\
![Screenshot A9](images/Screenshot A9.png)\
\
---\
\
## PART 7: Backend Setup\
\
### 7.1 nginx Setup\
\
Installed nginx and verified access via public IP.\
\
![Screenshot B1](images/Screenshot B1.png)\
\
---\
\
### 7.2 Gitea Backend (Docker)\
\
- Installed Docker\
- Ran Gitea container on port `3000`\
\
![Screenshot B2](images/Screenshot B2.png)  \
![Screenshot B3](images/Screenshot B3.png)\
\
---\
\
### 7.3 Audit/Event API (FastAPI)\
\
- Runs on port `5000`\
- Endpoints:\
  - `GET /api/health`\
  - `GET /api/events`\
  - `POST /api/events`\
- Logs stored in `api.log`\
\
![Screenshot B4](images/Screenshot B4.png)\
\
---\
\
## PART 8: Configure nginx Reverse Proxy\
\
Configured nginx for path-based routing:\
\
- `/` \uc0\u8594  Gitea (`:3000`)\
- `/api/` \uc0\u8594  FastAPI (`:5000`)\
\
![Screenshot B5](images/Screenshot B5.png)\
\
---\
\
## PART 9: Validation\
\
### 9.1 Internal Connectivity\
\
nginx successfully connects to both backends.\
\
![Screenshot C1](images/Screenshot C1.png)  \
![Screenshot C1a](images/Screenshot C1a.png)\
\
---\
\
### 9.2 External Access via Proxy\
\
- Gitea accessible via browser\
- API accessible via curl\
\
![Screenshot C2](images/Screenshot C2.png)  \
![Screenshot C3](images/Screenshot C3.png)\
\
---\
\
### 9.3 Logging Verification\
\
API logs all requests in `api.log`.\
\
![Screenshot C4](images/Screenshot C4.png)\
\
---\
\
### 9.4 Backend Isolation\
\
- No public IPs on backend instances\
- Direct access from internet fails\
\
![Screenshot C5](images/Screenshot C5.png)\
\
---\
\
### 9.5 NAT Gateway Validation\
\
Private instances can access internet (e.g., `apt update`, `curl google.com`).\
\
![Screenshot C6](images/Screenshot C6.png)\
\
---\
\
## PART 10: Architecture Diagram\
\
![Architecture](images/Architecture.png)\
\
---\
\
## Summary\
\
The architecture consists of:\
\
- Custom VPC (`10.0.0.0/16`)\
- Public subnet (`10.0.1.0/24`) with:\
  - nginx reverse proxy\
  - NAT Gateway\
- Private subnet (`10.0.2.0/24`) with:\
  - Gitea instance\
  - FastAPI audit/event API\
\
### Key Design Points\
\
- Public route table \uc0\u8594  Internet Gateway (external access)\
- Private route table \uc0\u8594  NAT Gateway (secure outbound access)\
- Security groups enforce least privilege:\
  - nginx: HTTP (80) open, SSH restricted\
  - backends: only accessible via nginx\
\
This setup ensures:\
- Secure isolation of backend services\
- Controlled public access via reverse proxy\
- Cost-aware NAT usage}