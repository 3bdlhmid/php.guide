---


---

<h2 id="🗺️-the-big-picture-what-were-building">🗺️ The Big Picture (What We’re Building)</h2>
<p>A tiny web app where:</p>
<ul>
<li>
<p>A user registers, logs in</p>
</li>
<li>
<p>Uploads images</p>
</li>
<li>
<p>Sees their own images in a grid</p>
</li>
<li>
<p>Opens one image</p>
</li>
<li>
<p>Deletes it</p>
</li>
</ul>
<p><strong>That’s it.</strong> Every decision below is filtered through: <em>does this serve those 5 things and nothing more?</em></p>
<hr>
<h2 id="🔀-flow-1-—-registration">🔀 Flow 1 — Registration</h2>
<h3 id="steps-what-actually-happens">Steps (what actually happens)</h3>
<ol>
<li>
<p>User opens <code>register.php</code> → sees form</p>
</li>
<li>
<p>Submits: email + password + confirm password</p>
</li>
<li>
<p>Server validates:</p>
</li>
</ol>
<ul>
<li>
<p>Email format valid</p>
</li>
<li>
<p>Email not already in DB</p>
</li>
<li>
<p>Password ≥ 8 chars</p>
</li>
<li>
<p>Passwords match</p>
</li>
</ul>
<ol start="4">
<li>
<p>Hash password</p>
</li>
<li>
<p>Insert user row</p>
</li>
<li>
<p>Log them in automatically</p>
</li>
<li>
<p>Redirect to <code>gallery.php</code></p>
</li>
</ol>
<h3 id="decisions-in-this-flow">Decisions in this flow</h3>
<p>| Decision | Options | Chosen | Why |</p>
<p>|—|---|—|---|</p>
<p>| Hash algorithm | md5 / sha1 / <code>password_hash</code> | <code>password_hash(PASSWORD_DEFAULT)</code> | md5 is broken; <code>password_hash</code> is future-proof and one line |</p>
<p>| Auto-login after register | yes / no | <strong>yes</strong> | Fewer steps = fewer bug surfaces for beginners |</p>
<p>| Confirm password field | yes / no | <strong>yes</strong> | Client-side sanity check, teaches validation flow |</p>
<p>| Where validation lives | in page / in <code>Validator</code> class | <strong><code>Validator</code> class</strong> | Reused across register/login/upload — DRY |</p>
<p>| Unique email check | in <code>UserRepository::findByEmail()</code> | <strong>yes</strong> | Keeps SQL out of the page |</p>
<p>| Error display | <code>die()</code> / array of errors / flash | <strong>array of errors</strong> | Simple, inline, no session needed yet |</p>
<h3 id="trade-off-noted">Trade-off noted</h3>
<p>Auto-login skips email verification. <strong>Accepted</strong> — no email system in scope. Would revisit only if real users were involved.</p>
<hr>
<h2 id="🔀-flow-2-—-login">🔀 Flow 2 — Login</h2>
<h3 id="steps">Steps</h3>
<ol>
<li>
<p>User opens <code>login.php</code> → sees form</p>
</li>
<li>
<p>Submits: email + password</p>
</li>
<li>
<p>Server:</p>
</li>
</ol>
<ul>
<li>
<p>Finds user by email (<code>UserRepository::findByEmail</code>)</p>
</li>
<li>
<p>If not found → generic error: <em>“Invalid credentials”</em></p>
</li>
<li>
<p>Verifies password with <code>password_verify</code></p>
</li>
<li>
<p>If wrong → same generic error</p>
</li>
</ul>
<ol start="4">
<li>On success:</li>
</ol>
<ul>
<li>
<p><code>session_regenerate_id(true)</code></p>
</li>
<li>
<p>Store <code>user_id</code> and <code>email</code> in <code>$_SESSION</code></p>
</li>
<li>
<p>Redirect to <code>gallery.php</code></p>
</li>
</ul>
<h3 id="decisions">Decisions</h3>
<p>| Decision | Options | Chosen | Why |</p>
<p>|—|---|—|---|</p>
<p>| Error specificity | “Email not found” vs “Invalid credentials” | <strong>generic</strong> | Prevents user enumeration — small habit, big lesson |</p>
<p>| Session storage | <code>$_SESSION['user'] = $userObject</code> / just IDs | <strong>just IDs</strong> | Objects get stale; IDs are stable |</p>
<p>| Session regeneration | always / only on login | <strong>only on login</strong> | Prevents session fixation |</p>
<p>| Auth logic location | page code / <code>Auth</code> class | <strong><code>Auth</code> class</strong> | Every protected page needs it — single source of truth |</p>
<p>| “Remember me” | yes / no | <strong>no</strong> | Out of scope; adds cookie + token complexity |</p>
<h3 id="trade-off-noted-1">Trade-off noted</h3>
<p>No rate limiting on login. <strong>Accepted</strong> — local project, no attacker. Would revisit if exposed to internet.</p>
<hr>
<h2 id="🔀-flow-3-—-protecting-pages">🔀 Flow 3 — Protecting Pages</h2>
<h3 id="the-pattern">The pattern</h3>
<p>Every protected page starts with:</p>
<pre class=" language-php"><code class="prism  language-php">
<span class="token keyword">require_once</span> <span class="token constant">__DIR__</span> <span class="token punctuation">.</span> <span class="token string">'/../src/bootstrap.php'</span><span class="token punctuation">;</span>

