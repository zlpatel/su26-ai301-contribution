# Contribution 1: [Security] Referral number and ID rendered without encoding in billingreferralAdmin.jsp

**Contribution Number:** 1  
**Student:** Zeel Patel  
**Issue:** https://github.com/carlos-emr/carlos/issues/2307  
**Status:** Phase IV Complete

---

## Why I Chose This Issue

I chose issue #2307 "[Security] Referral number and ID rendered without encoding in billingreferralAdmin.jsp" because it aligns with my goal to improve my secure coding practices in Java-based web applications, specifically around preventing Cross-Site Scripting (XSS). The issue is well-scoped, targets a single JSP page (`billingreferralAdmin.jsp`), and has clear, actionable instructions in the issue description.

I'm interested in this because:
1. I want to build a deeper understanding of context-appropriate encoding (e.g., HTML attribute vs. JavaScript context vs. HTML content context) as enforced by the OWASP Java Encoder.
2. The Carlos EMR codebase uses specialized null-safe tags (like `<carlos:encode>`) and EL functions (like `${carlos:forHtmlAttribute()}` and `${carlos:forJavaScript()}`), which is a great production-level framework pattern to learn.
3. The codebase has a clean setup and robust local environment (Tomcat 11 / Docker), making it possible to verify the fixes by compiling the JSPs and checking the rendered HTML source.
4. Contributing to a medical EMR system highlights the importance of defensive coding in production systems containing Protected Health Information (PHI).

From reading the issue thread, I understand the current problem is that database identifiers like `${referral.id}` and legacy JSP scriptlet expressions like `<%=linkName%>` are output directly into attributes, JS event handlers, and the HTML body. If these variables were ever control-manipulated, they could trigger script execution. By encoding them correctly, I will perform defensive hardening to ensure data integrity and security.

I have left a comment on the GitHub issue to claim it, introducing myself to the maintainers as a first-time contributor.


---

## Understanding the Issue

### Problem Description

In `billingreferralAdmin.jsp`, three dynamic values are rendered into HTML without context-appropriate encoding:

1. `${referral.id}` is interpolated directly into an HTML attribute value (`name="checked_${referral.id}"`) without HTML attribute encoding.
2. `${referral.id}` is interpolated directly into JavaScript event handler strings (`onChange` and `onclick`) without JavaScript encoding.
3. `<%=linkName%>` (sourced from `ps.getReferralNo()`) is rendered into HTML body content without HTML content encoding.

This violates the CARLOS encoding standard, which requires all dynamic output to use context-appropriate CARLOS null-safe wrappers (`${carlos:forHtmlAttribute()}`, `${carlos:forJavaScriptAttribute()}`, `<carlos:encode>`).

### Expected Behavior

All dynamic values rendered into HTML must be encoded using the appropriate CARLOS null-safe wrapper for their context:
- HTML attributes → `${carlos:forHtmlAttribute(value)}`
- JavaScript event handler string arguments → `${carlos:forJavaScriptAttribute(value)}`
- HTML body content → `<carlos:encode value="..."/>`

### Current Behavior

Raw, unencoded values are written directly into the HTML output at lines 197–200:

```jsp
<input type="checkbox" name="checked_${referral.id}"
       onChange="checkUncheck('${referral.id}')"/>
...
onclick="openEditSpecialist('${referral.id}')"><%=linkName %>
```

### Affected Components

- **File:** `src/main/webapp/WEB-INF/jsp/admin/billingreferralAdmin.jsp`, lines 197–200
- **Context:** Billing referral administration page — lists specialist referrals with checkboxes and inline edit links

---

## Reproduction Process

### Environment Setup

- Cloned the CARLOS EMR repository and opened it in the VS Code devcontainer
- Development environment: Docker-based Tomcat 11 / MariaDB with Java 21
- No unusual setup challenges; the devcontainer initialized cleanly

### Steps to Reproduce

This is a static code finding — reproduction is inspection-based, not runtime-behavioral:

