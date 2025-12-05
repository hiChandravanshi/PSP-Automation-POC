# For Automation Monorepo
```
automation-workspace/
├── apps/
│   ├── api-tests/
│   │   ├── project.json
│   │   ├── package.json
│   │   ├── Dockerfile
│   │   └── src/
│   │       ├── tests/
│   │       ├── fixtures/
│   │       └── configs/
│   │
│   ├── web-tests/
│   │   ├── project.json
│   │   ├── package.json
│   │   ├── Dockerfile
│   │   └── src/
│   │       ├── tests/
│   │       ├── pages/
│   │       ├── fixtures/
│   │       └── configs/
│   │
│   ├── mobile-tests/
│   │   ├── package.json         # ✅ only ONE package.json for all mobile apps
│   │   ├── project.json         # root project definition
│   │   ├── wdio.base.config.js
│   │   │
│   │   ├── app1/                # app-specific tests & config
│   │   │   ├── project.json     # Nx project for app1
│   │   │   ├── wdio.conf.js
│   │   │   └── src/
│   │   │       ├── tests/
│   │   │       ├── pages/
│   │   │       ├── fixtures/
│   │   │       └── configs/
│   │   │
│   │   ├── app2/
│   │   │   ├── project.json     # Nx project for app2
│   │   │   ├── wdio.conf.js
│   │   │   └── src/
│   │   │       ├── tests/
│   │   │       ├── pages/
│   │   │       ├── fixtures/
│   │   │       └── configs/
│
├── libs/
│   ├── wdio-helpers/           # common library for mobile tests
│   ├── playwright-helpers/     # common library for web tests    
│   ├── reporting/              # common library for all reporting tools and API
│   ├── cms-sdk/                # cms test user api library
│   ├── test-user-db/           # test user database helper library
│   ├── utils/                  # common utility library for all projects
│
├── tools/
│   ├── setup/
│   ├── db/
│   └── testdata/
│
├── .azure-pipelines/
│   ├── templates/
│   ├── pipelines/
│   └── env/
│
├── config/
│   ├── eslint/
│   ├── prettier/
│   ├── commitlint/
│   ├── husky/
│   └── lint-staged/
│
├── .husky/
├── .github/
├── nx.json
├── package.json                 # root package.json (for common dependency)
├── tsconfig.base.json
└── yarn.lock / package.lock
```