Auth<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">requireLogin</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">// redirects to login.php if not logged in</span>

</code></pre>
<h3 id="decisions-1">Decisions</h3>
<p>| Decision | Options | Chosen | Why |</p>
<p>|—|---|—|---|</p>
<p>| Manual check per page vs helper | manual / <code>Auth::requireLogin()</code> | <strong>helper</strong> | One line, cannot forget, consistent |</p>
<p>| What <code>requireLogin</code> does | redirects / returns false / throws | <strong>redirects</strong> | Simplest for a beginner’s mental model |</p>
<p>| Where to store “intended URL” | session / query string / skip | <strong>skip</strong> | Not worth complexity for this project |</p>
<hr>
<h2 id="🔀-flow-4-—-upload">🔀 Flow 4 — Upload</h2>
<h3 id="steps-1">Steps</h3>
<ol>
<li>
<p>User opens <code>upload.php</code> → form</p>
</li>
<li>
<p>Submits: file + title + description</p>
</li>
<li>
<p>Server:</p>
</li>
</ol>
<ul>
<li>
<p>Check logged in</p>
</li>
<li>
<p>Validate: file present, size ≤ 5MB, real image, allowed MIME (jpg/png/gif)</p>
</li>
<li>
<p>Validate: title non-empty, ≤ 100 chars</p>
</li>
<li>
<p>Generate new filename: <code>uniqid() . '.' . $ext</code></p>
</li>
<li>
<p>Move file from tmp → <code>uploads/</code></p>
</li>
<li>
<p>Save row in DB via <code>ImageRepository::save()</code></p>
</li>
</ul>
<ol start="4">
<li>Redirect to <code>gallery.php</code> (PRG pattern — Post/Redirect/Get)</li>
</ol>
<h3 id="decisions-2">Decisions</h3>
<p>| Decision | Options | Chosen | Why |</p>
<p>|—|---|—|---|</p>
<p>| Validate before move | always | <strong>yes</strong> | Never move an unvalidated file |</p>
<p>| File type check | extension / MIME / <code>getimagesize</code> | <strong><code>getimagesize</code> + MIME</strong> | Extension lies; MIME header can lie; <code>getimagesize</code> reads bytes |</p>
<p>| Original filename | keep / rename | <strong>rename</strong> | Prevents path traversal (<code>../../etc/passwd</code>) and collisions |</p>
<p>| Where uploads live | same folder as code / <code>public/uploads/</code> | <strong><code>public/uploads/</code></strong> | Must be web-accessible to be shown |</p>
<p>| Protect <code>uploads/</code> from PHP execution | .htaccess / server config / ignore | <strong>.htaccess</strong> | Cheapest way to say “never run PHP here” |</p>
<p>| After success | render page / redirect | <strong>redirect</strong> | Refresh won’t re-upload; classic PRG lesson |</p>
<p>| Save metadata | in file name / in DB | <strong>DB</strong> | Searchable, sortable, ownable |</p>
<h3 id="trade-off-noted-2">Trade-off noted</h3>
<p>No image resizing. <strong>Accepted</strong> — adds GD complexity. Revisit only if storage becomes a problem.</p>
<hr>
<h2 id="🔀-flow-5-—-gallery">🔀 Flow 5 — Gallery</h2>
<h3 id="steps-2">Steps</h3>
<ol>
<li>
<p>User opens <code>gallery.php</code></p>
</li>
<li>
<p><code>Auth::requireLogin()</code></p>
</li>
<li>
<p><code>$images = $imageRepo-&gt;findAllByUser(Auth::id())</code></p>
</li>
<li>
<p>Loop and render grid</p>
</li>
<li>
<p>If empty → show “No photos yet — upload your first”</p>
</li>
</ol>
<h3 id="decisions-3">Decisions</h3>
<p>| Decision | Options | Chosen | Why |</p>
<p>|—|---|—|---|</p>
<p>| Filter by user | yes / show all | <strong>yes</strong> | Ownership is a core requirement — teaches authorization |</p>
<p>| Sort order | oldest / newest | <strong>newest</strong> | Users expect latest first |</p>
<p>| Pagination | yes / no | <strong>no (yet)</strong> | YAGNI — add when &gt; 50 images |</p>
<p>| Thumbnails | generate / use original | <strong>original</strong> | Avoids GD; CSS shrinks the display. Add real thumbnails later |</p>
<p>| Query location | page / repo | <strong>repo</strong> | Consistent with other flows |</p>
<h3 id="trade-off-noted-3">Trade-off noted</h3>
<p>Loading full-size images in a grid is slow. <strong>Accepted</strong> — fine for a learning project. Real fix: generate thumbnails on upload.</p>
<hr>
<h2 id="🔀-flow-6-—-view-single-photo">🔀 Flow 6 — View Single Photo</h2>
<h3 id="steps-3">Steps</h3>
<ol>
<li>
<p>URL: <code>photo.php?id=5</code></p>
</li>
<li>
<p><code>Auth::requireLogin()</code></p>
</li>
<li>
<p><code>$image = $imageRepo-&gt;findById((int) $_GET['id'])</code></p>
</li>
<li>
<p><strong>Ownership check</strong>: <code>$image-&gt;userId !== Auth::id()</code> → <strong>404, not 403</strong></p>
</li>
<li>
<p>If null → 404</p>
</li>
<li>
<p>Render full image + details</p>
</li>
</ol>
<h3 id="decisions-4">Decisions</h3>
<p>| Decision | Options | Chosen | Why |</p>
<p>|—|---|—|---|</p>
<p>| Cast <code>$_GET['id']</code> | raw / <code>(int)</code> | <strong><code>(int)</code></strong> | Prevents type confusion, mandatory habit |</p>
<p>| Not-found response | 404 / 403 / blank | <strong>404</strong> | 403 reveals existence; 404 hides it — good security lesson |</p>
<p>| Same error for “not found” and “not yours” | yes / no | <strong>yes</strong> | Don’t leak whether the resource exists |</p>
<p>| Data shape returned | array / <code>Image</code> object | <strong><code>Image</code> object</strong> | Teaches value objects; methods like <code>-&gt;getUrl()</code> live on the object |</p>
<hr>
<h2 id="🔀-flow-7-—-delete">🔀 Flow 7 — Delete</h2>
<h3 id="steps-4">Steps</h3>
<ol>
<li>
<p>Button on <code>photo.php</code> → posts to <code>delete.php</code></p>
</li>
<li>
<p>Server:</p>
</li>
</ol>
<ul>
<li>
<p><code>Auth::requireLogin()</code></p>
</li>
<li>
<p>Check CSRF token</p>
</li>
<li>
<p>Load image, check ownership</p>
</li>
<li>
<p>Delete file from disk</p>
</li>
<li>
<p>Delete row from DB</p>
</li>
<li>
<p>Redirect to <code>gallery.php</code> with flash message</p>
</li>
</ul>
<h3 id="decisions-5">Decisions</h3>
<p>| Decision | Options | Chosen | Why |</p>
<p>|—|---|—|---|</p>
<p>| Method | GET / POST | <strong>POST</strong> | GET is for reading; deletes mutate state |</p>
<p>| Confirmation | none / JS confirm / server page | <strong>JS <code>confirm()</code></strong> | Cheapest UX, no extra page |</p>
<p>| Order: file vs DB | file first / DB first | <strong>file first, then DB</strong> | If file delete fails, DB still points to real file (recoverable). If DB deletes first and file fails, orphan file — harder to trace |</p>
<p>| CSRF | yes / no | <strong>yes</strong> | Teaches a real security habit; cheap to add |</p>
<p>| Authorization | owner only | <strong>owner only</strong> | Matches gallery rule |</p>
<h3 id="trade-off-noted-4">Trade-off noted</h3>
<p>JS <code>confirm()</code> can be bypassed. <strong>Accepted</strong> — the real defense is server-side ownership + CSRF, which we have.</p>
<hr>
<h2 id="🧱-structural-decisions-app-wide">🧱 Structural Decisions (App-wide)</h2>
<p>| Decision | Chosen | Why |</p>
<p>|—|---|—|</p>
<p>| Pattern | Plain pages + classes (no MVC) | Keeps request flow visible; teaches separation without routing magic |</p>
<p>| Class loading | <code>require_once</code> via <code>bootstrap.php</code> | Beginner-friendly; introduces Composer later as a <em>felt</em> upgrade |</p>
<p>| Database access | PDO + prepared statements in repositories | No SQL in pages; no injection risk |</p>
<p>| Auth | Session-based | Simplest correct model for a monolith |</p>
<p>| Views | Plain PHP in <code>views/</code> | No template engine to hide the output logic |</p>
<p>| Config | <code>.env</code> parsed by tiny loader | Introduces secrets hygiene without Composer |</p>
<p>| File structure | <code>public/</code> as web root | Good habit early; avoids exposing <code>src/</code> |</p>
<hr>
<h2 id="🚫-out-of-scope-explicitly-decided-against">🚫 Out of Scope (Explicitly Decided Against)</h2>
<p>Write these down. They prevent scope creep.</p>
<ul>
<li>
<p>❌ Email verification</p>
</li>
<li>
<p>❌ Password reset</p>
</li>
<li>
<p>❌ User roles / admin</p>
</li>
<li>
<p>❌ Social login</p>
</li>
<li>
<p>❌ Likes / comments</p>
</li>
<li>
<p>❌ Albums / tags</p>
</li>
<li>
<p>❌ Search</p>
</li>
<li>
<p>❌ Pagination</p>
</li>
<li>
<p>❌ Thumbnail generation</p>
</li>
<li>
<p>❌ REST API</p>
</li>
<li>
<p>❌ Framework / MVC / router</p>
</li>
<li>
<p>❌ Composer (for now)</p>
</li>
<li>
<p>❌ Unit tests (for now)</p>
</li>
<li>
<p>❌ Docker / deployment</p>
</li>
</ul>
<p>Every “no” above is a <strong>decision</strong> — not an oversight. Tell your friend: <em>“These are decided. If you want one, bring a constraint that justifies it.”</em></p>
<hr>
<h2 id="🧭-decision-log-template-give-him-this">🧭 Decision Log Template (Give Him This)</h2>
<p>For every non-obvious choice, he writes:</p>
<pre><code>
Date:

