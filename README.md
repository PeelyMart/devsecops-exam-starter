# DevSecOps Exam Starter

## Setup Instructions

### Build the Docker Image

```bash
docker build -t devsec-api .
```

### Run the Docker Container

```bash
docker run -p 3000:3000 devsec-api
```

### Verify the Application

Open a browser or use curl:

```bash
curl http://localhost:3000/health
```

Expected response:

```json
{
  "status": "ok"
}
```

---

# Architectural Explanation

## Docker Base Image Choice

This project uses the `node:22-alpine` base image.

I selected the Alpine variant because it is significantly smaller than the standard Node.js image while still providing everything necessary to run the application. One of the common principles when building Docker containers is to keep images as small as possible. Smaller images result in faster downloads, faster deployments, reduced storage usage, and a smaller attack surface.

Using a specific Node.js version (`node:22-alpine`) instead of a generic tag such as `node:latest` also provides consistency and reproducibility. Future changes to the latest image will not unexpectedly affect the application.

Additionally, the container is configured to run using the non-root `node` user rather than the root account, following container security best practices.

## Security Scanner Choices

I implemented two automated security checks within the CI pipeline.

### Dependency Scanning (npm audit)

`npm audit` is used to scan project dependencies for known security vulnerabilities.

This scanner was chosen because dependency vulnerabilities are one of the most common security risks in modern software projects. Applications often rely on many third-party packages, and a vulnerability in a dependency can introduce security issues even when the application's own source code is secure.

The scan runs automatically during GitHub Actions workflows and helps identify vulnerable packages before they are merged into the main branch.

### Secret Scanning (TruffleHog)

TruffleHog is used to detect accidentally committed secrets such as API keys, access tokens, passwords, and credentials.

Although `.gitignore` and `.dockerignore` are configured to exclude `.env` files, developers may still accidentally hardcode secrets directly into source code. TruffleHog scans repository contents and commit history to identify these cases and prevent sensitive information from being introduced into the codebase.

This provides an additional layer of protection beyond simply ignoring environment files.

---

# Vulnerability Demonstration

## Dependency Vulnerability Detection

A deliberately vulnerable dependency was introduced into the project for demonstration purposes.

When the GitHub Actions workflow executed:

```bash
npm audit --audit-level=high
```

the vulnerability was detected and the security stage of the pipeline failed.

### Screenshot: Dependency Scan Failure
![npm-audit](screnshots/trufflehog-detection.jpg)


---

## Secret Detection Demonstration

A test credential was intentionally added to demonstrate secret scanning.

TruffleHog successfully detected the secret during scanning and flagged it as a security finding.

### Screenshot: TruffleHog Detection

![trufflehog-detection](screnshots/npm-audit.jpg)

---

## Vulnerability Remediation

During development, a moderate severity vulnerability involving the `qs` package was identified through `npm audit`.

The issue affected the dependency chain used by both Express and Body Parser. After identifying the vulnerability, dependency updates were performed to resolve the issue and reduce the project's security risk.

---

# Challenges Faced

The biggest challenge during this project was learning Docker, as I had not previously worked extensively with containerization technologies.

Initially, understanding Docker images, containers, build contexts, and Dockerfiles was unfamiliar. However, after studying official documentation and community resources, the concepts became much easier to understand. My previous experience with Linux systems and scripting helped significantly because many Docker workflows feel similar to working in a lightweight Linux environment.

Additionally, experience configuring development tools such as Neovim exposed me to configuration files, automation, and command-line workflows, which made understanding Docker concepts much more approachable.

Overall, the project provided a practical introduction to Docker, CI/CD pipelines, and DevSecOps practices while demonstrating how security can be integrated directly into the software development lifecycle.
