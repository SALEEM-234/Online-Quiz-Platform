# Online Quiz Platform

A Django web app where users sign up, pick a quiz category, answer timed multiple-choice questions one at a time, and see their score and rank on a leaderboard.

> **Quick start:** Install dependencies, run migrations, create a superuser to add categories and questions, then visit `http://127.0.0.1:8000` to start quizzing.

---

## Table of Contents

- [What it can do](#what-it-can-do)
- [How it works (in plain words)](#how-it-works-in-plain-words)
- [What's inside the project](#whats-inside-the-project)
- [Before you start (prerequisites)](#before-you-start-prerequisites)
- [Step 1 — Clone the repository](#step-1--clone-the-repository)
- [Step 2 — Create and activate a virtual environment](#step-2--create-and-activate-a-virtual-environment)
- [Step 3 — Install dependencies](#step-3--install-dependencies)
- [Step 4 — Configure the environment (.env)](#step-4--configure-the-environment-env)
- [Step 5 — Apply database migrations](#step-5--apply-database-migrations)
- [Step 6 — Create a superuser and add quiz content](#step-6--create-a-superuser-and-add-quiz-content)
- [Step 7 — Run the development server](#step-7--run-the-development-server)
- [Using the app](#using-the-app)
- [Troubleshooting](#troubleshooting)
- [Tech stack](#tech-stack)

---

## What it can do

- User registration and login (no email required — username and password only)
- Browse quiz categories, each with a cover image, description, total marks, total questions, and a time limit
- Start a quiz for any category; the app tracks which questions you have already answered so you can navigate pages safely
- Answer multiple-choice questions one at a time (paginated — one question per page)
- Answers are shuffled randomly each time a question is displayed
- Submit an answer via AJAX — instant feedback without a page reload
- Marks are calculated automatically based on per-question point values
- A countdown timer (set per category) auto-submits the quiz when time runs out (via a cron/scheduled task)
- View your current score at any point during the quiz
- Leaderboard showing the top 20 scores across all completed quizzes
- Revisit attended questions and see which answer you gave and whether it was correct
- Full sign-out support

---

## How it works (in plain words)

```
User signs up / signs in
     |
     v
Home page  -->  list of quiz categories
     |
     v
Click a category  -->  Quiz object created (or existing one retrieved)
     |
     v
Quiz page  -->  one question per page (paginator)
     |          Answers shuffled randomly each render
     |
     v
Click an answer  -->  AJAX POST to /checkAnswer/<answer_uid>/<createObj>
     |                  - marks answer, adds to GivenQuizQuestions
     |                  - returns { is_correct, marks } JSON
     |
     v
Navigate to next question  -->  previous answer state reloaded from DB
     |
     v
All questions done / timer expires  -->  Quiz status = "yes", marks finalised
     |
     v
Leaderboard  -->  top 20 Quiz objects with status="yes", ordered by marks desc
```

---

## What's inside the project

```
Online-Quiz-Platform/
├── manage.py                   # Django entry point
├── requirements.txt            # Python dependencies
├── db.sqlite3                  # SQLite database (created after migrations)
│
├── quiz/                       # Django project config
│   ├── settings.py             # App settings, media files, installed apps
│   ├── urls.py                 # Root URL dispatcher (includes home.urls)
│   ├── wsgi.py                 # WSGI entry point
│   └── asgi.py                 # ASGI entry point
│
├── home/                       # Main quiz app
│   ├── models.py               # Category, Question, Answer, GivenQuizQuestions, Quiz
│   ├── views.py                # index, check_answer (AJAX), quiz, leaderboard,
│   │                           #   sign_up, sign_in, sign_out, loadAttendedQuestionData
│   ├── urls.py                 # URL patterns for all quiz routes
│   ├── cron.py                 # Scheduled task for auto-submitting timed-out quizzes
│   ├── admin.py                # Admin registration for all models
│   └── migrations/             # Database migration history
│
├── templates/                  # HTML templates
│   └── home/
│       ├── index.html          # Category listing / home page
│       ├── quiz.html           # Single-question quiz view with timer
│       ├── leaderboard.html    # Top 20 scores table
│       ├── signup.html         # Registration form
│       └── signin.html         # Login form
│
├── static/                     # CSS, JS (answer-checking logic, countdown timer)
└── media/                      # Uploaded category images
```

---

## Before you start (prerequisites)

1. Python 3.10 or newer installed
2. `pip` available (bundled with Python)
3. Git installed (to clone the repo)
4. A terminal / command prompt

---

## Step 1 — Clone the repository

**Windows (PowerShell) and macOS/Linux:**
```bash
git clone <repository-url>
cd Online-Quiz-Platform
```

---

## Step 2 — Create and activate a virtual environment

**Windows:**
```powershell
python -m venv venv
venv\Scripts\activate
```

**macOS/Linux:**
```bash
python3 -m venv venv
source venv/bin/activate
```

---

## Step 3 — Install dependencies

**Windows:**
```powershell
pip install -r requirements.txt
```

**macOS/Linux:**
```bash
pip3 install -r requirements.txt
```

---

## Step 4 — Configure the environment (.env)

This project uses `python-decouple` to read a secret key from a `.env` file. You must create this file before running the app — it will crash without it.

> **Note:** The variable name in this project is `KEY` (not `SECRET_KEY`).

### Generate a key

Run one of these commands and copy the output:

**Option A — using Django**
```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

**Option B — using Python's secrets module**
```bash
python -c "import secrets; print(secrets.token_urlsafe(50))"
```

### Create the .env file

Create a file named `.env` in the project root (same folder as `manage.py`) and add:

```env
KEY=paste-your-generated-key-here
```

That is the only required variable. The project uses SQLite by default so no database configuration is needed.

> **Important:** Never commit your `.env` file to version control.

---

## Step 5 — Apply database migrations

**Windows:**
```powershell
python manage.py makemigrations
python manage.py migrate
```

**macOS/Linux:**
```bash
python3 manage.py makemigrations
python3 manage.py migrate
```

---

## Step 6 — Create a superuser and add quiz content

The app needs categories and questions before users can take any quiz. Add them through the Django admin panel.

**Windows:**
```powershell
python manage.py createsuperuser
```

**macOS/Linux:**
```bash
python3 manage.py createsuperuser
```

After starting the server (Step 6), go to `http://127.0.0.1:8000/admin/` and:

1. Add a **Category** — give it a name, description, total time (in minutes), and optionally upload an image.
2. Add **Questions** linked to that category, with a mark value per question.
3. Add **Answers** linked to each question — mark exactly one as correct (`is_correct = True`).

---

## Step 7 — Run the development server

**Windows:**
```powershell
python manage.py runserver
```

**macOS/Linux:**
```bash
python3 manage.py runserver
```

Open your browser and go to: `http://127.0.0.1:8000`

---

## Using the app

1. **Sign up** — click "Sign Up", enter a username and password.
2. **Sign in** — use those credentials to log in.
3. **Pick a category** — the home page lists all quiz categories. Click one to start (or resume) a quiz.
4. **Answer questions** — each page shows one question with shuffled answer choices. Click an option to record your answer; you get instant feedback on whether it was correct.
5. **Navigate pages** — use the pagination controls to move between questions. Already-answered questions show your previous selection highlighted.
6. **Track your score** — your current marks are shown on the quiz page and update in real time.
7. **Finish** — once all questions are answered the quiz is marked complete and your score is saved.
8. **Leaderboard** — click "Leaderboard" in the nav to see the top 20 scores across all users.

---

## Troubleshooting

**`decouple.UndefinedValueError: KEY not found`**
The `.env` file is missing or doesn't have the `KEY` variable. Create `.env` in the project root (same folder as `manage.py`) and add:
```env
KEY=your-generated-secret-key
```
Generate a key with: `python -c "import secrets; print(secrets.token_urlsafe(50))"`

**Home page loads but no categories appear**
No categories have been added yet. Log in to `/admin/` as a superuser and create at least one Category with Questions and Answers.

**`Something Went Wrong` error on the quiz page**
This usually means no Quiz object was created before navigating directly to `/quiz/`. Always start a quiz by clicking a category from the home page, which creates the Quiz record first.

**Answers are not saving (AJAX returns 404)**
Make sure the Answer UUID in the URL is correct. If you edited answer records in the admin after starting a quiz session, the old UIDs may no longer match.

**Timer does not auto-submit**
The auto-submit feature relies on `home/cron.py` being executed by a task scheduler. In local development the timer only counts down on screen; you need to finish manually or set up a cron job.

**`Pillow` install fails on Windows**
Run `pip install Pillow --upgrade`. If that fails, install the Microsoft C++ Build Tools from the Visual Studio installer.

**Cannot log in after sign-up**
The registration form uses `password1` — make sure you filled in the password field (labelled "password" on the form). If you used the wrong field the account may have been created with a blank password; delete it from `/admin/` and sign up again.

---

## Tech stack

| Technology | Version | Purpose |
|---|---|---|
| Python | 3.10+ | Backend language |
| Django | — | Web framework — ORM, auth, views, paginator |
| Pillow | — | Image upload and processing for category images |
| SQLite | Built-in | Development database |
| JavaScript (vanilla) | — | AJAX answer submission, countdown timer |
| Gunicorn | — | Production WSGI server (optional) |
