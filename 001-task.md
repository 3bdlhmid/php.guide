---


---

<h1 id="task-build-a-simple-photo-gallery-php--oop-no-full-mvc">Task: Build a Simple Photo Gallery (PHP + OOP, No Full MVC)</h1>
<p><strong>Assigned to:</strong> [Friend’s name]</p>
<p><strong>Level:</strong> Beginner (knows PHP basics + OOP fundamentals)</p>
<p><strong>Estimated time:</strong> 2–3 weeks</p>
<p><strong>Deliverable:</strong> A working web app + short README</p>
<hr>
<h2 id="📌-project-brief">📌 Project Brief</h2>
<p>You are building a <strong>Photo Gallery</strong> where a logged-in user can upload photos, view them in a grid, see a single photo page, and delete their own photos.</p>
<p>You will use <strong>plain PHP with OOP classes</strong>, some <strong>HTML views</strong>, and <strong>MySQL</strong>. No framework, no full MVC — just enough structure to learn how to separate logic from presentation.</p>
<p>The goal is <strong>not</strong> to build a pretty app. The goal is to write <strong>clean, secure, well-organized PHP</strong>.</p>
<hr>
<h2 id="✅-functional-requirements-must-have">✅ Functional Requirements (Must Have)</h2>
<h3 id="authentication">1. Authentication</h3>
<ul>
<li>
<p><strong>Register</strong> with email + password</p>
</li>
<li>
<p><strong>Login</strong> with email + password</p>
</li>
<li>
<p><strong>Logout</strong></p>
</li>
<li>
<p>Only logged-in users can upload, view their gallery, or delete</p>
</li>
<li>
<p>Guests who open <code>/gallery.php</code> must be redirected to login</p>
</li>
</ul>
<h3 id="photo-upload">2. Photo Upload</h3>
<ul>
<li>
<p>Form with: <strong>image file</strong> (required), <strong>title</strong> (required, max 100 chars), <strong>description</strong> (optional, max 500 chars)</p>
</li>
<li>
<p>Accept only: <strong>JPG, JPEG, PNG, GIF</strong></p>
</li>
<li>
<p>Max file size: <strong>5 MB</strong></p>
</li>
<li>
<p>Uploaded file must be renamed to a unique name (e.g., <code>uniqid() . '.' . $ext</code>)</p>
</li>
<li>
<p>Store metadata in DB: <code>user_id</code>, <code>filename</code>, <code>title</code>, <code>description</code>, <code>mime_type</code>, <code>size</code>, <code>created_at</code></p>
</li>
<li>
<p>Save files to <code>/uploads/</code> folder (outside of PHP execution)</p>
</li>
</ul>
<h3 id="gallery-page">3. Gallery Page</h3>
<ul>
<li>
<p>Show all photos belonging to the <strong>logged-in user only</strong></p>
</li>
<li>
<p>Display in a <strong>grid</strong>: thumbnail + title</p>
</li>
<li>
<p>Ordered by newest first</p>
</li>
<li>
<p>If no photos: show a friendly empty message</p>
</li>
</ul>
<h3 id="single-photo-page">4. Single Photo Page</h3>
<ul>
<li>
<p>URL like <code>photo.php?id=5</code></p>
</li>
<li>
<p>Show: full image, title, description, upload date</p>
</li>
<li>
<p>If the photo does <strong>not</strong> belong to the current user → show <strong>404</strong> (do not reveal it exists)</p>
</li>
</ul>
<h3 id="delete-photo">5. Delete Photo</h3>
<ul>
<li>
<p>Delete button on the single photo page (only owner can see it)</p>
</li>
<li>
<p>Must delete <strong>both</strong> the DB row and the file on disk</p>
</li>
<li>
<p>Ask for confirmation (JS <code>confirm()</code> is fine)</p>
</li>
<li>
<p><strong>Use POST</strong>, not GET, for deletion</p>
</li>
</ul>
<hr>
<h2 id="🧱-technical-requirements">🧱 Technical Requirements</h2>
<h3 id="language--stack">Language / Stack</h3>
<ul>
<li>
<p><strong>PHP 8.1+</strong></p>
</li>
<li>
<p><strong>MySQL</strong> via <strong>PDO with prepared statements only</strong></p>
</li>
<li>
<p><strong>Composer</strong> for autoloading (<code>psr-4</code>)</p>
</li>
<li>
<p><strong>HTML/CSS</strong> (no frameworks needed, small custom CSS is fine)</p>
</li>
<li>
<p>No full MVC, no router class — you may use <strong>separate <code>.php</code> files per page</strong></p>
</li>
</ul>
<h3 id="required-oop-classes-minimum">Required OOP Classes (minimum)</h3>
<p>Put them in <code>src/</code> and autoload them via Composer:</p>
<p>| Class | Responsibility |</p>
<p>|-------|---------------|</p>
<p>| <code>Database</code> | Returns a single shared PDO connection |</p>
<p>| <code>User</code> | Represents a user (id, email) |</p>
<p>| <code>UserRepository</code> | <code>findByEmail()</code>, <code>create()</code>, <code>verifyPassword()</code> |</p>
<p>| <code>Image</code> | Represents an image (id, userId, filename, title, description, …) |</p>
<p>| <code>ImageRepository</code> | <code>findAllByUser()</code>, <code>findById()</code>, <code>save()</code>, <code>delete()</code> |</p>
<p>| <code>ImageUploader</code> | Validates + moves uploaded file, returns new filename |</p>
<p>| <code>Auth</code> | <code>login()</code>, <code>logout()</code>, <code>check()</code>, <code>user()</code> (uses <code>$_SESSION</code>) |</p>
<p>| <code>Validator</code> | Simple helper for required/length checks |</p>
<p><strong>Rules:</strong></p>
<ul>
<li>
<p>No SQL outside <code>*Repository</code> classes</p>
</li>
<li>
<p>No <code>$_POST</code> / <code>$_FILES</code> access inside classes — pass data in as arguments</p>
</li>
<li>
<p>No HTML inside classes — classes return data, views print it</p>
</li>
<li>
<p>Every class has <strong>one job</strong> (Single Responsibility)</p>
</li>
</ul>
<h3 id="file-structure-suggested">File Structure (suggested)</h3>
<pre><code>
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

