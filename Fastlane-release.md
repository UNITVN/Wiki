# Fastlane configuration

## Install fastlane
- Create a ./Gemfile in the root directory of your project with the content:
```ruby
source "https://rubygems.org"

gem "fastlane"
```
- Run `bundle update`
- Add both the `./Gemfile` and the `./Gemfile.lock` to version control

- add firebase_app_distribution plugin. Type folowing command into `Terminal`:
```sh 
bundle exec fastlane add_plugin versioning
```

## Application information
- in put application information into the top of Fastfile
```ruby 
build_number = 2;
version_number = "1.1";
scheme = "audio-converter"
app_identifier = "com.unitvn.audioconverter"
```

## For firebase distribution
- Add the following lane to Fastfile
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
      groups: "unitmember",
      release_notes: "release"
    )
  end
```
- add firebase_app_distribution plugin. Type folowing command into `Terminal`
```sh
bundle exec fastlane add_plugin firebase_app_distribution
```
- Run folowing command to build and upload application to the Firebase distribution:
```sh
bundle exec fastlane firebase
```

## For appstore distribution
- Add the following lane to Fastfile:
```ruby
  lane :release do
    # Run this if login session is expired: bundle exec fastlane spaceauth -u user@email.com

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
          app_identifier => "audioconverter-appstore",
        }
      }
    )

    upload_to_app_store(
      username: "trmquang3103@gmail.com",
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
- As your CI machine will not be able to prompt you for your two-factor authentication or two-step verification information, you can generate a login session for your Apple ID in advance by running:
```sh
bundle exec fastlane spaceauth -u user@email.com
```
- Run folowing command to build and upload application to the AppstoreConnect for review:
```sh
bundle exec fastlane release 
```
