
some say £800 - does that sound about right or too much or too little - they said they will handle it end to end incl all google policies

###### ChatGPT said:

£800 for **Google Play end-to-end handling** can be _reasonable_, _high_, or _overkill_ depending on **what “end to end” actually includes**.

Here’s the honest breakdown from someone who has seen dozens of these offers:

---

# ✅ **When £800 is a fair price**

£800 makes sense **only if they are doing _all_ of this**:

### ✔ 1. **Full code review**

- Fixing policy violations
    
- Rewriting parts of the app if needed
    
- Implementing privacy policy dialogs
    
- Adding MIT license visibility
    
- Updating dependencies (SDK 34, AndroidX, etc.)
    

### ✔ 2. **Building & signing the app**

- Setting up keystore
    
- Automating builds
    
- Ensuring it passes pre-launch tests
    

### ✔ 3. **Creating all Play Store assets**

- Feature graphic
    
- Screenshots
    
- App icon (512×512)
    
- Privacy policy webpage
    
- Store listing text
    

### ✔ 4. **Handling Google Play’s annoying policy reviews**

- Submitting multiple times
    
- Responding to rejections
    
- Fixing issues until it’s approved
    
- Helping you with data safety forms
    
- Handling the new SDK targeting requirements
    
- Coping with “missing declarations”
    

This is **time-intensive**, and a legit developer in Canada, UK, or US would charge **£500–£2,000** for this exact service.

So **if the offer includes all of that** → **£800 is okay**.

---

# ❌ **When £800 is a rip-off**

If they only plan to:

- upload your APK
    
- fill out the Google Play form
    
- write a basic privacy policy
    
- do one submission
    

→ Then £800 is **WAY too much**  
You should be paying **£50–£200** for that.

Most Fiverr “I will publish your app” gigs charge **£10–£60** because they assume:

- Your app is already compliant
    
- No code fixes
    
- One submission
    
- You provide all assets
    

£800 is only justified when it becomes real engineering work.

---

# 🎯 **My recommendation**

Ask them ONE question:

**“Does your £800 include unlimited resubmissions until Google approves it, including code fixes and policy compliance work?”**

If they say **YES**, then £800 is fair.  
If they say **NO**, then they’re selling a £50–£100 service for £800.

---

If you want, I can:  
✔ Write a message to ask them the right questions  
✔ Compare their claim vs market value  
✔ Tell you whether their specific offer is legit based on their profile and wording

Just tell me which freelancer quoted the £800.


![[Brief-management-Upload-Open-Source-App-to-Play-Store-11-21-2025_12_19_PM.png]]


# Google Play Submission Failure Modes — Comprehensive Reference

This document lists **all major known reasons** Google Play may reject, suspend, block, or prevent publishing of an Android app — including technical issues, metadata issues, and policy violations.

---

# 1. Technical Failures

## 1.1 SDK & Build Failures

- App not targeting latest required SDK (e.g., not targeting SDK 34+)
    
- Missing 64‑bit support (Google Play requires arm64-v8a)
    
- Using deprecated build tools or libraries
    
- Missing compatible ABI (arm64 only apps rejected if they claim more support)
    
- Version code not incremented
    
- AAB not signed with correct keystore
    
- Corrupted AAB/App Bundle
    
- App uses restricted or internal Android APIs
    

## 1.2 Pre‑Launch Report Crashes / ANRs

- App crashes immediately on startup
    
- ANR triggered during automated tests
    
- StrictMode violations (network on main thread)
    
- Missing runtime permission checks
    
- Unhandled exceptions in WebView or JS interfaces
    

## 1.3 Compatibility Failures

- Missing required screen sizes
    
- Incorrect manifest features (e.g., camera required when not needed)
    
- Declared hardware features block Play Store distribution
    
- App doesn’t support 64‑bit devices but claims to
    

---

# 2. Policy Failures

## 2.1 Privacy Policy

- Privacy policy missing
    
- Privacy policy inaccessible (404 or private)
    
- Privacy policy incomplete (missing data types)
    
- Privacy policy not matching actual app behaviour
    

## 2.2 Data Safety Form

- Wrong data categories selected
    
- App collects data but form says "no data collected"
    
- Using WebView that loads external pages without disclosure
    
- SDKs collecting data without correct declarations
    

## 2.3 Permissions & Sensitive APIs

- Background location without core justification
    
- Requesting SMS/Contacts/Call Log
    
- Installing unknown apps (REQUEST_INSTALL_PACKAGES)
    
- System alert window (SYSTEM_ALERT_WINDOW)
    