</code></pre>
<h3 id="database-schema">Database Schema</h3>
<pre class=" language-sql"><code class="prism  language-sql">
<span class="token keyword">CREATE</span> <span class="token keyword">TABLE</span> users <span class="token punctuation">(</span>

id <span class="token keyword">INT</span> <span class="token keyword">AUTO_INCREMENT</span> <span class="token keyword">PRIMARY</span> <span class="token keyword">KEY</span><span class="token punctuation">,</span>

email <span class="token keyword">VARCHAR</span><span class="token punctuation">(</span><span class="token number">255</span><span class="token punctuation">)</span> <span class="token operator">NOT</span> <span class="token boolean">NULL</span> <span class="token keyword">UNIQUE</span><span class="token punctuation">,</span>

password_hash <span class="token keyword">VARCHAR</span><span class="token punctuation">(</span><span class="token number">255</span><span class="token punctuation">)</span> <span class="token operator">NOT</span> <span class="token boolean">NULL</span><span class="token punctuation">,</span>

created_at <span class="token keyword">TIMESTAMP</span> <span class="token keyword">DEFAULT</span> <span class="token keyword">CURRENT_TIMESTAMP</span>

<span class="token punctuation">)</span><span class="token punctuation">;</span>

  

<span class="token keyword">CREATE</span> <span class="token keyword">TABLE</span> images <span class="token punctuation">(</span>

id <span class="token keyword">INT</span> <span class="token keyword">AUTO_INCREMENT</span> <span class="token keyword">PRIMARY</span> <span class="token keyword">KEY</span><span class="token punctuation">,</span>

user_id <span class="token keyword">INT</span> <span class="token operator">NOT</span> <span class="token boolean">NULL</span><span class="token punctuation">,</span>

filename <span class="token keyword">VARCHAR</span><span class="token punctuation">(</span><span class="token number">255</span><span class="token punctuation">)</span> <span class="token operator">NOT</span> <span class="token boolean">NULL</span><span class="token punctuation">,</span>

title <span class="token keyword">VARCHAR</span><span class="token punctuation">(</span><span class="token number">100</span><span class="token punctuation">)</span> <span class="token operator">NOT</span> <span class="token boolean">NULL</span><span class="token punctuation">,</span>

description <span class="token keyword">TEXT</span> <span class="token boolean">NULL</span><span class="token punctuation">,</span>

mime_type <span class="token keyword">VARCHAR</span><span class="token punctuation">(</span><span class="token number">50</span><span class="token punctuation">)</span> <span class="token operator">NOT</span> <span class="token boolean">NULL</span><span class="token punctuation">,</span>

size <span class="token keyword">INT</span> <span class="token operator">NOT</span> <span class="token boolean">NULL</span><span class="token punctuation">,</span>

created_at <span class="token keyword">TIMESTAMP</span> <span class="token keyword">DEFAULT</span> <span class="token keyword">CURRENT_TIMESTAMP</span><span class="token punctuation">,</span>

<span class="token keyword">FOREIGN</span> <span class="token keyword">KEY</span> <span class="token punctuation">(</span>user_id<span class="token punctuation">)</span> <span class="token keyword">REFERENCES</span> users<span class="token punctuation">(</span>id<span class="token punctuation">)</span> <span class="token keyword">ON</span> <span class="token keyword">DELETE</span> <span class="token keyword">CASCADE</span>

<span class="token punctuation">)</span><span class="token punctuation">;</span>

