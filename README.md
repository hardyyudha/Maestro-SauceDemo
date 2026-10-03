# MaestroStarter

Automated login tests using Maestro, organized by platform.

```text
test/user_login/
|-- web/      Implemented browser login flows
|-- android/  Reserved for Android app flows
`-- ios/      Reserved for iOS app flows
```

The current suite covers the web platform. Android and iOS folders are placeholders until the native app builds or app identifiers are available.

## Prerequisites

- Node.js and npm
- Maestro CLI installed and available on your `PATH`
- Google Chrome installed for web flows
- Android emulator/device and app build for Android flows
- iOS simulator and app build for iOS flows

The project includes a ChromeDriver npm dependency. Run `npm install` to install project dependencies before running the tests.

## Setup

Create a `.env` file in the project root with the values for your test environment:

```env
PROD_URL=https://your-test-site.example
VALID_USERNAME=your_valid_username
UNIVERSAL_PASSWORD=your_test_password
```

Keep `.env` private. It is excluded from Git by `.gitignore`.

## Run the tests

From the project root, run:

```bash
npm install
npm run maestro
```

The npm script loads `.env` and runs `test/user_login/web/user_login.yaml`. That orchestrator calls each web login flow sequentially in the order listed below:

1. `01.login_valid_credential.yaml` - valid credentials
2. `02.login_invalid_username.yaml` - unknown username
3. `03.login_invalid_password.yaml` - incorrect password
4. `04.login_empty_username.yaml` - missing username
5. `05.login_empty_password.yaml` - missing password
6. `06.login_empty_field.yaml` - both fields empty

## Add a test flow

1. Add a numbered Maestro YAML file in `test/user_login/web/`.
2. Add a `runFlow` entry for it in `test/user_login/web/user_login.yaml` at the position where it should run.
3. Run `npm run maestro` to execute the suite.

For example:

```yaml
- runFlow: "07.login_locked_user.yaml"
```

Flow order is controlled by the `runFlow` entries in the orchestrator, not by the npm script or filename sorting.
