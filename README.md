# Contribution 1: [Security] Referral number and ID rendered without encoding in billingreferralAdmin.jsp

**Contribution Number:** 1  
**Student:** Zeel Patel  
**Issue:** https://github.com/carlos-emr/carlos/issues/2307  
**Status:** Phase I Complete

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

[In your own words, what's broken or missing?]

### Expected Behavior

[What should happen?]

### Current Behavior

[What actually happens?]

### Affected Components

[Which parts of the codebase are involved?]

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