Question:

Options:

Chosen:

Why:

Trade-off accepted:

Revisit if:

</code></pre>
<p>Example entry:</p>
<pre><code>
Date: 2026-02-14

Question: Where to store uploaded files?

Options: (A) DB blob (B) disk + DB path (C) disk only

Chosen: B

Why: Fast, inspectable, supports ownership + metadata

Trade-off accepted: Two sources of truth → must clean both on delete

Revisit if: Moving to cloud/S3, or file count &gt; 10k

</code></pre>
<p>Do this <strong>10 times</strong> during the project. By the end, he thinks like a dev, not a tutorial-follower.</p>
<hr>
<h2 id="🎯-the-one-rule-for-this-project">🎯 The One Rule for This Project</h2>
<blockquote>
<p><strong>If a decision doesn’t serve one of the 7 flows above, it’s out of scope.</strong></p>
</blockquote>
<p>That single sentence replaces a 50-page spec. Print it. Tape it to his monitor.</p>
<hr>
<h2 id="📌-what-to-review-with-him-weekly">📌 What to Review With Him Weekly</h2>
<p>Ask these 5 questions every Friday:</p>
<ol>
<li>
<p>Which flow did you finish?</p>
</li>
<li>
<p>What was the hardest decision this week? Show me your log entry.</p>
</li>
<li>
<p>Which constraint did you hit? Did it change anything?</p>
</li>
<li>
<p>What did you decide <em>not</em> to do this week, and why?</p>
</li>
<li>
<p>If you started over, what’s one decision you’d make differently?</p>
</li>
</ol>
<p>The answers reveal more than the code ever will.</p>
<hr>
<p>Want me to also produce:</p>
<ul>
<li>
<p>A <strong>fill-in-the-blank decision log file</strong> he can copy into the repo?</p>
</li>
<li>
<p>A <strong>flow diagram</strong> (text/ASCII) for each of the 7 flows to paste in the README?</p>
</li>
<li>
<p>A <strong>“red flags in code review”</strong> checklist tied to these flows?</p>
</li>
</ul>
<p>Say which and I’ll draft it.</p>