</code></pre>
<hr>
<h2 id="🔐-security-requirements-non-negotiable">🔐 Security Requirements (Non-Negotiable)</h2>
<ul>
<li>
<p>All DB queries use <strong>PDO prepared statements</strong></p>
</li>
<li>
<p>Passwords stored with <code>password_hash($pw, PASSWORD_DEFAULT)</code></p>
</li>
<li>
<p>Password check with <code>password_verify()</code></p>
</li>
<li>
<p>All output escaped with <code>htmlspecialchars($v, ENT_QUOTES, 'UTF-8')</code></p>
</li>
<li>
<p>Uploads validated with <code>getimagesize()</code> — not just extension</p>
</li>
<li>
<p>Uploaded files renamed — never trust the original filename</p>
</li>
<li>
<p><code>uploads/</code> folder is <strong>outside</strong> web-executable PHP (block <code>.php</code> execution)</p>
</li>
<li>
<p>Delete uses <strong>POST + CSRF token</strong></p>
</li>
<li>
<p>Session regenerated on login (<code>session_regenerate_id(true)</code>)</p>
</li>
<li>
<p>On logout: <code>session_destroy()</code> + clear cookie</p>
</li>
</ul>
<hr>
<h2 id="🎨-views-requirements">🎨 Views Requirements</h2>
<ul>
<li>
<p>Views are plain PHP files that <strong>only print</strong> — no queries, no business logic</p>
</li>
<li>
<p>A shared <code>header.php</code> and <code>footer.php</code></p>
</li>
<li>
<p>All pages include the same navigation:</p>
</li>
<li>
<p>If logged in: <strong>Gallery | Upload | Logout (email)</strong></p>
</li>
<li>
<p>If guest: <strong>Login | Register</strong></p>
</li>
<li>
<p>Minimal CSS — a simple grid is enough</p>
</li>
<li>
<p>Error messages shown under forms, not <code>die()</code></p>
</li>
</ul>
<hr>
<h2 id="📖-readme-requirements">📖 README Requirements</h2>
<p>Your README must contain:</p>
<ol>
<li>
<p><strong>Setup steps</strong> (create DB, import SQL, configure <code>.env</code>, run <code>composer install</code>)</p>
</li>
<li>
<p><strong>Screenshot</strong> of gallery page</p>
</li>
<li>
<p><strong>Short explanation</strong> of each class and what it does</p>
</li>
<li>
<p><strong>One paragraph</strong> on how you handled security</p>
</li>
<li>
<p><strong>One paragraph</strong> on what was hardest and what you’d improve</p>
</li>
</ol>
<hr>
<h2 id="🧪-acceptance-test-i-will-run-this">🧪 Acceptance Test (I will run this)</h2>
<p>Your app is <strong>done</strong> when I can:</p>
<ol>
<li>
<p>Register a new user → get redirected to gallery</p>
</li>
<li>
<p>Upload 3 images (jpg, png, gif) → see them in grid</p>
</li>
<li>
<p>Try to upload a <code>.php</code> file → rejected with a clear message</p>
</li>
<li>
<p>Try to upload a 10 MB file → rejected</p>
</li>
<li>
<p>Open a photo → see full image + details</p>
</li>
<li>
<p>Log in as a second user → <strong>cannot see</strong> first user’s photos</p>
</li>
<li>
<p>Manually visit <code>photo.php?id=1</code> (not mine) → 404</p>
</li>
<li>
<p>Delete my own photo → disappears from grid and disk</p>
</li>
<li>
<p>Logout → gallery redirects to login</p>
</li>
<li>
<p>Inspect code: no raw SQL, no <code>md5</code>, no <code>$_POST</code> in classes</p>
</li>
</ol>
<hr>
<h2 id="🚫-not-allowed-in-this-task">🚫 Not Allowed (in this task)</h2>
<ul>
<li>
<p>No framework (Laravel, Symfony, Slim, CodeIgniter)</p>
</li>
<li>
<p>No full MVC, no custom router, no template engine</p>
</li>
<li>
<p>No ORM (Eloquent, Doctrine)</p>
</li>
<li>
<p>Copy-pasting tutorials — you must understand every line</p>
</li>
</ul>
<hr>
<h2 id="🎁-bonus-optional-only-after-main-is-done">🎁 Bonus (Optional, only after main is done)</h2>
<ul>
<li>
<p>Pagination (10 per page)</p>
</li>
<li>
<p>Search by title</p>
</li>
<li>
<p>Generate thumbnails with GD</p>
</li>
<li>
<p>Show count of user’s images on gallery</p>
</li>
</ul>
<hr>
<h2 id="🗓️-suggested-order-of-work">🗓️ Suggested Order of Work</h2>
<ol>
<li>
<p>Set up Composer + autoload + DB connection</p>
</li>
<li>
<p>Create <code>users</code> table, <code>UserRepository</code>, <code>register.php</code> + <code>login.php</code></p>
</li>
<li>
<p>Build <code>Auth</code> class + session handling</p>
</li>
<li>
<p>Build <code>ImageUploader</code> (validation first, then move file)</p>
</li>
<li>
<p>Build <code>ImageRepository</code> + upload form</p>
</li>
<li>
<p>Build gallery grid view</p>
</li>
<li>
<p>Build single photo page</p>
</li>
<li>
<p>Add delete with CSRF</p>
</li>
<li>
<p>Write README</p>
</li>
<li>
<p>Self-run acceptance test</p>
</li>
</ol>
<hr>
<h2 id="💬-rules-between-us">💬 Rules Between Us</h2>
<ul>
<li>
<p>You <strong>may ask me anything</strong>, but I won’t write code for you</p>
</li>
<li>
<p>Before asking, spend <strong>20 minutes</strong> stuck on it</p>
</li>
<li>
<p>When you ask, tell me: <em>what you tried, what you expected, what happened</em></p>
</li>
<li>
<p><strong>Commit every day</strong> — even small progress</p>
</li>
<li>
<p>Every Friday: <strong>demo what works + explain one class</strong></p>
</li>
</ul>
<hr>
<h2 id="📤-submission">📤 Submission</h2>
<p>When done, send me:</p>
<ul>
<li>
<p>Link to GitHub repo</p>
</li>
<li>
<p>README</p>
</li>
<li>
<p>A short video (or live demo) walking through the acceptance test</p>
</li>
<li>
<p>Any question you got stuck on and how you solved it</p>
</li>
</ul>
<hr>
<p>Want me to also write:</p>
<ul>
<li>
<p>The <strong><code>composer.json</code> + <code>.env.example</code></strong> starter files?</p>
</li>
<li>
<p>The <strong>SQL setup file</strong> ready to import?</p>
</li>
<li>
<p>A <strong>checklist you can tick per week</strong> as a mentor?</p>
</li>
</ul>
<p>Say the word and I’ll generate them.</p>
<hr>

