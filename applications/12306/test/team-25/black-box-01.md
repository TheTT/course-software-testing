# black-box-01 — A registered account cannot sign in with its own credentials once its identifier also matches another account

- **Type:** Black-box
- **Requirement(s):** REQ-2.2.2, REQ-2.1.1
- **Affected area:** Login (`Email/Username/Mobile number` + `Password` → `LOGIN`)

---

## 1. Test method and design basis

**Method:** black-box testing, equivalence partitioning over the *login identifier domain*
declared by REQ-2.2.2 (username / email address / mobile number).

The existing baseline tests each identifier class **in isolation, on a single account**
(`REQ-2.2.2.spec.ts` authenticates one clean account by username, then by email, then
by mobile). They never create a state in which **one string is simultaneously a valid
username of one account and a valid email address of another**.

This test adds that input class, built entirely through the declared registration flow:

| Input class | Accounts involved | Selected because |
| --- | --- | --- |
| Single-account identifier | 1 | Already covered by `REQ-2.2.2.spec.ts` |
| **Cross-account identifier collision** | 2 | **Not covered — the case under test** |

The collision is legal under the registration contract: registration enforces uniqueness
of the **passport number and username only** (REQ-2.1.1); it never requires email addresses
to be unique, and it never forbids a new email address from being equal to an existing
username. Therefore the state is reachable using only specified functionality.

---

## 2. Requirement reference

**REQ-2.2.2 — Authenticate traveler credentials**

> The login form **accepts a verified username, email address, or mobile number** in its
> "Email/Username/Mobile number" field and a password in its "Password" field. Activating
> the "LOGIN" button with **valid credentials persists the session**, displays a status
> message containing "Login successful", and exposes a "Sign Out" link.
>
> Reject login when the password is incorrect and show the message "Incorrect password.".

**REQ-2.1.1 — Register a traveler account** (scenario `SCN-REQ-2.1.1-01`)

> Then: Registration successful. is displayed; **the saved credentials sign in after
> reload**, and Sign Out remains visible after the second reload.

**Shared registration contract (REQ-2.1.1)**

> A successful submission persists the account and displays "Registration successful.".
> **The passport number and username must be unique.**

⇒ Two conclusions used to derive the expected result:

1. Registering account B with `username = bb@t.com` and `email = aa@t.com` must succeed,
   because neither field violates the declared uniqueness rules (B's username is new,
   and email uniqueness is not required).
2. After that registration, B's own credentials (`bb@t.com` + B's password) must establish
   B's session, per REQ-2.2.2 and per the "saved credentials sign in" clause of REQ-2.1.1.

---

## 3. Difference from existing tests

| Existing baseline test | What it covers | What it does **not** cover |
| --- | --- | --- |
| `REQ-2.2.2.spec.ts` – *Authenticate by username email and mobile* | The three identifier forms of **one** account | Never two accounts whose identifier domains overlap |
| `REQ-2.2.2.spec.ts` – *Reject wrong/unknown/missing* | Single-account negative branches | Never a *valid* credential set rejected because of another account |
| `REQ-2.1.2.spec.ts` – *Create durable account* | Account persistence after reload | Does not sign in afterwards with the created credentials |
| `REQ-2.1.3.spec.ts` – *Reject duplicate passport / duplicate username* | Uniqueness of passport and username | Does not exercise the permitted overlap between **email and username** |

No existing test constructs the state required to observe this behaviour.

---

## 4. Preconditions and reproduction steps

**Precondition:** a freshly initialised application instance (`.data/` removed before
startup, so no account other than the seeded ones exists), logged out.

> ⚠️ **Registration order matters.** The account registered **first** is the one whose
> record takes precedence, so account A must be created before account B.

