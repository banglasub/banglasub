# 🎬 BanglaSubTitles: The Ultimate Bangla Subtitle Library

[![Live Demo](https://img.shields.io/badge/Live-banglasub.vercel.app-60a5fa?style=for-the-badge&logo=vercel)](https://banglasub.vercel.app)
[![License: MIT](https://img.shields.io/badge/License-MIT-34d399?style=for-the-badge)](https://opensource.org/licenses/MIT)
[![Stack](https://img.shields.io/badge/Stack-Vanilla%20JS%20%2B%20Vercel-f7df1e?style=for-the-badge&logo=javascript)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)

**BanglaSubTitles** is a high-performance, community-driven web application designed to bridge the gap between international content and Bengali-speaking audiences. Built with a focus on speed, precision, and user experience, it serves as a sophisticated index for high-quality Bangla subtitles.

---

## 🚀 Key Features

### 🧠 Intelligent Content Processing
-   **Automated Heuristics**: Advanced regex-based parsing to detect **Movie vs. TV Series** types from complex filenames.
-   **Smart Normalization**: Automatically strips release tags (e.g., `1080p`, `BluRay`, `x265`, `DDP5.1`) to group multiple subtitle versions under a single, clean title entry.
-   **Dynamic Metadata Enrichment**: Real-time integration with **The Movie Database (TMDB)** to fetch posters, ratings, cast, and detailed overviews.

### 🔍 Advanced Search & Navigation
-   **Ultra-Fast Fuzzy Search**: Integrated with [uFuzzy](https://github.com/leeoniya/uFuzzy) for instantaneous, client-side searching across the entire library with support for typos and partial matches.
-   **Multi-Dimensional Filtering**: Filter content by **Year**, **Genre**, and **Type** (Movie/TV) simultaneously.
-   **Dual-Mode Interface**: Toggle seamlessly between a visually rich **Card Grid** and a high-density **List View**.

### ⚡ Performance & UX
-   **Non-Blocking Architecture**: Metadata pre-fetching and lazy-loading ensure the initial page load remains lightning-fast.
-   **Weekly Highlights**: An automated, touch-friendly slider showcasing the latest 20 subtitle additions.
-   **Direct-to-Download**: Secure backend proxying for subtitle downloads with one-click "Copy Link" functionality via the Clipboard API.

---

## 🛠️ Technical Architecture

### Frontend Stack
-   **Core**: Vanilla JavaScript (ES6+) — Zero heavy frameworks for maximum performance.
-   **Styling**: Modern CSS3 utilizing **CSS Variables**, **Flexbox**, and **CSS Grid** for a fully responsive, dark-themed UI.
-   **Search**: [uFuzzy](https://github.com/leeoniya/uFuzzy) (A tiny, efficient fuzzy search engine).
-   **Icons**: FontAwesome 6.5.0.

### Backend & API
-   **Runtime**: Vercel Serverless Functions (Node.js).
-   **Data Source**: Dynamic fetching from the GitHub repository tree via the GitHub REST API.
-   **Metadata**: TMDB API for rich media information.

---

## 📖 How to Use for Collection

### 1. Adding New Subtitles
The system is designed to be "Drop-and-Go". To add new subtitles to the library:
1.  Navigate to the folder corresponding to the movie/series release year (e.g., `/2024/`).
2.  Upload your `.srt` file. 
3.  **Naming Strategy**: For best results, use standard release names. The script will automatically parse them.
    -   *Example*: `Oppenheimer.2023.1080p.BluRay.Bangla.srt`
    -   *Example*: `The.Last.of.Us.S01E01.WEB-DL.Bangla.srt`

### 2. Local Setup & Deployment
To run this project locally or deploy your own instance:

1.  **Clone & Install**:
    ```bash
    git clone https://github.com/banglasub/banglasub.git
    cd banglasub
    npm install -g vercel
    ```
2.  **Environment Configuration**:
    Set the following secrets in your Vercel project settings:
    -   `GITHUB_REPO_OWNER`: Your GitHub username.
    -   `GITHUB_REPO_NAME`: The repository name.
    -   `GITHUB_TOKEN`: A GitHub PAT with `repo` permissions.
    -   `TMDB_API_KEY`: Your API key from [TMDB](https://www.themoviedb.org/settings/api).

3.  **Launch**:
    ```bash
    vercel dev # Local development
    vercel     # Deploy to production
    ```

---

## 📜 License & Attribution

This project is licensed under the **MIT License**.

-   **Metadata**: All movie/series information and artwork are provided by [TMDB](https://www.themoviedb.org/).
-   **Subtitles**: All subtitles are community-contributed and credited to their original creators.
-   **Development**: Maintained by **Bugsfree Studio**.

---
*Empowering the Bengali community through global cinema.*