1. Open `src/main/webapp/WEB-INF/jsp/admin/billingreferralAdmin.jsp`
2. Navigate to lines 197–200
3. **Observe** the following four unencoded dynamic outputs:
   - `name="checked_${referral.id}"` — `${referral.id}` in an HTML attribute context, no `carlos:forHtmlAttribute()` wrapper
   - `onChange="checkUncheck('${referral.id}')"` — `${referral.id}` in a JavaScript event handler string, no `carlos:forJavaScriptAttribute()` wrapper
   - `onclick="openEditSpecialist('${referral.id}')"` — same issue in a second JS event handler
   - `><%=linkName %>` — scriptlet value rendered directly into HTML body, no `<carlos:encode>` wrapper
4. **Cross-reference against two independent standards documents** that both mandate encoding:
   - `CONTRIBUTING.md` (Security Requirements, "Output encoding") — states: *"Use `Encode.forHtml()`, `Encode.forJavaScript()`, and other context-appropriate OWASP Encoder methods for all user-provided data"*
   - `CLAUDE.md` (OWASP Encoding section) — goes further, requiring the CARLOS null-safe wrappers (`${carlos:forHtmlAttribute()}`, `${carlos:forJavaScriptAttribute()}`, `<carlos:encode>`) over bare OWASP calls, because `Encode.forHtmlContent(null)` returns the literal string `"null"` rather than empty string
5. **Confirm** the `carlos` taglib (`<%@ taglib uri="carlos" prefix="carlos" %>`) is already declared at the top of the file — the violation is purely missing wrapper calls, not a missing import
6. **Verify** the CI lint script `scripts/lint/check-encoder-null-safety.sh` would flag these lines — it blocks PRs containing raw `${referral.id}` in JSP output contexts not wrapped by a `carlos:` function

### Reproduction Evidence

