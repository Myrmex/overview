---
layout: page
title: Cadmus Development
subtitle: History - Gravatar
---

## Gravatar Rafactoring

📆 Date: 2024-07-16.

This minor refactoring was applied to remove dependencies from the NPM package `gravatar`, which was not ESM.

Gravatar code has been refactored accordingly, and auth library versions have bumped to 5.1.x. To migrate:

(1) replace packages:

```bash
npm uninstall gravatar --force
npm i ts-md5
```

(2) typically Gravatar is used only in the app root component. Remove function `getGravatarUrl` from it and replace it with a simpler call to the gravatar pipe, using the right size (typically 32):

```html
<img
  alt="avatar"
  [src]="user.email | gravatar : 32"
  [alt]="user.userName"
/>
```

Additionally, note that:

- the Gravatar service now provides more options. The default size is 32 and no longer 80, which was assumed when you did not specify a size in calling the service method.
- the Gravatar pipe can be used by just specifying the size as before; just remember that if you don't specify it, it will default to 32 and not to 80.
