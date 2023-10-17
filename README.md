# modify these two Above Files first In Dir.
- C:\src\flutter\packages\flutter_tools\templates\app


# Add this file in Directory and register in temp

- C:\src\flutter\packages\flutter_tools\templates\app_shared\.github.tmpl\workflows\build.yml

```yaml


name: Manual Workflow for Building APK OR AAB

on:
  workflow_dispatch:
    inputs:
      aab_generation:
        description: 'Generate AAB'
        required: false
        default: false
        type: boolean
      build_runner_build:
        description: 'Build Runner Build'
        required: false
        default: false
        type: boolean
      change_splash:
        description: 'Change Splash'
        required: false
        default: false
        type: boolean
      i18n_generation:
        description: 'Generate i18n'
        required: false
        default: false
        type: boolean
      generate_launcher_icons:
        description: 'Generate Launcher Icons'
        required: false
        default: false
        type: boolean
      custom_app_name:
        description: 'Custom App Name (Leave empty to skip)'
        required: false
        type: string
      custom_package_name:
        description: 'Custom Package Name (Leave empty to skip)'
        required: false
        type: string

jobs:
  manual_dispatch:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up Java
        uses: actions/setup-java@v1
        with:
          java-version: '12.x'
      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: 'stable'
      - name: Check Flutter Version
        run: flutter --version
      - name: Get Flutter Dependencies
        run: flutter pub get
      - name: Activate Rename
        run: flutter pub global activate rename

      - name: Build Runner Build
        if: ${{ github.event.inputs.build_runner_build == true }}
        run: flutter packages pub run build_runner build --delete-conflicting-outputs

      - name: Change Splash
        if: ${{ github.event.inputs.change_splash == true }}
        run: flutter pub run flutter_native_splash:create

      - name: Generate Launcher Icons
        if: ${{ github.event.inputs.generate_launcher_icons == true }}
        run: flutter pub run flutter_launcher_icons

      - name: Generate i18n
        if: ${{ github.event.inputs.i18n_generation == true }}
        run: flutter gen-l10n

      - name: Change App Name
        if: ${{ github.event.inputs.custom_app_name != '' }}
        run: rename setAppName --value "${{ github.event.inputs.custom_app_name }}"

      - name: Change Package Name
        if: ${{ github.event.inputs.custom-package-name != '' }}
        run: rename setBundleId --value "${{ github.event.inputs.custom-package-name }}"

      - name: Generate AAB or APK
        run: |
          if ${{ github.event.inputs.aab_generation == true }}; then
            flutter build appbundle --release
            echo "artifact_path=build/app/outputs/bundle/release/app-release.aab" >> $GITHUB_ENV
          else
            flutter build apk --release --split-per-abi
            echo "artifact_path=build/app/outputs/flutter-apk/app-release.apk" >> $GITHUB_ENV
          fi

      - name: Push to Releases
        uses: ncipollo/release-action@v1
        with:
          artifacts: ${{ env.artifact_path }}
          tag: v1.0.${{ github.run-number }}
          token: ${{ secrets.TOKEN }}



```


# Register this in template_manifest.json
- C:\src\flutter\packages\flutter_tools\templates\template_manifest.json


```json


"templates/app_shared/.github.tmpl/workflows/build.yml",
```


# for Ios 




```yml

name: Manual Workflow for Building IPA

on:
  workflow_dispatch:
    inputs:
      build_ipa:
        description: 'Build IPA'
        required: false
        default: false
        type: boolean
      build_runner_build:
        description: 'Build Runner Build'
        required: false
        default: false
        type: boolean
      change_splash:
        description: 'Change Splash'
        required: false
        default: false
        type: boolean
      i18n_generation:
        description: 'Generate i18n'
        required: false
        default: false
        type: boolean
      generate_launcher_icons:
        description: 'Generate Launcher Icons'
        required: false
        default: false
        type: boolean
      custom_ipa_name:
        description: 'Custom IPA Name (Leave empty to skip)'
        required: false
        type: string
      custom_ipa_bundle_id:
        description: 'Custom IPA Bundle Identifier (Leave empty to skip)'
        required: false
        type: string
      push_in_Release:
        description: 'Push in Release'
        required: false
        default: true
        type: boolean

jobs:
  manual_dispatch:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v2

      - name: Build Runner Build
        if: ${{ github.event.inputs.build_runner_build == true }}
        run: flutter packages pub run build_runner build --delete-conflicting-outputs

      - name: Change Splash
        if: ${{ github.event.inputs.change_splash == true }}
        run: flutter pub run flutter_native_splash:create

      - name: Generate Launcher Icons
        if: ${{ github.event.inputs.generate_launcher_icons == true }}
        run: flutter pub run flutter_launcher_icons

      - name: Generate i18n
        if: ${{ github.event.inputs.i18n_generation == true }}
        run: flutter gen-l10n

      - name: Build IPA
        if: ${{ github.event.inputs.build_ipa == true }}
        run: |
          flutter build ios --release
          mv build/ios/iphoneos/Runner.app Runner.ipa

          if [[ -n "${{ github.event.inputs.custom_ipa_name }}" ]]; then
            mv Runner.ipa "${{ github.event.inputs.custom_ipa_name }}.ipa"
          fi
          if [[ -n "${{ github.event.inputs.custom_ipa_bundle_id }}" ]]; then
            plutil -replace CFBundleIdentifier -string "${{ github.event.inputs.custom_ipa_bundle_id }}" "${{ github.event.inputs.custom_ipa_name }}.ipa/Info.plist"
          fi

      - name: Push IPA to Releases
        if: ${{ github.event.inputs.build_ipa == true }}
        uses: actions/upload-artifact@v2
        with:
          name: YourApp-IPA
          path: "${{ github.event.inputs.custom_ipa_name }}.ipa"

      - name: Push to Releases
        if: ${{ github.event.inputs.push_in_Release == true }}
        uses: ncipollo/release-action@v1
        with:
          artifacts: "${{ github.event.inputs.custom_ipa_name }}.ipa"
          tag: v1.0-${{ github.run_number }}
          token: ${{ secrets.TOKEN }}






```