| Step | Action | Value |
| --- | --- | --- |
| 1 | Open **Register** | — |
| 2 | Register **account A** | Name `na`, Passport `1`, Username `aa@t.com`, Email `bb@t.com`, Password `aa`, Confirm Password `aa`, Nationality `China`, Passport expiration date `22220202`, Date of birth `20001111`, Gender `Male`, agreement checkbox ticked |
| 3 | Confirm the result | `Registration successful.` (see `assets/register_a.png`) |
| 4 | Open **Register** again | — |
| 5 | Register **account B** | Name `nb`, Passport `2`, Username `bb@t.com`, Email `aa@t.com`, Password `bb`, Confirm Password `bb`, all other fields as in step 2 |
| 6 | Confirm the result | `Registration successful.` (see `assets/register_b.png`) |
| 7 | Open **Login**, enter `aa@t.com` / `aa`, click **LOGIN** | Succeeds as account A |
| 8 | Sign out, enter `aa@t.com` / `bb`, click **LOGIN** | — |
| 9 | Sign out, enter `bb@t.com` / `aa`, click **LOGIN** | — |
| 10 | Sign out, enter `bb@t.com` / `bb`, click **LOGIN** | — |

The only structural requirement on the data is the collision:
`A.username == B.email == aa@t.com` and `B.username == A.email == bb@t.com`.
Account A is registered **before** account B; the order determines which record wins.

---

## 5. Expected and actual behaviour

**Expected** (derived from REQ-2.2.2 + REQ-2.1.1): each account authenticates with its own
credentials, because both were created successfully and each username/password pair is valid.

| # | Identifier | Password | Belongs to | Expected | Actual |
| --- | --- | --- | --- | --- | --- |
| 1 | `aa@t.com` | `aa` | account A (username) | Login succeeds → **account A**, `Sign Out` visible | Login succeeds → account A |
| 2 | `aa@t.com` | `bb` | account B (email) | Login succeeds → **account B** | **`Incorrect password.`** |
| 3 | `bb@t.com` | `aa` | account A (email) | Login succeeds → **account A** | Login succeeds → account A |
| 4 | `bb@t.com` | `bb` | account B (username) | Login succeeds → **account B** | **`Incorrect password.`** |

**Actual:** account B can never authenticate. Both of its identifiers are resolved to
account A's record, so only account A's password is ever accepted and B's own password is
reported as `Incorrect password.`.

This is visible in the evidence: when row 3 succeeds, the signed-in session is headed
`aa@t.com` and the Personal Center reports **Account number `aa@t.com` / Security mailbox
`bb@t.com`**, i.e. the session belongs to account A even though the identifier entered was
account B's username `bb@t.com`.

---

## 6. Evidence

| Evidence | File | Shows |
| --- | --- | --- |
| Registration of account A | `assets/register_a.png` | Filled values (Username `aa@t.com`, Email `bb@t.com`) and `Registration successful.` |
| Registration of account B | `assets/register_b.png` | Filled values (Username `bb@t.com`, Email `aa@t.com`) and `Registration successful.` |
| Row 1: `aa@t.com` / `aa` | `assets/login_a_pa.png` | Login succeeds; header shows `aa@t.com` and `Sign Out`; Personal Center reports Account number `aa@t.com` |
| Row 2: `aa@t.com` / `bb` | `assets/login_a_pb.png` | `Incorrect password.`; still logged out |
| Row 3: `bb@t.com` / `aa` | `assets/login_b_pa.png` | Login succeeds but the session is **account A** (header `aa@t.com`) |
| Row 4: `bb@t.com` / `bb` | `assets/login_b_pb.png` | `Incorrect password.`; account B cannot sign in |

Passwords are masked in the screenshots; they are the two-character values `aa` (account A)
and `bb` (account B) entered during registration.

No automation script is required; all steps are performed through the UI.

---

## Appendix — White-box corroboration (not the design basis of this test)

The observed behaviour is explained by `backend/app.py`, in the login handler
(`_login`, lines 634–641):

```python
row = db.execute(
    "SELECT * FROM accounts WHERE lower(username)=lower(?) OR lower(email)=lower(?) OR mobile=?",
    (ident, ident, ident),
).fetchone()
if row is None:
    raise ValueError("User not found.")
if row["password_hash"] != password_hash(password):
    raise ValueError("Incorrect password.")
```

The identifier is matched against three different columns with `OR`, but only **one** row
is taken (`fetchone()`, i.e. the lowest `rowid`). The password is then compared against
that single row, so every identifier that matches *any* column of the earlier account
resolves to that account and rejects all other accounts' passwords. The password check
is never attempted against the remaining candidate rows.

This is supporting evidence for the mechanism only. The test itself was derived from the
requirement text and can be executed without reading the source.
