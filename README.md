# ServerlessShare

A secure, end-to-end encrypted file sharing app built with vanilla JavaScript and AWS serverless infrastructure. Files are encrypted in the browser using AES-256-GCM before upload — your server never sees the raw file. The decryption key lives only in the share link, never on any server.

## Live Demo

🔗 [serverlessshare.vercel.app](https://serverlessshare.vercel.app)

## Features

- 🔒 **End-to-end encryption** — AES-256-GCM encryption runs entirely in the browser using the Web Crypto API
- 🔑 **Zero-knowledge** — the decryption key is embedded in the `#fragment` of the share link, which browsers never send to any server
- 📁 **Multiple file upload** — select or drag and drop multiple files at once, each gets its own encrypted share link
- 🚀 **Serverless** — no backend server, powered entirely by AWS Lambda + API Gateway + S3
- ⏱️ **Auto-expiry** — uploaded files are automatically deleted from S3 after 24 hours
- 🛡️ **File validation** — blocks dangerous file types using extension, MIME type, and magic bytes checks
- 📏 **Size limit** — enforced at both frontend and Lambda level (50 MB max per file)
- 🌌 **Animated starfield UI** — premium dark UI with animated starfield background

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML, CSS, JavaScript |
| Encryption | Web Crypto API (AES-256-GCM) |
| Hosting | Vercel |
| File Storage | AWS S3 |
| API | AWS API Gateway |
| Backend Logic | AWS Lambda (Node.js) |
| Auth | AWS IAM |

## How Encryption Works

1. User selects a file in the browser
2. Browser generates a random **AES-256 key** and **IV** using `crypto.subtle`
3. File is encrypted in memory — raw bytes never leave the device
4. Encrypted blob (`IV + ciphertext`) is uploaded to S3 via a presigned URL
5. Share link is built as `https://yoursite.com/#decrypt:fileId:filename:KEY`
6. The `#fragment` (containing the key) is **never sent to any server** by the browser
7. Recipient opens the link, browser reads the key from the fragment, fetches the encrypted blob, decrypts locally, and saves the original file

## Architecture

```
Browser                          AWS
───────                          ───
Select file
Encrypt (AES-256-GCM)
    │
    ├── POST /upload-url ──────► Lambda (generates presigned S3 URL)
    ├── PUT (encrypted blob) ──► S3 (stores ciphertext only)
    └── POST /save-file ───────► Lambda (saves metadata to DB)

Share link: https://site.com/#decrypt:fileId:name:KEY
                                        ▲
                              key never hits server

Recipient opens link
Browser reads #KEY
    ├── GET /download/fileId ──► Lambda → S3 (fetches ciphertext)
    └── Decrypt locally ──────► saves original file
```

## Security

- S3 stores only AES-256-GCM encrypted blobs — unreadable without the key
- Decryption key exists only in the share link `#fragment`
- Presigned upload URLs expire in 5 minutes
- Files auto-delete from S3 after 24 hours
- File type validated by extension + MIME type + magic bytes (first 8 bytes)
- All traffic over HTTPS only (enforced by S3 bucket policy)
- S3 bucket policy blocks non-`.enc` uploads and direct public access

## Project Structure

```
serverlessShare/
└── index.html    # entire frontend — UI, encryption, upload, decrypt
```

## AWS Setup

You need the following AWS resources:

- **S3 bucket** — stores encrypted files, lifecycle rule deletes after 1 day
- **Lambda function 1** — `upload-url` — generates presigned S3 PUT URLs
- **Lambda function 2** — `save-file` — saves file metadata
- **Lambda function 3** — `download` — serves files for decryption
- **API Gateway** — routes HTTP requests to Lambda functions

## Deployment

### Frontend (Vercel)
1. Push `index.html` to a GitHub repo
2. Import the repo on [vercel.com](https://vercel.com)
3. Deploy — no build step needed

### Backend (AWS)
1. Create an S3 bucket with the CORS config and lifecycle rule below
2. Deploy the three Lambda functions
3. Create an API Gateway and connect the routes

### S3 CORS Config
```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["PUT", "GET", "HEAD"],
    "AllowedOrigins": ["https://your-vercel-url.vercel.app"],
    "ExposeHeaders": ["ETag"],
    "MaxAgeSeconds": 3000
  }
]
```

## License

MIT