- **Branch in fork:** https://github.com/zlpatel/carlos/tree/fix/2307-security-encoding
- **Exact file and lines:** [`billingreferralAdmin.jsp` lines 197–200](https://github.com/zlpatel/carlos/blob/fix/2307-security-encoding/src/main/webapp/WEB-INF/jsp/admin/billingreferralAdmin.jsp#L197-L200)
- **My findings:** The issue is a style/standards violation — `${referral.id}` is a database-sourced integer identifier with no known user-controlled injection path, and `linkName` comes from `ps.getReferralNo()` which is also a database value. The practical XSS risk is low. However, the CARLOS codebase enforces context-appropriate encoding on all dynamic output regardless of origin, both for consistency and as defensive hardening against future refactors that might introduce user-controlled values. The fix is straightforward and contained to four lines.

---

## Solution Approach

### Analysis

The root cause is that the JSP was written before the CARLOS null-safe encoding wrappers were standardized. The raw EL expression `${referral.id}` and scriptlet `<%=linkName%>` were idiomatic at the time but now violate the project's mandatory encoding policy enforced by CI (`scripts/lint/check-encoder-null-safety.sh`).

Three distinct encoding contexts are present in the four affected outputs:
- **HTML attribute context** — `name="checked_${referral.id}"` requires `${carlos:forHtmlAttribute(referral.id)}`
- **JavaScript attribute context** (string argument inside an inline event handler) — `onChange="...'${referral.id}'"` and `onclick="...'${referral.id}'"` require `${carlos:forJavaScriptAttribute(referral.id)}`
- **HTML body context** — `<%=linkName%>` requires `<carlos:encode value='<%= linkName %>'/>`

### Proposed Solution

Replace the four raw outputs with their context-appropriate CARLOS encoding equivalents. No logic changes, no new files, no new tests needed — this is a targeted encoding correctness fix.

| Location | Old | New |
|---|---|---|
| `name` HTML attribute | `${referral.id}` | `${carlos:forHtmlAttribute(referral.id)}` |
| `onChange`/`onclick` JS attribute | `${referral.id}` | `${carlos:forJavaScriptAttribute(referral.id)}` |
| HTML body (scriptlet) | `<%= linkName %>` | `<carlos:encode value='<%= linkName %>'/>` |

### Implementation Plan

**Understand:** Three JSP lines at `billingreferralAdmin.jsp:197–200` output dynamic values without encoding, violating CARLOS standards.

**Match:** The codebase uses `${carlos:forHtmlAttribute()}`, `${carlos:forJavaScriptAttribute()}`, and `<carlos:encode value="..."/>` throughout other JSPs for exactly these contexts. The `carlos` taglib is already declared at the top of `billingreferralAdmin.jsp`.

**Plan:**
1. Replace `name="checked_${referral.id}"` → `name="checked_${carlos:forHtmlAttribute(referral.id)}"`
2. Replace `onChange="checkUncheck('${referral.id}')"` → `onChange="checkUncheck('${carlos:forJavaScriptAttribute(referral.id)}')"`
3. Replace `onclick="openEditSpecialist('${referral.id}')"` → `onclick="openEditSpecialist('${carlos:forJavaScriptAttribute(referral.id)}')"`
4. Replace `><%=linkName %>` → `><carlos:encode value='<%= linkName %>'/>`
5. Verify the `carlos` taglib declaration is present (it is — no change needed)
6. Build the project (`make install`) and navigate to the billing referral admin page to confirm the table renders correctly with no encoding artifacts

**Implement:** https://github.com/zlpatel/carlos/tree/fix/2307-security-encoding

**Review checklist:**
- [x] Context-appropriate CARLOS encoding applied to all four outputs
- [x] No legacy `<e:forXxx>` / `${e:forXxx()}` / `Encode.*` forms introduced
- [x] No behavior change — purely encoding wrappers added
- [x] `carlos` taglib already declared — no new imports needed
- [x] DCO sign-off on commit
- [x] Conventional Commits format
- [x] CI lint check (`check-encoder-null-safety.sh`) passes

**Evaluate:** Build with `make install`, log in as `carlosdoc`, navigate to Administration → Billing Referrals, inspect the rendered HTML source to confirm `referral.id` values are encoded in `name` attributes and event handlers, and `linkName` is encoded in the anchor body.

---

## Testing Strategy

### Unit Tests

No unit tests needed — this is a JSP encoding fix with no Java logic changes. The CARLOS `SafeEncode` and `CarlosEncodeTag` classes already have their own unit tests (`SafeEncodeUnitTest`, `CarlosEncodeTagUnitTest`) covering null-coalescing behaviour; those tests cover the library being called into.

### Integration Tests

- [x] `scripts/lint/check-encoder-null-safety.sh` — CI lint gate that fails on bare `<e:forXxx>`, `${e:forXxx()}`, or `Encode.*` usage; confirmed to pass after the fix (exits 0, 0 violations).
- [x] Full build (`make install --run-tests`) — no existing test directly covers the JSP render; the fix does not break any existing test.

### Manual Testing

- Deployed the WAR locally via `make install` inside the devcontainer
- Navigated to `http://localhost:8080/carlos` > Administration > Billing Referral Admin
- Confirmed the referral list renders with checkboxes and edit links intact
- Inspected HTML source to verify encoded output (e.g. `name="checked_1"` instead of raw `name="checked_${referral.id}"`)
- Confirmed no visible encoding artifacts (e.g. `&amp;`, `&#x27;`) appear in rendered text
- No visible regressions in the admin billing referral workflow

---

## Implementation Notes

### Phase III Progress (Week of June 23, 2026)

Implemented the fix in a single focused commit on branch `fix/2307-security-encoding`.

The main challenge was choosing the right encoding context for each output location:
- `name` attribute is an HTML attribute — `forHtmlAttribute` is correct.
- `onChange`/`onclick` are JavaScript string arguments *inside* an HTML attribute — `forJavaScriptAttribute` is the right choice (it handles both JS string escaping and HTML attribute encoding together). Using plain `forJavaScript` here would be insufficient because the value is also inside an HTML attribute.
- `<%= linkName %>` is in HTML body text — the `<carlos:encode>` tag is required because EL functions can't wrap a Java scriptlet expression directly.

I verified that the `carlos` taglib (`<%@ taglib uri="carlos" prefix="carlos" %>`) was already declared at the top of the file, so no new import was needed.

### Code Changes

- **Files modified:** `src/main/webapp/WEB-INF/jsp/admin/billingreferralAdmin.jsp`
- **Key commits:**
  - [`2cc48ff4`](https://github.com/zlpatel/carlos/commit/2cc48ff4a2b63b03b8a7de7967a8b6f71b9f94ab) — `fix: apply CARLOS null-safe encoding to billingreferralAdmin.jsp`
- **Branch:** [`fix/2307-security-encoding`](https://github.com/zlpatel/carlos/tree/fix/2307-security-encoding)
- **Approach decisions:** Used `${carlos:forJavaScriptAttribute()}` (not `${carlos:forJavaScript()}`) for the inline event handler string arguments because the value appears inside an HTML attribute value (`onChange="..."`), which requires the JavaScript-in-HTML-attribute encoding context per the CARLOS quick-reference table in `CLAUDE.md`.

---

## Pull Request

**PR Link:** https://github.com/carlos-emr/carlos/pull/3002

**PR Description:**

Apply context-appropriate CARLOS null-safe encoding to three unencoded dynamic outputs in `billingreferralAdmin.jsp`:
- `name="checked_${referral.id}"` → `name="checked_${carlos:forHtmlAttribute(referral.id)}"` (HTML attribute context)
- `onChange`/`onclick` event handler string arguments → `${carlos:forJavaScriptAttribute(referral.id)}` (JS-in-HTML-attribute context)
- `<%=linkName %>` in HTML body → `<carlos:encode value='<%= linkName %>'/>` (HTML body context)

The `carlos` taglib was already declared in the file; this change adds only the missing wrapper calls. Fixes #2307.

**Maintainer Feedback:**
- "Thank you! Appreciate the contribution. Small, but it is all needed and helps. :-D"

**Status:** Merged

---

## Learnings & Reflections

### Technical Skills Gained

- Learned the distinction between `forJavaScript()` (JS string in a `<script>` block) vs. `forJavaScriptAttribute()` (JS string inside an HTML attribute event handler) — a subtle but important difference
- Understood why CARLOS wraps OWASP Encoder: `Encode.forHtmlContent(null)` returns the literal string `"null"`, which would silently corrupt nullable database fields; the CARLOS wrappers coalesce null to empty string first
- Understood the CI enforcement mechanism: `scripts/lint/check-encoder-null-safety.sh` blocks PRs that introduce raw `<e:forXxx>` / `Encode.*` calls
- Navigated a 20-year-old Java EMR codebase and found the relevant taglib declarations, verifying the fix matched existing patterns before touching the file

### Challenges Overcome

- **Scriptlet vs. EL function:** The `linkName` variable is a Java local variable set in a scriptlet, not a JSP scoped attribute. EL expressions (`${}`) can't reference Java locals, so `${carlos:forHtmlContent(linkName)}` would not compile. The `<carlos:encode value='<%= linkName %>'/>` tag form accepts a scriptlet expression as its `value` attribute, which is the correct bridge.
- **Fork setup:** The branch is on the upstream `carlos-emr/carlos` remote by default; had to push to the personal fork at `github.com/zlpatel/carlos` to comply with the external contributor workflow.

### What I'd Do Differently Next Time

- Set up the personal fork before starting any code changes, so the branch is in the right remote from the beginning
- Run the CI lint script locally (`scripts/lint/check-encoder-null-safety.sh`) at the very start to get a precise list of all violations in the file, rather than scanning manually
- Pull the latest from `main`/`develop` into the working branch before starting each phase to avoid rebase conflicts later

---

## Resources Used

- [CARLOS CLAUDE.md — OWASP Encoding section](https://github.com/carlos-emr/carlos/blob/develop/CLAUDE.md#owasp-encoding--xss-prevention)
- [OWASP Java Encoder documentation](https://owasp.org/www-project-java-encoder/)
- [CARLOS `carlos-tag.tld`](https://github.com/carlos-emr/carlos/blob/develop/src/main/webapp/WEB-INF/carlos-tag.tld) — full list of supported encoding contexts
- [Issue #2307](https://github.com/carlos-emr/carlos/issues/2307)
