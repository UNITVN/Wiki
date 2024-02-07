# Fastlane configuration

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
- in put application information into the top of Fastfile

```sh
open fastlane/Fastfile 
```

```ruby 
build_number = 2;
version_number = "1.1";
scheme = "audio-converter"
app_identifier = "com.unitvn.audioconverter"
```
---
---
## 3. Release application
### For firebase distribution
  1. Config fastlane
      1. Add the following lane to Fastfile
      ```sh
      open fastlane/Fastfile 
      ```
      ```

      platform :ios do
        desc "Description of what the lane does"
        lane :custom_lane do
          # add actions here: https://docs.fastlane.tools/actions
        end


        <<<include>>>


      end
      ```
      2. Update provisioningProfiles => app_identifier => your cer
      ```ruby 
      lane :firebase do
        increment_build_number(
          build_number: build_number
        )
        increment_version_number_in_xcodeproj(
          # specify specific version number (optional, omitting it increments patch version number)
          version_number: version_number,   
          # (optional, you must specify the path to your main Xcode project if it is not in the project root directory
          # or if you have multiple xcodeproj's in the root directory)
          xcodeproj: scheme + '.xcodeproj',
          # (optional)
          target: scheme # or `scheme`
        )
        gym(
          scheme: scheme,
          export_method: "ad-hoc",
          export_options: {
            provisioningProfiles: {
              app_identifier => "unit-adhoc-all",
            }
          }
        )
        
        firebase_app_distribution(
          app: "<your Firebase app ID>",
          groups: "unitmember",
          release_notes: "release"
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
1. Add the following lane to Fastfile:
```ruby
  lane :release do
    # ENV["FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD"] = "your-app-specific-password"

    increment_build_number(
      build_number: build_number
    )

    increment_version_number_in_xcodeproj(
      # specify specific version number (optional, omitting it increments patch version number)
      version_number: version_number,   
      # (optional, you must specify the path to your main Xcode project if it is not in the project root directory
      # or if you have multiple xcodeproj's in the root directory)
      xcodeproj: scheme + '.xcodeproj',
      # (optional)
      target: scheme # or `scheme`
    )

    gym(
      scheme: scheme,
      export_method: "app-store",
      export_options: {
        provisioningProfiles: {
          app_identifier => "appstore_provisioning_profile",
        }
      }
    )

    upload_to_app_store(
      username: "user@email.com",
      team_name: "UNIT VIET NAM JOINT STOCK COMPANY",
      force: true,
      app_version: version_number,
      app_identifier: app_identifier,
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
    4. Add following to the top of lane definition::
    ```
    ENV["FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD"] = "your-app-specific-password"
    ```
3. As your CI machine will not be able to prompt you for your two-factor authentication or two-step verification information, you can generate a login session for your Apple ID in advance by running:
```sh
bundle exec fastlane spaceauth -u user@email.com
```
- Run folowing command to build and upload application to the AppstoreConnect for review:
```sh
bundle exec fastlane release 
```
