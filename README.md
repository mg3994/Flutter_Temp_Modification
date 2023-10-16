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
        default: 'false'
      build_runner_build:
        description: 'Build Runner Build'
        required: false
        default: 'false'
      change_splash:
        description: 'Change Splash'
        required: false
        default: 'false'
      i18n_generation:
        description: 'Generate i18n'
        required: false
        default: 'false'
      generate_launcher_icons:
        description: 'Generate Launcher Icons'
        required: false
        default: 'false'
      custom_app_name:
        description: 'Custom App Name (Leave empty to skip)'
        required: false
      custom_package_name:
        description: 'Custom Package Name (Leave empty to skip)'
        required: false

jobs:
  manual_dispatch:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-java@v1
        with:
          java-version: '12.x'
      - uses: subosito/flutter-action@v2
        with:
          channel: 'stable'
      - run: flutter --version
      - run: flutter pub get
      - run: flutter pub global activate rename

      # Run Build Runner Build if the input is 'true'
      - name: Build Runner Build
        if: ${{ github.event.inputs.build_runner_build == 'true' }}
        run: flutter packages pub run build_runner build --delete-conflicting-outputs

      # Change Splash if the input is 'true'
      - name: Change Splash
        if: ${{ github.event.inputs.change_splash == 'true' }}
        run: flutter pub run flutter_native_splash:create

      # Generate Launcher Icons if the input is 'true'
      - name: Generate Launcher Icons
        if: ${{ github.event.inputs.generate_launcher_icons == 'true' }}
        run: flutter pub run flutter_launcher_icons

      # Change App Name if the input is 'true' and custom_app_name is provided
      - name: Change App Name
        if: ${{ github.event.inputs.custom_app_name != '' }}
        run: rename setAppName --value "${{ github.event.inputs.custom_app_name }}"

      # Change Package Name if the input is 'true' and custom_package_name is provided
      - name: Change Package Name
        if: ${{ github.event.inputs.custom_package_name != '' }}
        run: rename setBundleId --value "${{ github.event.inputs.custom_package_name }}"

      # Generate i18n if the input is 'true'
      - name: Generate i18n
        if: ${{ github.event.inputs.i18n_generation == 'true' }}
        run: flutter gen-l10n

      - name: Generate AAB or APK
        id: generate_artifact
        run: |
          if [ "${{ github.event.inputs.aab_generation }}" = 'true' ]; then
            flutter build appbundle --release
            echo "::set-output name=artifact_path::build/app/outputs/bundle/release/app-release.aab"
          else
            flutter build apk --release --split-per-abi
            echo "::set-output name=artifact_path::build/app/outputs/flutter-apk/app-release.apk"
          fi
        shell: bash

      - name: Push to Releases
        uses: ncipollo/release-action@v1
        with:
          artifacts: ${{ steps.generate_artifact.outputs.artifact_path }}
          tag: v1.0.${{ github.run_number }}
          token: ${{ secrets.TOKEN }}

```


# Register this in template_manifest.json
- C:\src\flutter\packages\flutter_tools\templates\template_manifest.json


```json


"templates/app_shared/.github.tmpl/workflows/build.yml",
```