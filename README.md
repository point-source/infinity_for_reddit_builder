# Infinity For Reddit Builder

This repository contains GitHub Actions workflows to automatically build and release the [Infinity For Reddit](https://github.com/Docile-Alligator/Infinity-For-Reddit) app with custom API credentials.

## Features

- Automatically monitors Infinity For Reddit for new releases
- Builds the app for arm64-v8a architecture
- Replaces API credentials with custom values
- Customizes the app's user agent with your Reddit username
- Creates releases with built APKs
- Maintains the last 3 releases
- Supports manual triggering
- Validates required secrets before building
- Automatically signs APKs
- Compatible with [Obtainium](https://github.com/ImranR98/Obtainium) for automatic updates on your device

## GitHub Setup

1. Fork this repository to your GitHub account

   - **Important**: Make your fork private to keep your releases and API credentials secure
   - To make your fork private, go to your repository's Settings > General > Danger Zone > Change repository visibility

2. Add the following secrets to your repository:

   - Go to your repository's Settings > Secrets and variables > Actions
   - Add these secrets:
     - `REDDIT_API_CLIENT_ID`: Your Reddit API client ID (Required)
     - `REDDIT_USERNAME`: Your Reddit username without the /u/ prefix (Required)
     - `REDDIT_API_REDIRECT_URI`: Your Reddit API redirect URI (Optional, defaults to http://127.0.0.1)
     - `KEYSTORE_BASE64`: Base64-encoded keystore file (Required for app signing)
     - `KEYSTORE_PASSWORD`: Password for the keystore (Optional, defaults to "Infinity")
     - `KEY_ALIAS`: Alias of the key to use for signing (Optional, defaults to "Infinity")
     - `KEY_PASSWORD`: Password for the key (Optional, defaults to "Infinity")

3. The workflow will automatically:
   - Check for new releases every day at 6am
   - Validate required secrets
   - Build the app when a new release is found
   - Create a release with the built APK
   - Clean up old releases (keeping only the last 3)

## Keystore Setup

### Creating a Keystore

If you do not have a [keystore file such as this one](https://github.com/TanukiAI/Infinity-keystore/blob/main/Infinity.jks), you can create one using keytool:

1. Generate a keystore file using keytool:

   ```bash
   keytool -genkey -v -keystore infinity.keystore -alias infinity -keyalg RSA -keysize 2048 -validity 10000
   ```

   When prompted, enter the following information:

   - Keystore password: `Infinity`
   - Key password: `Infinity`
   - Alias: `Infinity`
   - Fill in the other details as needed

### Converting Keystore to Base64

To convert your JKS keystore file to base64 format for GitHub Secrets:

**On macOS/Linux:**

```bash
base64 -i infinity.keystore -o keystore_base64.txt
```

**On Windows (PowerShell):**

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("infinity.keystore")) | Out-File -FilePath keystore_base64.txt
```

**On Windows (Command Prompt):**

```cmd
certutil -encode infinity.keystore keystore_base64.txt
```

After running the appropriate command, open the `keystore_base64.txt` file and copy its contents. This is what you'll use as the value for the `KEYSTORE_BASE64` secret in your GitHub repository.

### Using Default Keystore Credentials

If you're using the default keystore credentials (password: "Infinity", alias: "Infinity", key password: "Infinity"), you don't need to set the following secrets:

- `KEYSTORE_PASSWORD`
- `KEY_ALIAS`
- `KEY_PASSWORD`

The workflow will automatically use these default values if the secrets are not provided.

## Automatic Updates with Obtainium

[Obtainium](https://github.com/ImranR98/Obtainium) is an Android app that allows you to install and update apps directly from their GitHub releases pages. You can use it to automatically receive notifications when new releases are available in your fork.

### Setting up Obtainium with your private fork

1. Install Obtainium from [F-Droid](https://f-droid.org/packages/dev.imranr.obtainium/) or [GitHub](https://github.com/ImranR98/Obtainium/releases)

2. Create a GitHub Personal Access Token:

   - Go to GitHub > Settings > Developer settings > Personal access tokens > Fine-grained tokens
   - Click "Generate new token"
   - Give your token a descriptive name (e.g., "Obtainium for Infinity")
   - Set the expiration as needed
   - Under "Repository access", select "Only select repositories" and choose your private fork
   - Under "Permissions", expand "Repository permissions"
   - Set "Contents" to "Read-only" (this is all Obtainium needs)
   - Click "Generate token"
   - Copy the token (you won't be able to see it again)

3. Add your fork to Obtainium:

   - Open Obtainium and tap the "+" button
   - Select "GitHub" as the source
   - Enter your fork's repository name (e.g., `yourusername/infinity_for_reddit_builder`)
   - Enter your GitHub username
   - Paste your Personal Access Token
   - Tap "Add App"

4. Configure update settings:
   - Tap on the app in Obtainium
   - Enable "Auto-update" if you want automatic updates
   - Set your preferred notification settings

Now Obtainium will check your private fork for new releases and notify you when they're available. You can install updates directly from the app.

## Local Testing

### Prerequisites

1. Install required tools:

   ```bash
   # Install GitHub CLI
   brew install gh  # macOS
   # or
   sudo apt install gh  # Ubuntu/Debian

   # Install JDK 17
   brew install openjdk@17  # macOS
   # or
   sudo apt install openjdk-17-jdk  # Ubuntu/Debian

   # Install Android SDK and build tools
   brew install android-commandlinetools  # macOS
   # or
   sudo apt install android-sdk  # Ubuntu/Debian
   ```

2. Set up environment variables:

   ```bash
   # Required variables
   export REDDIT_API_CLIENT_ID="your_client_id_here"
   export REDDIT_USERNAME="your_reddit_username"  # without the /u/ prefix

   # Optional variable (defaults to http://127.0.0.1 if not set)
   export REDDIT_API_REDIRECT_URI="http://127.0.0.1"
   ```

   Or copy the `.env.example` file to `.env` and edit it:

   ```bash
   cp .env.example .env
   # Then edit .env with your values
   ```

### Running the Workflow Locally

1. Clone this repository:

   ```bash
   git clone https://github.com/yourusername/infinity_builder.git
   cd infinity_builder
   ```

2. Run the workflow:

   ```bash
   gh workflow run build_infinity.yml
   ```

3. Monitor the workflow:
   ```bash
   gh run watch $(gh run list --workflow=build_infinity.yml --limit=1 --json databaseId --jq '.[0].databaseId')
   ```

### Manual Build Process

If you want to build without using GitHub Actions:

1. Clone Infinity For Reddit:

   ```bash
   git clone https://github.com/Docile-Alligator/Infinity-For-Reddit.git
   cd Infinity-For-Reddit
   ```

2. Replace API credentials and user agent:

   ```bash
   # Replace Reddit API client ID
   find . -type f -name "*.java" -exec sed -i "s/your_client_id_here/$REDDIT_API_CLIENT_ID/g" {} +

   # Replace redirect URI (uses default if not set)
   REDIRECT_URI=${REDDIT_API_REDIRECT_URI:-"http://127.0.0.1"}
   find . -type f -name "*.java" -exec sed -i "s|your_redirect_uri_here|$REDIRECT_URI|g" {} +

   # Replace user agent
   USER_AGENT="android:personal-app:0.0.1 (by /u/$REDDIT_USERNAME)"
   find . -type f -name "*.java" -exec sed -i "s|android:personal-app:0.0.1 (by /u/your_reddit_username)|$USER_AGENT|g" {} +
   ```

3. Update lint baseline:

   ```bash
   ./gradlew updateLintBaseline
   ```

4. Configure signing in build.gradle:

   Add the following to `app/build.gradle` inside the `android` block:

   ```gradle
   signingConfigs {
       release {
           storeFile file("infinity.keystore")
           storePassword "Infinity"
           keyAlias "Infinity"
           keyPassword "Infinity"
       }
   }
   ```

   And inside the `buildTypes` block:

   ```gradle
   release {
       signingConfig signingConfigs.release
   }
   ```

5. Build the APK:

   ```bash
   ./gradlew assembleRelease
   ```

The signed APK will be available at `./app/build/outputs/apk/release/app-release.apk`.

## Troubleshooting

### Common Issues

1. **Build fails with Gradle errors**

   - Ensure you have JDK 17 installed and JAVA_HOME is set correctly
   - Try running `./gradlew clean` before building
   - Check that the signing configuration in build.gradle is correct

2. **API credentials not being replaced**

   - Check if the placeholder text matches exactly in the source files
   - Verify your environment variables are set correctly
   - Make sure your Reddit username doesn't contain special characters
   - Ensure required secrets (REDDIT_API_CLIENT_ID and REDDIT_USERNAME) are set
   - Verify the API credentials format (client ID: 20-30 alphanumeric chars, username: 3-20 chars)

3. **GitHub Actions permissions issues**

   - If you see "Resource not accessible by integration" errors:
     - Check that the workflow has the necessary permissions in repository settings
     - Verify that the `permissions` section in the workflow file is correct
     - Make sure you've enabled "Read and write permissions" in your repository settings

4. **Keystore issues**
   - If the workflow fails with keystore-related errors:
     - Verify that the keystore was correctly converted to base64
     - Check that the keystore password, key alias, and key password are correct
     - Ensure the keystore file is valid and not corrupted
     - Try recreating the keystore and converting it to base64 again
     - If using default credentials, make sure your keystore was created with "Infinity" as the password and alias
     - Verify that the signing configuration in build.gradle matches your keystore credentials

### Getting Help

- Check the [Infinity For Reddit repository](https://github.com/Docile-Alligator/Infinity-For-Reddit) for build-related issues
- Open an issue in this repository for workflow-specific problems

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details.
