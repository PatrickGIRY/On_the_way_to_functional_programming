# Setup of JS project

2025-08-03

```sh
cd attendees-finder-js

npm init

package name: (attendees-finder) qttendees-finder-js
version: (1.0.0) 
description: Kata to find attendees by first name
entry point: (index.js) attendees-finder.js
test command: vitest --coverage
git repository: https://github.com/PatrickGIRY/attendees-finder-js
keywords: Functional programming
author: Patrick GIRY
license: (ISC) 

npm install vitest --save-dev
```

- Upgrade `package.json`

```json
{
  "name": "qttendees-finder-js",
  "version": "1.0.0",
  "description": "Kata to find attendees by first name",
  "main": "attendees-finder.js",
  "scripts": {
    "test": "vitest --coverage"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/PatrickGIRY/attendees-finder-js.git"
  },
  "keywords": [
    "Functional",
    "programming"
  ],
  "author": "Patrick GIRY",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/PatrickGIRY/attendees-finder-js/issues"
  },
  "homepage": "https://github.com/PatrickGIRY/attendees-finder-js#readme",
  "devDependencies": {
    "@vitest/coverage-v8": "^3.2.4",
    "vitest": "^3.2.4"
  }
}
```
