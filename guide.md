
## 🗺️ The Big Picture (What We're Building)

  

A tiny web app where:

- A user registers, logs in

- Uploads images

- Sees their own images in a grid

- Opens one image

- Deletes it

  

**That's it.** Every decision below is filtered through: *does this serve those 5 things and nothing more?*

  

---

  

## 🔀 Flow 1 — Registration

  

### Steps (what actually happens)

1. User opens `register.php` → sees form

2. Submits: email + password + confirm password

3. Server validates:

- Email format valid

- Email not already in DB

- Password ≥ 8 chars

- Passwords match

4. Hash password

5. Insert user row

6. Log them in automatically

7. Redirect to `gallery.php`

  

### Decisions in this flow

  

| Decision | Options | Chosen | Why |

|---|---|---|---|

| Hash algorithm | md5 / sha1 / `password_hash` | `password_hash(PASSWORD_DEFAULT)` | md5 is broken; `password_hash` is future-proof and one line |

| Auto-login after register | yes / no | **yes** | Fewer steps = fewer bug surfaces for beginners |

| Confirm password field | yes / no | **yes** | Client-side sanity check, teaches validation flow |

| Where validation lives | in page / in `Validator` class | **`Validator` class** | Reused across register/login/upload — DRY |

| Unique email check | in `UserRepository::findByEmail()` | **yes** | Keeps SQL out of the page |

| Error display | `die()` / array of errors / flash | **array of errors** | Simple, inline, no session needed yet |

  

### Trade-off noted

Auto-login skips email verification. **Accepted** — no email system in scope. Would revisit only if real users were involved.

  

---

  

## 🔀 Flow 2 — Login

  

### Steps

1. User opens `login.php` → sees form

2. Submits: email + password

3. Server:

- Finds user by email (`UserRepository::findByEmail`)

- If not found → generic error: *"Invalid credentials"*

- Verifies password with `password_verify`

- If wrong → same generic error

4. On success:

- `session_regenerate_id(true)`

- Store `user_id` and `email` in `$_SESSION`

- Redirect to `gallery.php`

  

### Decisions

  

| Decision | Options | Chosen | Why |

|---|---|---|---|

| Error specificity | "Email not found" vs "Invalid credentials" | **generic** | Prevents user enumeration — small habit, big lesson |

| Session storage | `$_SESSION['user'] = $userObject` / just IDs | **just IDs** | Objects get stale; IDs are stable |

| Session regeneration | always / only on login | **only on login** | Prevents session fixation |

| Auth logic location | page code / `Auth` class | **`Auth` class** | Every protected page needs it — single source of truth |

| "Remember me" | yes / no | **no** | Out of scope; adds cookie + token complexity |

  

### Trade-off noted

No rate limiting on login. **Accepted** — local project, no attacker. Would revisit if exposed to internet.

  

---

  

## 🔀 Flow 3 — Protecting Pages

  

### The pattern

Every protected page starts with:

```php

require_once __DIR__ . '/../src/bootstrap.php';

Auth::requireLogin(); // redirects to login.php if not logged in

```

  

### Decisions

  

| Decision | Options | Chosen | Why |

|---|---|---|---|

| Manual check per page vs helper | manual / `Auth::requireLogin()` | **helper** | One line, cannot forget, consistent |

| What `requireLogin` does | redirects / returns false / throws | **redirects** | Simplest for a beginner's mental model |

| Where to store "intended URL" | session / query string / skip | **skip** | Not worth complexity for this project |

  

---

  

## 🔀 Flow 4 — Upload

  

### Steps

1. User opens `upload.php` → form

2. Submits: file + title + description

3. Server:

- Check logged in

- Validate: file present, size ≤ 5MB, real image, allowed MIME (jpg/png/gif)

- Validate: title non-empty, ≤ 100 chars

- Generate new filename: `uniqid() . '.' . $ext`

- Move file from tmp → `uploads/`

- Save row in DB via `ImageRepository::save()`

4. Redirect to `gallery.php` (PRG pattern — Post/Redirect/Get)

  

### Decisions

  

| Decision | Options | Chosen | Why |

|---|---|---|---|

| Validate before move | always | **yes** | Never move an unvalidated file |

| File type check | extension / MIME / `getimagesize` | **`getimagesize` + MIME** | Extension lies; MIME header can lie; `getimagesize` reads bytes |

| Original filename | keep / rename | **rename** | Prevents path traversal (`../../etc/passwd`) and collisions |

| Where uploads live | same folder as code / `public/uploads/` | **`public/uploads/`** | Must be web-accessible to be shown |

| Protect `uploads/` from PHP execution | .htaccess / server config / ignore | **.htaccess** | Cheapest way to say "never run PHP here" |

| After success | render page / redirect | **redirect** | Refresh won't re-upload; classic PRG lesson |

| Save metadata | in file name / in DB | **DB** | Searchable, sortable, ownable |

  

### Trade-off noted

No image resizing. **Accepted** — adds GD complexity. Revisit only if storage becomes a problem.

  

---

  

## 🔀 Flow 5 — Gallery

  

### Steps

1. User opens `gallery.php`

2. `Auth::requireLogin()`

3. `$images = $imageRepo->findAllByUser(Auth::id())`

4. Loop and render grid

5. If empty → show "No photos yet — upload your first"

  

### Decisions

  

| Decision | Options | Chosen | Why |

|---|---|---|---|

| Filter by user | yes / show all | **yes** | Ownership is a core requirement — teaches authorization |

