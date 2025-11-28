# Deploying to Cloudflare Pages

This guide outlines how to deploy the **AI enabled blue team operations for small to medium size businesses** website to Cloudflare Pages.

## Prerequisites

1.  A [Cloudflare account](https://dash.cloudflare.com/sign-up).
2.  The project source code pushed to a GitHub repository.

## Deployment Steps

1.  **Log in to Cloudflare Dashboard**: Go to the [Cloudflare Dashboard](https://dash.cloudflare.com/) and select **Workers & Pages**.
2.  **Create Application**: Click **Create application**.
3.  **Select Pages**: **Important!** Click the **Pages** tab (next to the default "Workers" tab). Do not use the "Workers" options.
4.  **Connect to Git**: Click **Connect to Git**.
5.  **Select Repository**: Choose the GitHub repository containing this project (`smbsec`).
6.  **Configure Build Settings**:
    *   **Project Name**: Enter a name for your project (e.g., `smbsec-book`).
    *   **Production Branch**: `main` (or your default branch).
    *   **Framework Preset**: Select **Hugo**.
    *   **Build Command**: `hugo`
    *   **Build Output Directory**: `public`
    *   **Root Directory**: `website` (This tells Cloudflare the site source is in the `website` folder).
5.  **Environment Variables (Optional)**:
    *   If you need a specific Hugo version, add an environment variable:
        *   **Variable Name**: `HUGO_VERSION`
        *   **Value**: `0.152.2` (or your desired version)
6.  **Deploy**: Click **Save and Deploy**.

## Verifying Deployment

Cloudflare Pages will build the site and provide a unique URL (e.g., `https://smbsec-book.pages.dev`).

*   **Success**: You should see the "AI enabled blue team operations..." home page.
*   **Navigation**: Click the menu links (Chapter 1, Chapter 2, etc.) to ensure routing works.

## Continuous Deployment

Every time you push changes to the `main` branch, Cloudflare Pages will automatically rebuild and deploy the site.
