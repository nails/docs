---
description: >-
  This module provides a user login/logout system as well as administrative
  utilities for managing a site's users.
---

# Auth

Auth owns users, groups, sessions, and everything about signing in and out. It provides password login, social sign-on, registration, password reset, and the admin screens for managing accounts.

## Ways to sign in

| Method | Notes |
| --- | --- |
| Password | Email or username, depending on `APP_NATIVE_LOGIN_USING` |
| [Passkeys](auth/passkeys.md) | WebAuthn; a fingerprint, face, screen lock, or security key instead of a password |
| Social sign-on | Configured per provider under Settings → Authentication |

For a second step after password login, see [Multi-Factor Auth](multi-factor-auth/). A user-verified [passkey](auth/passkeys.md) login satisfies that second step on its own.

## Protecting against brute force

Failed logins are counted per user. After five, the account is locked out for five minutes, and every attempt is delayed briefly to make guessing expensive. A captcha can be required on the login, registration, and password reset forms under Settings → Authentication.