| Sort order | oldest / newest | **newest** | Users expect latest first |

| Pagination | yes / no | **no (yet)** | YAGNI — add when > 50 images |

| Thumbnails | generate / use original | **original** | Avoids GD; CSS shrinks the display. Add real thumbnails later |

| Query location | page / repo | **repo** | Consistent with other flows |

  

### Trade-off noted

Loading full-size images in a grid is slow. **Accepted** — fine for a learning project. Real fix: generate thumbnails on upload.

  

---

  

## 🔀 Flow 6 — View Single Photo

  

### Steps

1. URL: `photo.php?id=5`

2. `Auth::requireLogin()`

3. `$image = $imageRepo->findById((int) $_GET['id'])`

4. **Ownership check**: `$image->userId !== Auth::id()` → **404, not 403**

5. If null → 404

6. Render full image + details

  

### Decisions

  

| Decision | Options | Chosen | Why |

|---|---|---|---|

| Cast `$_GET['id']` | raw / `(int)` | **`(int)`** | Prevents type confusion, mandatory habit |

| Not-found response | 404 / 403 / blank | **404** | 403 reveals existence; 404 hides it — good security lesson |

| Same error for "not found" and "not yours" | yes / no | **yes** | Don't leak whether the resource exists |

| Data shape returned | array / `Image` object | **`Image` object** | Teaches value objects; methods like `->getUrl()` live on the object |

  

---

  

## 🔀 Flow 7 — Delete

  

### Steps

1. Button on `photo.php` → posts to `delete.php`

2. Server:

- `Auth::requireLogin()`

- Check CSRF token

- Load image, check ownership

- Delete file from disk

- Delete row from DB

- Redirect to `gallery.php` with flash message

  

### Decisions

  

| Decision | Options | Chosen | Why |

|---|---|---|---|

| Method | GET / POST | **POST** | GET is for reading; deletes mutate state |

| Confirmation | none / JS confirm / server page | **JS `confirm()`** | Cheapest UX, no extra page |

| Order: file vs DB | file first / DB first | **file first, then DB** | If file delete fails, DB still points to real file (recoverable). If DB deletes first and file fails, orphan file — harder to trace |

| CSRF | yes / no | **yes** | Teaches a real security habit; cheap to add |

| Authorization | owner only | **owner only** | Matches gallery rule |

  

### Trade-off noted

JS `confirm()` can be bypassed. **Accepted** — the real defense is server-side ownership + CSRF, which we have.

  

---

  

## 🧱 Structural Decisions (App-wide)

  

| Decision | Chosen | Why |

|---|---|---|

| Pattern | Plain pages + classes (no MVC) | Keeps request flow visible; teaches separation without routing magic |

| Class loading | `require_once` via `bootstrap.php` | Beginner-friendly; introduces Composer later as a *felt* upgrade |

| Database access | PDO + prepared statements in repositories | No SQL in pages; no injection risk |

| Auth | Session-based | Simplest correct model for a monolith |

| Views | Plain PHP in `views/` | No template engine to hide the output logic |

| Config | `.env` parsed by tiny loader | Introduces secrets hygiene without Composer |

| File structure | `public/` as web root | Good habit early; avoids exposing `src/` |

  

---

  

## 🚫 Out of Scope (Explicitly Decided Against)

  

Write these down. They prevent scope creep.

  

- ❌ Email verification

- ❌ Password reset

- ❌ User roles / admin

- ❌ Social login

- ❌ Likes / comments

- ❌ Albums / tags

- ❌ Search

- ❌ Pagination

- ❌ Thumbnail generation

- ❌ REST API

- ❌ Framework / MVC / router

- ❌ Composer (for now)

- ❌ Unit tests (for now)

- ❌ Docker / deployment

  

Every "no" above is a **decision** — not an oversight. Tell your friend: *"These are decided. If you want one, bring a constraint that justifies it."*

  

---

  

## 🧭 Decision Log Template (Give Him This)

  

For every non-obvious choice, he writes:

  

```

Date:

Question:

Options:

Chosen:

Why:

Trade-off accepted:

Revisit if:

```

  

Example entry:

  

```

Date: 2026-02-14

Question: Where to store uploaded files?

Options: (A) DB blob (B) disk + DB path (C) disk only

Chosen: B

Why: Fast, inspectable, supports ownership + metadata

Trade-off accepted: Two sources of truth → must clean both on delete

Revisit if: Moving to cloud/S3, or file count > 10k

```

  

Do this **10 times** during the project. By the end, he thinks like a dev, not a tutorial-follower.

  

---

  

## 🎯 The One Rule for This Project

  

> **If a decision doesn't serve one of the 7 flows above, it's out of scope.**

  

That single sentence replaces a 50-page spec. Print it. Tape it to his monitor.

  

---

  

## 📌 What to Review With Him Weekly

  

Ask these 5 questions every Friday:

  

1. Which flow did you finish?

2. What was the hardest decision this week? Show me your log entry.

3. Which constraint did you hit? Did it change anything?

4. What did you decide *not* to do this week, and why?

5. If you started over, what's one decision you'd make differently?

  

The answers reveal more than the code ever will.

  

---

  

Want me to also produce:

- A **fill-in-the-blank decision log file** he can copy into the repo?

- A **flow diagram** (text/ASCII) for each of the 7 flows to paste in the README?

- A **"red flags in code review"** checklist tied to these flows?

  

Say which and I'll draft it.