- All-File access (MANAGE_EXTERNAL_STORAGE)
    
- Biometric or device admin permission misdeclared
    

## 2.4 User Data & Access

- Apps asking for sign-in without privacy documentation
    
- Apps requesting sensitive access for non-core features
    

## 2.5 Deceptive Behaviour

- Misleading claims
    
- Fake functionality
    
- App pretending to be another brand (impersonation)
    
- Misuse of logos or copyrighted images
    

## 2.6 Spam & Low Quality

- Repetitive content
    
- WebView wrappers with no added functionality
    
- Copy-paste apps ("template apps")
    
- Broken navigation or UI
    

## 2.7 Malware / Security

- Obfuscated behaviour that looks suspicious
    
- Dynamic code loading without justification
    
- SDKs flagged as unsafe
    
- Unencrypted communication of sensitive data
    

---

# 3. Metadata Failures

## 3.1 Listing Content

- Too short or too long description
    
- Keyword stuffing
    
- Unsupported formatting (emoji spam, ALL CAPS)
    
- Misleading feature lists
    

## 3.2 Screenshot & Graphic Requirements

- Wrong screenshot resolution
    
- Screenshots not representing actual app UI
    
- Missing Feature Graphic
    
- App icon not compliant with adaptive icon rules
    
- Upscaled or blurry images
    

## 3.3 App Category & Rating

- Incorrect category selection
    
- Incorrect content rating questionnaire
    
- Missing "Made For Families" disclosures (if targeting kids)
    

---

# 4. Asset & Bundle Issues

## 4.1 App Bundle Errors

- Missing base module
    
- Invalid dynamic feature configuration
    
- Missing locales
    

## 4.2 Signing Problems

- Wrong keystore used
    
- App signing key mismatch with previous upload
    
- Expired signing certificate
    

---

# 5. Store Listing Violations

## 5.1 Inappropriate Content

- Sexual content
    
- Violent or graphic material
    
- Images depicting drugs, weapons
    

## 5.2 Misleading or Harmful Claims

- "Guaranteed to make money"
    
- Medical/health cures
    
- Political misinformation
    

## 5.3 Ads Violations

- Ads shown without user disclosure
    
- Ads appearing immediately on opening the app
    
- Ads inappropriate for children
    

---

# 6. Device & Feature Issues

## 6.1 Play‑Protect Issues

- Unsafe permissions flagged
    
- Malware-like behaviours
    
- Sensitive API usage not declared
    

## 6.2 Device Admin

- Using Device Administrator without correct disclosures
    
- Incorrectly declaring enterprise features
    

---

# 7. Submission Workflow Failures

## 7.1 Missing Required Elements

- Feature graphic missing
    
- Short description missing
    
- App icon incorrect format
    
- No contact email
    
- No support URL
    

## 7.2 App Review Questions

- "App purpose description" too vague
    
- "How does your app handle user data?" incomplete
    

## 7.3 Rejection Loop Issues

- Fixing wrong thing repeatedly
    
- Not understanding policy reason text
    
- Google auto-reject from previous flags
    

---

# 8. WebView-Specific Failures

## 8.1 Content Policy Violations

- WebView loads content banned by Play policies
    
- WebView loads login forms without disclosures
    

## 8.2 Security

- Mixed HTTP/HTTPS content
    
- JavaScript interface exposing private APIs
    

---

# 9. Monetization / Payments

## 9.1 In-App Purchases

- Using external payment systems improperly
    
- Selling digital goods without Google Play Billing
    

## 9.2 Ads

- Incomplete ad disclosures
    
- Child-directed app serving personalised ads
    

---

# 10. Kids / Family Program Failures

## 10.1 Child Safety Requirements

- Showing adult ads
    
- Linking to external sites without parental gate
    
- Missing teacher/parent documentation
    

---

# 11. Ongoing Compliance Failures

- SDKs updated with new policy requirements
    
- Multiple apps flagged linking to same privacy policy domain
    
- App behaviour changed after update
    

---

# 12. Developer Account Issues

- Account verification pending
    
- Invalid payment profile
    
- Play Console warnings unresolved
    

---

# 13. Geo, Legal & Content Restrictions

- Unsupported feature in certain regions (e.g., VPN apps)
    
- Export compliance issues
    
- Crypto regulation violations
    

---

# 14. Rare / Edge-Case Failures

- AI-generated content requiring disclosures
    
- Apps that mimic OS UI elements
    
- Apps without a meaningful UI (launcher shortcuts only)
    
- Apps with background services but no foreground activity
    

---

# End

This document can be expanded further into checklists or automated linting rules on request.

end
