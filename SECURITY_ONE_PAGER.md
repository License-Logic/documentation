# License Logic Security Overview

**Protecting Your Data with Enterprise-Grade Security**

---

## What We Collect & Why

### Desktop Agents (Windows & Mac)
| Data Type | Purpose | Scope | 
|-----------|---------|-------|
| **Application Usage** |  Track software license utilization | Process name, duration, start/stop times | 
| **Browser History** | Identify SaaS applications in use | URL, page title, timestamp |
| **User Identity** | Associate usage with employees | Email (via Entra/AD lookup or manual) |

**What We DON'T Collect:**
- Keystrokes or screen captures
- File contents or document data
- Personal browsing or non-work accounts
- Audio, video, or camera data

### Browser Extension (Chrome)
- Browser history for SaaS detection
- Enterprise email for user identification
- Read-only access - no content modification

### Identity Provider Integrations

**Microsoft Entra ID:**
- Employee directory (name, email, title, department)
- Device registry (device name, OS, compliance status)
- Read-only access via Microsoft Graph API
- Scopes: `User.Read.All`, `Device.Read.All`, `Directory.Read.All`

**Google Workspace:**
- Employee directory (same as above)
- Chrome OS device mappings
- Application sign-in logs
- Read-only access via Admin SDK
- Scopes: `admin.directory.user.readonly`, `admin.directory.device.chromeos.readonly`

---

## Data Flow Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        CUSTOMER ENVIRONMENT                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────────┐  │
│  │ Windows  │  │   Mac    │  │  Chrome  │  │  Entra/Google  │  │
│  │  Agent   │  │  Agent   │  │Extension │  │  OAuth Flow    │  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └───────┬────────┘  │
│       │             │             │                 │           │
│       └─────────────┴─────────────┴─────────────────┘           │
│                            │                                     │
│                    ┌───────▼───────┐                            │
│                    │   ENCRYPTED   │                            │
│                    │   UPLOAD      │                            │
│                    │  (TLS 1.2+)   │                            │
│                    └───────┬───────┘                            │
└────────────────────────────┼────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│                    LICENSE LOGIC CLOUD (AWS)                     │
│                                                                  │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────────┐    │
│  │   S3 (Data)  │   │  RDS (DB)    │   │   Lambda (ETL)   │    │
│  │  AES-256     │   │  Encrypted   │   │   Processing     │    │
│  │  Encrypted   │   │  at Rest     │   │   (No PII Logs)  │    │
│  └──────────────┘   └──────────────┘   └──────────────────┘    │
└──────────────────────────────────────────────────────────────────┘
```

---

## Encryption & Security COntrols

### Data Encryption
| State | Method | Standard |
|-------|--------|----------|
| **In Transit** | TLS 1.2+ | HTTPS only, no plaintext |
| **At Rest (Logs)** | AES-256-GCM | Per-file unique keys |
| **At Rest (Database)** | AWS RDS Encryption | AES-256 |
| **At Rest (S3)** | Server-Side Encryption | SSE-S3/SSE-KMS |
| **OAuth Tokens** | AES-256-GCM | Encrypted before DB storage |

### Agent Security
- **Hybrid encryption**: AES-256 for data, RSA-2048 for key exchange
- **Unique Keys**: Fresh AES key/IV generated per log file
- **Minimal Permissions**: Agents run with least-privilege access
- **No Remote Access**: Agents upload only - no inbound connections


### Access Controls
- Role-based access control (RBAC)
- Multi-factor authentication supported
- API authentication via OAuth 2.0
- Audit logging for all admin actions

---

## Infrastructure Security

| Component | Security Measure |
|-----------|------------------|
| **Hosting** | AWS (SOC 2 Type II certified) |
| **Network** | VPC isolation, private subnets | 
| **API Gateway** | Rate limiting (50 req/sec), WAF protection |
| **Database** | Private subnet, SSL required, encrypted backups |
| **Monitoring** | CloudWatch logging, anomaly detection
| **Backups** | Daily automated backups, 30-day retention |

---

## Data Handling & Retention

### Retention Policy
- **Usage Logs**: 12 months (configurable)
- **Employee Directory**: Synced daily, removed on offboarding
- **OAuth Tokens**: Encrypted, refreshed automatically

### Data Deletion
- Customer data deleted within 30 days of contract termination
- On-demand deletion available upon request
- Cryptographic erasure for encrypted data

### Subprocessors
| Service | Purpose | Data Processed |
|---------|---------|----------------|
| AWS | Cloud Infrastructure | User email, name |
| Neon | Database Hosting | Application data |

---

## Compliance & Privacy

### Privacy by Design
- Minimal data collection - only what's needed
- No employee productivity monitoring
- Data used exclusively for license optimization
- Customer-owned data - exportable on request

### Regulatory Alignment
- GDPR-ready data handling practices
- CCPA-compliance privacy notices
- Standard DPA available upon request

---

## Integration Security

### OAuth Consnet Flow
1. Admin initiates connection from License Logic
2. Redirect to Microsoft/Google for authentication
3. Admin grants read-only permissions
4. Tokens encrypted and stored securely
5. Daily sync via secure API calls

### Least Privilege Principle
- **Read-only access** to identity providers
- No write/modify permissions requested
- No access to email content, files, or calendars
- Directory and device info only

---

## Incident Response

- 24-hour breach notification commitment
- Documented incident response procedures
- Regular security assessments
- Vulnerability management program

---

## Contact

**Security Questions**: security@licenselogic.co
**Data Protection**: privacy@licenselogic.co
**Support**: support@licenselogic.co

---

*Last Updated: December 2025*
*Version: 1.0*

