# Translate localization text
# 1. Add supported languages to project
1. Create folder `Localized` in the project root directory
2. Move the file `Localizable.strings` into the new created folder
1. Remove the file `Localizable.strings` reference from the project.
3. Open new terminal window at the folder `Localized`
1. Create a file with name `localized.rb` in the project root directory
    ```sh
    touch localized.rb
    ```
2. Add content to this file.
    - Replace `<project-name>` with your project name
    ```ruby
    require 'xcodeproj'
    project_path = '../<project-name>.xcodeproj'
    project = Xcodeproj::Project.open(project_path)

    for o in project.objects do 
        if o.is_a? Xcodeproj::Project::Object::PBXProject
            if o.isa == "PBXProject"
                group = o
                break
            end
        end
    end

    for o in project.objects do 
    if o.is_a? Xcodeproj::Project::Object::PBXGroup
        path = o.path
        if path && path.end_with?("Localized")
            LocalizableGroup = o
            break
        end
    end
    end

    # variantGroup = group.new(Xcodeproj::Project::Object::PBXVariantGroup)

    variant = LocalizableGroup.new_variant_group("Localizable.strings")

    langs = [                           # Languages to convert. i.e. English:en
    "en",
    "vi",
    "ar",
    "fr",
    "de",
    "id",
    "it",
    "ja",
    "ko",
    "pt-PT",
    "zh-Hans",
    "es",
    "th",
    "zh-Hant",
    "tr",
    "ca",
    "pl",
    "sk",
    "ms",
    "hr",
    "cs",
    "da",
    "nl"
    ]

    group.known_regions = langs

    for language in langs do
    dest_folder = language + ".lproj"
    Dir.mkdir(dest_folder)
    FileUtils.cp("Localizable.strings", dest_folder)
    file = project.new_file(language + ".lproj/Localizable.strings")
    file.move(variant)
    file.name = language
    end


    project.save

    ```

1. run: 
    ```sh
    ruby localized.rb
    ```
1. This will create a new localizable file reference in the project. Then you need to translate every single `Localizable.strings` file, that located in the `Localized/*.lproj` folder.
