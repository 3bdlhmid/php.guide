# Task: Build a Simple Photo Gallery (PHP + OOP, No Full MVC)

  

**Assigned to:** [Friend's name]

**Level:** Beginner (knows PHP basics + OOP fundamentals)

**Estimated time:** 2–3 weeks

**Deliverable:** A working web app + short README

  

---

  

## 📌 Project Brief

  

You are building a **Photo Gallery** where a logged-in user can upload photos, view them in a grid, see a single photo page, and delete their own photos.

  

You will use **plain PHP with OOP classes**, some **HTML views**, and **MySQL**. No framework, no full MVC — just enough structure to learn how to separate logic from presentation.

  

The goal is **not** to build a pretty app. The goal is to write **clean, secure, well-organized PHP**.

  

---

  

## ✅ Functional Requirements (Must Have)

  

### 1. Authentication

- **Register** with email + password

- **Login** with email + password

- **Logout**

- Only logged-in users can upload, view their gallery, or delete

- Guests who open `/gallery.php` must be redirected to login

  

### 2. Photo Upload

- Form with: **image file** (required), **title** (required, max 100 chars), **description** (optional, max 500 chars)

- Accept only: **JPG, JPEG, PNG, GIF**

- Max file size: **5 MB**

- Uploaded file must be renamed to a unique name (e.g., `uniqid() . '.' . $ext`)

- Store metadata in DB: `user_id`, `filename`, `title`, `description`, `mime_type`, `size`, `created_at`

- Save files to `/uploads/` folder (outside of PHP execution)

  

### 3. Gallery Page

- Show all photos belonging to the **logged-in user only**

- Display in a **grid**: thumbnail + title

- Ordered by newest first

- If no photos: show a friendly empty message

  

### 4. Single Photo Page

- URL like `photo.php?id=5`

- Show: full image, title, description, upload date

- If the photo does **not** belong to the current user → show **404** (do not reveal it exists)

  

### 5. Delete Photo

- Delete button on the single photo page (only owner can see it)

- Must delete **both** the DB row and the file on disk

- Ask for confirmation (JS `confirm()` is fine)

- **Use POST**, not GET, for deletion

  

---

  

## 🧱 Technical Requirements

  

### Language / Stack

- **PHP 8.1+**

- **MySQL** via **PDO with prepared statements only**

- **Composer** for autoloading (`psr-4`)

- **HTML/CSS** (no frameworks needed, small custom CSS is fine)

- No full MVC, no router class — you may use **separate `.php` files per page**

  

### Required OOP Classes (minimum)

  

Put them in `src/` and autoload them via Composer:

  

| Class | Responsibility |

|-------|---------------|

| `Database` | Returns a single shared PDO connection |

| `User` | Represents a user (id, email) |

| `UserRepository` | `findByEmail()`, `create()`, `verifyPassword()` |

| `Image` | Represents an image (id, userId, filename, title, description, ...) |

| `ImageRepository` | `findAllByUser()`, `findById()`, `save()`, `delete()` |

| `ImageUploader` | Validates + moves uploaded file, returns new filename |

| `Auth` | `login()`, `logout()`, `check()`, `user()` (uses `$_SESSION`) |

| `Validator` | Simple helper for required/length checks |

  

**Rules:**

- No SQL outside `*Repository` classes

- No `$_POST` / `$_FILES` access inside classes — pass data in as arguments

- No HTML inside classes — classes return data, views print it

- Every class has **one job** (Single Responsibility)

  

### File Structure (suggested)

```

photo-gallery/

├── public/

│ ├── index.php # redirects to gallery or login

│ ├── register.php

│ ├── login.php

│ ├── logout.php

│ ├── gallery.php

│ ├── upload.php

│ ├── photo.php

│ ├── delete.php

│ └── uploads/ # user images (writable)

├── src/

│ ├── Database.php

│ ├── Auth.php

│ ├── Validator.php

│ ├── Models/

│ │ ├── User.php

│ │ └── Image.php

│ ├── Repositories/

│ │ ├── UserRepository.php

│ │ └── ImageRepository.php

│ └── Services/

│ └── ImageUploader.php

├── views/

│ ├── layout/header.php

│ ├── layout/footer.php

│ ├── auth/register.php

│ ├── auth/login.php

│ └── gallery/

│ ├── index.php

│ ├── upload.php

│ └── show.php

├── config/

│ └── config.php # DB creds (loaded from .env)

├── composer.json

├── .env.example

└── README.md

```

  

### Database Schema

  

```sql

CREATE TABLE users (

id INT AUTO_INCREMENT PRIMARY KEY,

email VARCHAR(255) NOT NULL UNIQUE,

password_hash VARCHAR(255) NOT NULL,

created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP

);

  

CREATE TABLE images (

id INT AUTO_INCREMENT PRIMARY KEY,

user_id INT NOT NULL,

filename VARCHAR(255) NOT NULL,

title VARCHAR(100) NOT NULL,

description TEXT NULL,

mime_type VARCHAR(50) NOT NULL,

size INT NOT NULL,

created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE

);

```

  

---

  

## 🔐 Security Requirements (Non-Negotiable)

  

- All DB queries use **PDO prepared statements**

- Passwords stored with `password_hash($pw, PASSWORD_DEFAULT)`

- Password check with `password_verify()`

- All output escaped with `htmlspecialchars($v, ENT_QUOTES, 'UTF-8')`

- Uploads validated with `getimagesize()` — not just extension

- Uploaded files renamed — never trust the original filename

- `uploads/` folder is **outside** web-executable PHP (block `.php` execution)

- Delete uses **POST + CSRF token**

- Session regenerated on login (`session_regenerate_id(true)`)

- On logout: `session_destroy()` + clear cookie

  

---

  

## 🎨 Views Requirements

  

- Views are plain PHP files that **only print** — no queries, no business logic

- A shared `header.php` and `footer.php`

- All pages include the same navigation:

- If logged in: **Gallery | Upload | Logout (email)**

- If guest: **Login | Register**

- Minimal CSS — a simple grid is enough

- Error messages shown under forms, not `die()`

  

---

  

## 📖 README Requirements

  

Your README must contain:

1. **Setup steps** (create DB, import SQL, configure `.env`, run `composer install`)

2. **Screenshot** of gallery page

3. **Short explanation** of each class and what it does

4. **One paragraph** on how you handled security

5. **One paragraph** on what was hardest and what you'd improve

  

---

  

## 🧪 Acceptance Test (I will run this)

  

Your app is **done** when I can:

  

1. Register a new user → get redirected to gallery

2. Upload 3 images (jpg, png, gif) → see them in grid

3. Try to upload a `.php` file → rejected with a clear message

4. Try to upload a 10 MB file → rejected

5. Open a photo → see full image + details

6. Log in as a second user → **cannot see** first user's photos

7. Manually visit `photo.php?id=1` (not mine) → 404

8. Delete my own photo → disappears from grid and disk

9. Logout → gallery redirects to login

10. Inspect code: no raw SQL, no `md5`, no `$_POST` in classes

  

---

  

## 🚫 Not Allowed (in this task)

  

- No framework (Laravel, Symfony, Slim, CodeIgniter)

- No full MVC, no custom router, no template engine

- No ORM (Eloquent, Doctrine)

- Copy-pasting tutorials — you must understand every line

  

---

  

## 🎁 Bonus (Optional, only after main is done)

  

- Pagination (10 per page)

- Search by title

- Generate thumbnails with GD

- Show count of user's images on gallery

  

---

  

## 🗓️ Suggested Order of Work

  

1. Set up Composer + autoload + DB connection

2. Create `users` table, `UserRepository`, `register.php` + `login.php`

3. Build `Auth` class + session handling

4. Build `ImageUploader` (validation first, then move file)

5. Build `ImageRepository` + upload form

6. Build gallery grid view

7. Build single photo page

8. Add delete with CSRF

9. Write README

10. Self-run acceptance test

  

---

  

## 💬 Rules Between Us

  

- You **may ask me anything**, but I won't write code for you

- Before asking, spend **20 minutes** stuck on it

- When you ask, tell me: *what you tried, what you expected, what happened*

- **Commit every day** — even small progress

- Every Friday: **demo what works + explain one class**

  

---

  

## 📤 Submission

  

When done, send me:

- Link to GitHub repo

- README

- A short video (or live demo) walking through the acceptance test

- Any question you got stuck on and how you solved it

  

---

  

Want me to also write:

- The **`composer.json` + `.env.example`** starter files?

- The **SQL setup file** ready to import?

- A **checklist you can tick per week** as a mentor?

  

Say the word and I'll generate them.

---
