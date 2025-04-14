# 🐾 Sona Vault 🐺

## Your Personal Furry Art Repository

Sona Vault is a self-hosted platform where furries can securely store, organize, and showcase their fursona artwork in one convenient location.

![Sona Vault Banner](https://placeholder.com/sona-vault-banner)

## 🌟 Features

### 🖼️ Art Management

- **Secure Upload System**: Keep your precious artwork safe in your personal vault
- **Content Categorization**: Separate SFW and NSFW artwork with NSFW hidden by default
- **Custom Galleries**: Organize your sonas by species, artist, or any tag you choose

### 🔗 Artist Attribution

- **Creator Credits**: Link each artwork to its original artist
- **Artist Profiles**: Create and maintain artist profiles with their handles
- **Artist Favorites**: Mark your favorite artists for easy reference

### 📊 Posting Tracker

- **Platform Tracking**: Simply record which platforms you've shared each piece on
- **Post History**: See at a glance where you've already posted your art
- **Repost Prevention**: Avoid accidentally reposting the same art on the same platform

### 🔒 Privacy & Control

- **Self-Hosted Security**: Your art stays on your server, under your control
- **Customizable Sharing**: Choose what to share and with whom
- **Content Filtering**: Robust NSFW controls to keep sensitive content private
- **Backup System**: Never lose your precious collection

## 🚀 Getting Started

```bash
# Clone the repository
git clone https://github.com/yourusername/sona-vault.git

# Navigate to the project directory
cd sona-vault

# Start the development database
docker-compose up -d

# Install dependencies
npm install

# Configure your environment
cp .env.example .env
nano .env

# Start your development server
npm run dev

## 💻 Tech Stack

- **Framework**: [T3 Stack](https://create.t3.gg/)
  - [Next.js](https://nextjs.org)
  - [tRPC](https://trpc.io)
  - [Tailwind CSS](https://tailwindcss.com)
  - [TypeScript](https://typescriptlang.org)
  - [Prisma](https://prisma.io)
- **Database**: PostgreSQL (via Docker for development)
- **Authentication**: NextAuth.js
- **Storage**: Local or cloud options (AWS S3, Google Cloud Storage)

## 🐳 Docker Development

The project includes a `docker-compose.yaml` file for easy database setup.

## 🛠️ Self-Hosting Requirements

- Node.js LTS
- 1GB RAM minimum (2GB recommended)
- 10GB storage (varies based on your collection size)
- Modern web browser
- Docker (for easy deployment)
```

## 🤝 Contributing

We welcome contributions from the furry community! Whether you're a developer, designer, or just have great ideas, check out our [contribution guidelines](CONTRIBUTING.md).

## 📜 License

Sona Vault is released under the MIT License - see the [LICENSE](LICENSE) file for details.

---

### 🐾 "Keep your sonas safe, organized, and celebrated!"
