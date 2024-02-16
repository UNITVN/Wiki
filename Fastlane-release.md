# Fastlane configuration
Macos M1 Install ruby = rbenv
Install rbenv
# Install rbenv
```
brew install rbenv
rbenv init
curl -fsSL https://github.com/rbenv/rbenv-installer/raw/main/bin/rbenv-doctor | bash

benv install -l 
=>> Select lastest version

rbenv install 3.0.3

rbenv global 3.0.3

ruby -v
```

## 1. Install fastlane
- Create a ./Gemfile in the root directory of your project with the content:
```ruby
source "https://rubygems.org"

gem "fastlane"
```
- Run `bundle update`
- Add both the `./Gemfile` and the `./Gemfile.lock` to version control

- Create dir fastlane -> select option is 4 manual
```sh  
bundle exec fastlane init
```
- add firebase_app_distribution plugin. Type folowing command into `Terminal`:
```sh 
bundle exec fastlane add_plugin versioning
```
---
---
## 2. Application information
- Open Fastfile
```sh
open fastlane/Fastfile 
```
- Add following lines to the top of Fastfile

```ruby 
# Load template
import_from_git(
  url: 'git@github.com:UNITVN/default-fastfile.git',
  path: 'default-fastfile'
)

# Replace following lines with your own values from project
ENV["BUILD_NUMBER"] = "1"
ENV["VERSION_NUMBER"] = "1.1.1"
ENV["SCHEME_NAME"] = "snap-editor"
ENV["APP_IDENTIFIER"] = "com.unitvn.photoeditor"
ENV["FASTLANE_USER"] = "Quangtm193@gmail.com"
ENV["ADHOC_PROVISIONING_PROFILE"] = "snap-edit-adhoc"
ENV["APPSTORE_PROVISIONING_PROFILE"] = "snapeditor_appstore"
```
---
---
## 3. Release application
### For firebase distribution
  1. Config fastlane
      1. Add the following lane to Fastfile inside `platform :ios do` block: 
      ```
      desc "Build and upload to firebase"
      lane :firebase do
        build(
          export_method: "ad-hoc"
        )
        
        firebase_app_distribution(
          app: "<your Firebase app ID>"
          groups: "unitmember",
          release_notes: "sprint 4",
          service_credentials_file: "/Users/quang.tranminh/Library/Mobile Documents/com~apple~CloudDocs/unitvnios-8079edc9e94a.json"
        )
      end
      ```
      2. add firebase_app_distribution plugin. Type folowing command into `Terminal`
      ```
      bundle exec fastlane add_plugin firebase_app_distribution
      ```
      3. Replace `<your Firebase app ID>` with your firebase app ID. Required only if your app does not contain a Firebase config file (GoogleService-Info.plist): Your app's Firebase App ID. You can find the App ID in the Firebase console, on the [General Settings page](https://console.firebase.google.com/project/_/settings/general/?authuser=0).

  2. Authenticate with Firebase
     - Use Firebase service account credentials
      > - Add Account access permission to group project 
      > - On the [Google Cloud console](https://console.cloud.google.com/projectselector2/iam-admin/serviceaccounts?authuser=0), select your project and create a new service account.
      > - Add the **Firebase App Distribution Admin** role
      > - Create a private json key and move the key to a location accessible to your build environment. Be sure to keep this file somewhere safe, as it grants administrator access to App Distribution in your Firebase project
      > 1. Provide or locate your service account credentials:
      >    - a. To pass your service account key to your lane's firebase_app_distribution action, set the service_credentials_file parameter with the path to your private key JSON file
      >
      >    - b. To locate your credentials with ADC, set the environment variable GOOGLE_APPLICATION_CREDENTIALS to the path for the >private key JSON file. For example:
      >    ```
      >     export GOOGLE_APPLICATION_CREDENTIALS=/absolute/path/to/credentials/file.json
      >    ```
      - Sign in using the Firebase CLI
      > - install firebase CLI
      > ```sh 
      > curl -sL https://firebase.tools | bash
      > ```
      > - Log into Firebase using your Google account by running the following command:
      > 
      > ###### This command connects your local machine to Firebase and grants you access to your Firebase projects.
      > ```
      > firebase login
      > ```

3. Distribute the application
- Run folowing command to build and upload application to the Firebase distribution:
```sh
bundle exec fastlane firebase
```
---
### For appstore distribution
1. Add the following lane to Fastfile inside `platform :ios do` block:
```ruby
desc "Build and upload to AppstoreConnect"
lane :release do
  ENV["FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD"] = "<APPLICATION_SPECIFIC_PASSWORD>"

  build(
    export_method: "app-store"
  )

  upload_to_app_store(
    username: ENV["FASTLANE_USER"],
    team_name: "UNIT VIET NAM JOINT STOCK COMPANY",
    force: true,
    app_version: ENV["VERSION_NUMBER"],
    app_identifier: ENV["APP_IDENTIFIER"],
    submit_for_review: true,
    run_precheck_before_submit: false,
    skip_binary_upload: false,
    skip_screenshots: true,
    skip_metadata: false,
    automatic_release: true,
    submission_information: {
      export_compliance_uses_encryption: false,
      add_id_info_uses_idfa: false
    }
  )
end
```
2. Replace: 
    - `user@email.com` with your `email` 
    - `appstore_provisioning_profile` **with** provision profile that can be use to upload app binary to appstore
3. Create APPLICATION_SPECIFIC_PASSWORD
    1. Sign in to [appleid.apple.com](appleid.apple.com).
    2. In the **Sign-In and Security** section, select **App-Specific Passwords**.
    3. Select **Generate an app-specific password** or select the Add button, then follow the steps on your screen.
    4. Replace `<APPLICATION_SPECIFIC_PASSWORD>` with `your application specific password`:
    ```
    ENV["FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD"] = "your-app-specific-password"
    ```
- Run folowing command to build and upload application to the AppstoreConnect for review:
```sh
bundle exec fastlane release 
```
