
# APKForge v1 — বাংলা সেটআপ গাইড

APKForge হলো একটি starter project: মোবাইল ব্রাউজার থেকে HTML project ZIP আপলোড করে GitHub Actions-এ Android WebView APK build করার ব্যবস্থা।

## গুরুত্বপূর্ণ সীমাবদ্ধতা
- এই ZIP একটি **starter MVP**, নিজে থেকেই কোনো সার্ভারে deploy হয়ে যাবে না।
- Cloud build চালাতে একটি GitHub repository, GitHub Actions workflow এবং একটি publicly reachable backend server লাগবে।
- এই সংস্করণে GitHub Actions `debug` APK তৈরি করে। ব্যক্তিগত পরীক্ষা/ইনস্টলের জন্য এটি উপযোগী; Play Store release-এর জন্য আলাদা release signing ও testing দরকার।
- Camera permission manifest-এ যোগ করা যায়, কিন্তু ওয়েব পেজে camera ব্যবহার করতে HTTPS origin এবং Android WebView permission handling দুটোই ঠিক থাকতে হবে। এই starter-এ camera permission অনুরোধ অনুমোদনের বেসিক handling আছে; সব camera library/device-এ কাজের নিশ্চয়তা নেই।
- কেবল নিজের বা ব্যবহারের অনুমতি থাকা HTML/project আপলোড করবে।

## ZIP-এ কী আছে
- `web/` — মোবাইল-ফ্রেন্ডলি APKForge dashboard
- `server/` — Node.js backend; upload, GitHub workflow dispatch, status ও artifact download
- `android-template/` — Android WebView wrapper template
- `.github/workflows/build-apk.yml` — GitHub Actions cloud build workflow
- `sample-project/` — পরীক্ষার জন্য sample HTML

## সেটআপ — Android ফোন দিয়েই

### 1. GitHub repository তৈরি
1. GitHub-এ login করো।
2. নতুন repository তৈরি করো, যেমন `apkforge-cloud-builder`।
3. এই ZIP extract করে এর ভেতরের সব ফাইল repository-তে upload করো (root folder-সহ নয়; ফাইলগুলো repository root-এ থাকবে)।
4. GitHub repository-র **Settings → Actions → General** এ গিয়ে Actions অনুমোদিত আছে কিনা দেখো।

### 2. Backend deploy
Node.js 20 সমর্থন করে এমন hosting-এ `server/` deploy করো (যেমন Render Web Service)।
- Build command: `npm install`
- Start command: `npm start`
- Root directory: `server`

Environment variables:
- `GITHUB_TOKEN` — fine-grained token; repository Contents: Read and write, Actions: Read and write, Metadata: Read-only
- `GITHUB_OWNER` — GitHub username
- `GITHUB_REPO` — repository name
- `GITHUB_WORKFLOW` — `build-apk.yml`
- `GITHUB_BRANCH` — `main`
- `PUBLIC_BASE_URL` — backend-এর public HTTPS URL, শেষে slash ছাড়া
- `APP_API_KEY` — নিজে বানানো দীর্ঘ random secret
- `PORT` — hosting সাধারণত নিজে দেয়; না দিলে `3000`

**Token কখনো frontend-এ বা কারও সঙ্গে share করবে না।** Server environment variables-এ রাখবে। Token-এর permission শুধু এই project repository-তে সীমাবদ্ধ রাখবে।

### 3. GitHub Actions workflow
`.github/workflows/build-apk.yml` repository-তে আছে কিনা দেখো। প্রথম build-এর আগে GitHub-এ Actions enable থাকতে হবে।

### 4. Website connect
`web/index.html` খুলে `API_BASE`-এ backend-এর HTTPS URL বসাও।
শুধু নিজের ব্যক্তিগত private website দিয়ে পরীক্ষা করার MVP পদ্ধতি: `web/index.html`-এর `API_KEY`-এ `APP_API_KEY`-এর মান বসালে backend request অনুমোদন করবে। এতে key browser source-এ দেখা যাবে, তাই এই পদ্ধতি কখনো public website-এ ব্যবহার করবে না। Public service-এর জন্য server-side login/session যোগ করতে হবে।

### Production security (খুব জরুরি)
এই starter backend-এ API key header দিয়ে protected endpoints আছে। Browser-এ key রাখলে সেটি দেখা সম্ভব। তাই:
- নিজের ব্যক্তিগত ব্যবহারে private website এবং নিজের access ছাড়া publish কোরো না।
- Public service বানানোর আগে proper user authentication, per-user rate limit, upload quota, CSRF/CORS policy, abuse protection, and isolated build jobs যোগ করো।
- সবচেয়ে ভালো: frontend login + server-side session ব্যবহার করে key browser-এ না পাঠানো।

## HTML Project ZIP format
ZIP-এর root-এ `index.html` থাকা ভালো:
```
my-app.zip
  index.html
  style.css
  script.js
  assets/
```
অথবা ZIP-এর মধ্যে একটি project folder থাকলেও backend প্রথমে root `index.html` খুঁজবে না; তাই `index.html` সরাসরি ZIP root-এ রাখো।
External CDN/API-এর উপর নির্ভরশীল পেজের জন্য ইন্টারনেট permission দরকার। Offline ব্যবহারের জন্য সব asset ZIP-এ রাখো।

## Build limitations
- Upload limit: 10 MB.
- App package ID অবশ্যই `com.example.myapp`-এর মতো বৈধ reverse-domain format হতে হবে।
- App name-এ সাধারণ অক্ষর ব্যবহার করো।
- Build সফল হলে GitHub Actions artifact `apkforge-debug-apk` তৈরি হয়; backend সেটি থেকে `.apk` ফাইল বের করে সরাসরি download দেয়.
- APK download না হলে GitHub repository → Actions → সর্বশেষ run খুলে error log দেখো।

## নিরাপত্তা
APKForge অজানা HTML-কে trusted ধরে না। আপলোড করা ওয়েব কোড APK-তে চলবে; অপরিচিত কোড চালানোর আগে দেখে নাও। Password, API key বা private data HTML-এ রেখো না।
