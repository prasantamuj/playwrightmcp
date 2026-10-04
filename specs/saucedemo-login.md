# Test Plan: Sauce Demo Login Flow

**Target:** https://www.saucedemo.com/
**Seed:** tests/seed.spec.ts
**Date:** 2026-10-04

## Overview
The Sauce Demo login page provides authentication for access to an inventory system. This plan covers successful login, account lockout, and form validation errors to ensure the authentication flow works as expected for both valid and invalid scenarios.

## Preconditions
- Application is accessible at https://www.saucedemo.com/
- Browser is at the login page (username and password fields visible)
- All test users and password are available from tests/data/users.json
- No prior authentication session exists (page redirects unauthenticated users to login)

## Scenarios

### Scenario 1.1 — Standard User Successful Login
- **Priority:** P0
- **Tags:** @smoke
- **Preconditions:** Browser at login page; standard_user credentials available
- **Steps:**
  1. Enter username "standard_user" in the Username field — expected: text appears in field
  2. Enter password "secret_sauce" in the Password field — expected: password is masked in field
  3. Click the Login button — expected: page navigates away from login form
- **Assertions:**
  - Page URL is https://www.saucedemo.com/inventory.html
  - Page displays product inventory (products like "Sauce Labs Backpack", "Sauce Labs Bike Light" are visible)
  - An "Add to cart" button appears for each product
  - Header contains "Swag Labs" branding and cart button
- **Edge cases considered:**
  - User may have bookmarked the inventory URL directly; should be redirected to login if not authenticated
  - Session should persist across page refreshes
  - Login should not require special URL parameters or fragments

### Scenario 1.2 — Locked Out User Account
- **Priority:** P1
- **Tags:** @regression
- **Preconditions:** Browser at login page; locked_out_user credentials available
- **Steps:**
  1. Enter username "locked_out_user" in the Username field — expected: text appears in field
  2. Enter password "secret_sauce" in the Password field — expected: password is masked
  3. Click the Login button — expected: login attempt fails
- **Assertions:**
  - Page remains at https://www.saucedemo.com/ (login page URL)
  - An alert error message appears with text: "Epic sadface: Sorry, this user has been locked out."
  - Username and password fields retain their values
  - A "Dismiss error" button is present in the alert
- **Edge cases considered:**
  - Alert should be dismissible without affecting form state
  - Multiple login attempts with locked account should display the same message

### Scenario 1.3 — Empty Username Submission
- **Priority:** P1
- **Tags:** @regression
- **Preconditions:** Browser at login page
- **Steps:**
  1. Leave the Username field empty
  2. Enter any password (e.g., "secret_sauce") in the Password field — expected: text appears
  3. Click the Login button — expected: validation error appears
- **Assertions:**
  - Page remains at https://www.saucedemo.com/ (login page URL)
  - An alert error message appears with text: "Epic sadface: Username is required"
  - Password field retains its entered value
  - Form is ready for retry without page reload
- **Edge cases considered:**
  - Username field may be pre-focused on page load; user may attempt submission by tabbing past it
  - Validation should trigger on button click, not on field blur
  - Error should clear when user begins typing in username field

### Scenario 1.4 — Empty Password Submission
- **Priority:** P1
- **Tags:** @regression
- **Preconditions:** Browser at login page
- **Steps:**
  1. Enter username "standard_user" in the Username field — expected: text appears in field
  2. Leave the Password field empty
  3. Click the Login button — expected: validation error appears
- **Assertions:**
  - Page remains at https://www.saucedemo.com/ (login page URL)
  - An alert error message appears with text: "Epic sadface: Password is required"
  - Username field retains its entered value ("standard_user")
  - Form is ready for retry without page reload
- **Edge cases considered:**
  - Validation should trigger even if username is valid
  - User may attempt submission by pressing Enter in the password field
  - Error should clear when user begins typing in password field

### Scenario 1.5 — Invalid Credentials (Username and Password Mismatch)
- **Priority:** P1
- **Tags:** @regression
- **Preconditions:** Browser at login page
- **Steps:**
  1. Enter an invalid username (e.g., "nonexistent_user") in the Username field — expected: text appears in field
  2. Enter an invalid password (e.g., "wrongpassword") in the Password field — expected: text appears
  3. Click the Login button — expected: login attempt fails
- **Assertions:**
  - Page remains at https://www.saucedemo.com/ (login page URL)
  - An alert error message appears with text: "Epic sadface: Username and password do not match any user in this service"
  - Both Username and Password fields retain their entered values
  - Form is ready for retry without page reload
- **Edge cases considered:**
  - Message should be generic to avoid username enumeration attacks
  - Valid username with wrong password should show same error as invalid username
  - Error message should not disclose whether username exists or not

## Not covered (and why)
- **Valid username with incorrect password:** Same error message as invalid credentials (Scenario 1.5); testing both would be redundant
- **SQL injection or XSS in login fields:** Out of scope for functional login testing; security testing would be handled separately
- **Account recovery or password reset flows:** Not part of the login authentication flow
- **Multi-factor authentication (MFA):** Not implemented in this application
- **Rate limiting or brute force protection:** Would require multiple rapid submissions; better tested with performance/security testing tools
- **Browser autofill behavior:** Depends on browser configuration; not relevant to login functionality testing
- **Keyboard-only navigation:** Implicitly covered by form interaction (Tab, Enter); could be explicit if accessibility testing is prioritized
- **Remember me / persistent login:** Not offered by this login form
- **Login with email vs username:** Application only accepts username format
